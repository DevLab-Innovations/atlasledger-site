# Recurring Bills Technical Documentation

This document provides technical details for developers working with the recurring bills system in Atlas Ledger.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Database Schema](#database-schema)
- [RecurringBillService API](#recurringbillservice-api)
- [Period Key Format](#period-key-format)
- [Date Calculations](#date-calculations)
- [UI Integration](#ui-integration)
- [Migration Guide](#migration-guide)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

---

## Architecture Overview

### Design Pattern: Hybrid Approach

The recurring bills system uses a **hybrid architecture** that combines:

1. **Enhanced Bills Table** - Stores bill definitions with period tracking fields
2. **Bill Payments Table** - Tracks payment history per period

This approach provides:

- Full payment history without modifying existing bill records
- Period-aware payment tracking for accurate overdue detection
- Support for variable amounts, partial payments, and skipped periods
- Minimal breaking changes to existing codebase

### Key Components

```
┌─────────────────────┐
│  BillsListScreen    │  Displays bills with period context
│  BillFormScreen     │  Create/edit bills with start_date
│  BillPaymentHistory │  Shows payment history
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ RecurringBillService│  Business logic layer
│  - Period calc      │  Singleton service
│  - Due date calc    │  Pattern: RecurringIncomeService
│  - Payment tracking │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  DatabaseService    │  Data access layer
│  - Bills CRUD       │
│  - BillPayments CRUD│
└─────────────────────┘
           │
           ▼
┌─────────────────────┐
│   SQLite Database   │
│  - bills table      │
│  - bill_payments    │
└─────────────────────┘
```

### Design Reference

See [`/openspec/changes/design-recurring-bills.md`](/Users/neptune/repos/agenticSdlc/openspec/changes/design-recurring-bills.md) for the full design rationale and decision record.

---

## Database Schema

### `bills` Table (Enhanced)

```sql
CREATE TABLE bills (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  name TEXT NOT NULL,
  amount REAL NOT NULL,
  due_day INTEGER NOT NULL,  -- 1-31 for monthly, 1-7 for weekly
  category TEXT NOT NULL,
  is_recurring BOOLEAN NOT NULL DEFAULT 1,
  frequency TEXT CHECK(frequency IN ('weekly', 'monthly', 'quarterly', 'annual')),

  -- New fields for period tracking
  start_date TEXT,           -- ISO date: when bill tracking starts
  anchor_date TEXT,          -- ISO date: for weekly/biweekly alignment

  -- Legacy fields (maintained for backward compatibility)
  is_paid BOOLEAN DEFAULT 0,
  paid_date TEXT,
  expense_id INTEGER,

  notes TEXT,
  pay_period_override TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
);
```

**Key Fields**:

- `start_date`: Determines the first payment period. Defaults to `created_at` date.
- `anchor_date`: For weekly bills, aligns week calculations.
- `due_day`: For monthly bills (1-31), for weekly bills (1-7, where 1=Monday).

### `bill_payments` Table (New)

```sql
CREATE TABLE bill_payments (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  bill_id INTEGER NOT NULL,
  period_key TEXT NOT NULL,           -- e.g., "2026-01", "2026-W05", "2026-Q1", "2026"
  due_date TEXT NOT NULL,             -- Full ISO date: "2026-01-15"
  amount_due REAL NOT NULL,           -- Expected amount
  amount_paid REAL,                   -- Actual amount paid
  status TEXT NOT NULL DEFAULT 'pending' CHECK(status IN ('pending', 'paid', 'skipped', 'partial')),
  paid_date TEXT,                     -- When payment was made
  expense_id INTEGER,                 -- Link to expense entry
  skip_reason TEXT,                   -- Reason for skipping
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (bill_id) REFERENCES bills(id) ON DELETE CASCADE,
  FOREIGN KEY (expense_id) REFERENCES expenses(id) ON DELETE SET NULL,
  UNIQUE(bill_id, period_key)         -- One record per bill per period
);

CREATE INDEX idx_bill_payments_bill_id ON bill_payments(bill_id);
CREATE INDEX idx_bill_payments_period_key ON bill_payments(period_key);
CREATE INDEX idx_bill_payments_due_date ON bill_payments(due_date);
CREATE INDEX idx_bill_payments_status ON bill_payments(status);
```

**Key Constraints**:

- `UNIQUE(bill_id, period_key)`: Ensures only one payment record per period
- `ON DELETE CASCADE`: Deleting a bill deletes all payment records
- `ON DELETE SET NULL`: Deleting an expense doesn't delete payment record

---

## RecurringBillService API

### Singleton Pattern

```typescript
import { RecurringBillService } from '@/services/RecurringBillService';

const service = RecurringBillService.getInstance();
```

### Core Methods

#### `getPendingPayments(options)`

Get all pending bill payments for an account within a date range.

```typescript
interface GetPendingPaymentsOptions {
  accountId: number;
  startDate?: string;  // ISO date, defaults to start of current month
  endDate?: string;    // ISO date, defaults to end of next month
}

const pending = await service.getPendingPayments({
  accountId: 1,
  startDate: '2026-01-01',
  endDate: '2026-02-28',
});

// Returns: PendingBillPayment[]
// [
//   {
//     bill: { id: 1, name: "Netflix", amount: 15.99, ... },
//     payment: { period_key: "2026-01", due_date: "2026-01-15", status: "pending", ... },
//     daysUntilDue: 5,
//     isOverdue: false
//   },
//   ...
// ]
```

**Behavior**:

- Generates payment records on-demand (not stored until marked as paid)
- Excludes already paid or skipped periods
- Sorted by overdue status first, then due date

#### `markAsPaid(options)`

Mark a bill payment as paid for a specific period.

```typescript
interface MarkAsPaidOptions {
  billId: number;
  periodKey: string;
  paidDate?: string;      // ISO date, defaults to today
  amountPaid?: number;    // Defaults to bill amount
  notes?: string;
}

const result = await service.markAsPaid({
  billId: 1,
  periodKey: '2026-01',
  paidDate: '2026-01-15',
  amountPaid: 127.50,     // Variable amount
  notes: 'Higher usage this month',
});

// Returns: { paymentId: 42, expenseId: 123 }
```

**Side Effects**:

1. Creates an expense entry with category and amount
2. Creates or updates `bill_payments` record
3. Syncs legacy `is_paid`, `paid_date`, `expense_id` fields on `bills` table

**Error Cases**:

- Throws if bill not found
- Throws if already paid for this period

#### `unmarkAsPaid(billId, periodKey)`

Undo a payment (mark as unpaid).

```typescript
await service.unmarkAsPaid(1, '2026-01');
```

**Side Effects**:

1. Deletes the linked expense entry
2. Resets payment record to `pending` status
3. Syncs legacy fields on `bills` table

#### `skipPayment(billId, periodKey, reason?)`

Mark a payment as intentionally skipped.

```typescript
const paymentId = await service.skipPayment(
  1,
  '2026-02',
  'On vacation - paused subscription'
);
```

**Behavior**:

- Creates or updates payment record with `status = 'skipped'`
- Skipped payments don't show as overdue
- No expense entry created

#### `checkOverdue(bill)`

Check if a bill is overdue for the current period.

```typescript
const overdueInfo = await service.checkOverdue(bill);

if (overdueInfo) {
  console.log(`Overdue by ${overdueInfo.daysOverdue} days`);
  console.log(`Period: ${overdueInfo.periodKey}`);
  console.log(`Due date: ${overdueInfo.dueDate}`);
}
```

**Returns**:

- `null` if not overdue (already paid/skipped or not yet due)
- Object with `isOverdue`, `daysOverdue`, `periodKey`, `dueDate` if overdue

#### `getPaymentHistory(billId, options?)`

Get payment history for a bill.

```typescript
const history = await service.getPaymentHistory(1, {
  limit: 12,
  fromDate: '2025-01-01',
  toDate: '2026-12-31',
});

// Returns: BillPayment[] (newest first)
```

#### `isPaidForPeriod(billId, periodKey)`

Check if a bill has been paid for a specific period.

```typescript
const isPaid = await service.isPaidForPeriod(1, '2026-01');
// Returns: boolean
```

### Period Calculation Methods

#### `getPeriodKey(date, frequency)`

Calculate the period key for a given date and frequency.

```typescript
const periodKey = service.getPeriodKey(new Date('2026-01-15'), 'monthly');
// Returns: "2026-01"

const weekKey = service.getPeriodKey(new Date('2026-01-29'), 'weekly');
// Returns: "2026-W05"

const quarterKey = service.getPeriodKey(new Date('2026-04-15'), 'quarterly');
// Returns: "2026-Q2"

const yearKey = service.getPeriodKey(new Date('2026-06-30'), 'annual');
// Returns: "2026"
```

#### `getNextPeriodKey(currentKey, frequency)`

Get the next period after the current one.

```typescript
const nextMonth = service.getNextPeriodKey('2026-01', 'monthly');
// Returns: "2026-02"

const nextQuarter = service.getNextPeriodKey('2026-Q4', 'quarterly');
// Returns: "2027-Q1"
```

#### `calculateDueDate(bill, periodKey)`

Calculate the full due date for a bill in a specific period.

```typescript
const bill = { due_day: 15, frequency: 'monthly', ... };
const dueDate = service.calculateDueDate(bill, '2026-01');
// Returns: "2026-01-15"

// Handles day 31 in short months
const feb31 = service.calculateDueDate({ due_day: 31, frequency: 'monthly' }, '2026-02');
// Returns: "2026-02-28" (clamped to last day of Feb)
```

---

## Period Key Format

Period keys identify specific billing periods. The format varies by frequency:

| Frequency | Format | Example | Notes |
|-----------|--------|---------|-------|
| `weekly` | `YYYY-Www` | `2026-W05` | ISO week number (1-53) |
| `monthly` | `YYYY-MM` | `2026-01` | Standard month (01-12) |
| `quarterly` | `YYYY-Qq` | `2026-Q1` | Quarter 1-4 |
| `annual` | `YYYY` | `2026` | Year only |

### ISO Week Numbers

Weekly bills use [ISO 8601 week dates](https://en.wikipedia.org/wiki/ISO_week_date):

- Week starts on Monday
- Week 1 is the week containing January 4th
- Most years have 52 weeks, some have 53

**Example**:
- `2026-W01`: Monday, Dec 29, 2025 to Sunday, Jan 4, 2026
- `2026-W05`: Monday, Jan 26, 2026 to Sunday, Feb 1, 2026

---

## Date Calculations

### Due Date Calculation Logic

#### Monthly Bills

For a bill due on day `D` in period `YYYY-MM`:

1. Get the last day of the month
2. Clamp `D` to the valid range: `min(D, lastDay)`
3. Return `YYYY-MM-DD`

**Example**: Bill due on 31st

```
Jan 2026: 31 days → due Jan 31
Feb 2026: 28 days → due Feb 28  (clamped)
Mar 2026: 31 days → due Mar 31
Apr 2026: 30 days → due Apr 30  (clamped)
```

#### Weekly Bills

For a bill due on day-of-week `D` (1=Monday, 7=Sunday) in period `YYYY-Www`:

1. Calculate the Monday of ISO week `w` in year `YYYY`
2. Add `(D - 1)` days to get the target day
3. Return ISO date

#### Quarterly Bills

Quarters map to months:

- Q1: January-March → due in **January**
- Q2: April-June → due in **April**
- Q3: July-September → due in **July**
- Q4: October-December → due in **October**

The `due_day` specifies the day within the first month of the quarter.

#### Annual Bills

The `start_date` determines which month the annual bill is due. The `due_day` specifies the day within that month.

**Example**:
- Start date: March 15, 2025
- Due day: 15
- Annual periods: 2025 → due Mar 15, 2025; 2026 → due Mar 15, 2026

### Overdue Detection

The system compares **full due dates**, not just day numbers.

**Algorithm**:

```typescript
function isOverdue(bill: Bill): boolean {
  const today = new Date();
  const currentPeriod = getPeriodKey(today, bill.frequency);

  // Check if already paid this period
  const payment = await getBillPaymentByPeriod(bill.id, currentPeriod);
  if (payment?.status === 'paid' || payment?.status === 'skipped') {
    return false;  // Not overdue - period handled
  }

  // Calculate due date for current period
  const dueDate = calculateDueDate(bill, currentPeriod);

  // Compare full dates
  return today > new Date(dueDate);
}
```

**Why This Matters**:

Old system (broken):
```
Bill due on 1st, today is Jan 29
→ 29 > 1 → OVERDUE ❌ (false positive if Jan already paid)
```

New system (correct):
```
Bill due on 1st, today is Jan 29
→ Current period: "2026-01"
→ Due date: "2026-01-01"
→ Check: Already paid for "2026-01"? YES
→ NOT OVERDUE ✓
```

---

## UI Integration

### BillsListScreen Integration

The Bills List screen uses `RecurringBillService` to:

1. **Load pending payments** for the current/next period
2. **Display period context** (e.g., "Paid - January")
3. **Show days until due** or days overdue
4. **Group bills** by status (overdue, upcoming, paid)

**Example Implementation**:

```typescript
const recurringBillService = RecurringBillService.getInstance();

// Load pending payments
const pending = await recurringBillService.getPendingPayments({
  accountId: defaultAccount.id,
});

// Display each bill with period context
pending.forEach((item) => {
  const { bill, payment, daysUntilDue, isOverdue } = item;

  // Show period-aware status
  if (payment.status === 'paid') {
    statusText = `Paid - ${formatPeriodKey(payment.period_key)}`;
  } else if (isOverdue) {
    statusText = `${Math.abs(daysUntilDue)} days overdue`;
  } else {
    statusText = `Due in ${daysUntilDue} days`;
  }
});
```

### BillFormScreen Integration

The Bill Form allows setting:

- **Start Date**: When to begin tracking
- **Due Day**: Day of month/week for due date
- **Frequency**: How often it repeats

**Validation**:

- `due_day` must be 1-31 for monthly bills
- `due_day` must be 1-7 for weekly bills (1=Monday)
- `start_date` is required for recurring bills

### BillPaymentHistoryScreen

Displays payment history grouped by year, showing:

- Period name (formatted)
- Status badge (Paid, Skipped, Partial)
- Due date and paid date
- Amount paid vs. expected
- Variance (if different)
- Late indicator (if paid after due date)

---

## Migration Guide

### Database Migration (v10)

The migration adds the `bill_payments` table and enhances the `bills` table.

**Key Steps**:

1. Create `bill_payments` table
2. Add `start_date` and `anchor_date` to `bills`
3. Backfill `start_date` from `created_at`
4. Create payment records for currently-paid bills
5. Create indexes

**Backward Compatibility**:

Legacy fields (`is_paid`, `paid_date`, `expense_id`) are **kept and synced** during a transition period. This ensures:

- Existing code continues to work
- Gradual migration to new system
- Future cleanup migration can remove legacy fields

### For Existing Bills

When the migration runs:

1. All existing bills get `start_date = date(created_at)`
2. Bills marked `is_paid = 1` get a payment record for the current period
3. Bills marked `is_paid = 0` have no payment records (will generate on-demand)

### For Developers

**Before Migration**:

```typescript
const bill = await dbService.getBillById(1);
if (bill.is_paid) {
  console.log(`Paid on ${bill.paid_date}`);
}
```

**After Migration** (both work):

```typescript
// Option 1: Legacy fields (still synced)
if (bill.is_paid) {
  console.log(`Paid on ${bill.paid_date}`);
}

// Option 2: New system (recommended)
const isPaid = await recurringBillService.isPaidForPeriod(bill.id, currentPeriod);
if (isPaid) {
  const payment = await dbService.getBillPaymentByPeriod(bill.id, currentPeriod);
  console.log(`Paid on ${payment.paid_date} for period ${payment.period_key}`);
}
```

---

## Testing

### Unit Tests

Key test cases for `RecurringBillService`:

**Period Key Calculations**:

```typescript
describe('getPeriodKey', () => {
  it('returns correct monthly period key', () => {
    expect(service.getPeriodKey(new Date('2026-01-15'), 'monthly')).toBe('2026-01');
  });

  it('returns correct weekly period key', () => {
    expect(service.getPeriodKey(new Date('2026-01-29'), 'weekly')).toBe('2026-W05');
  });

  it('returns correct quarterly period key', () => {
    expect(service.getPeriodKey(new Date('2026-04-15'), 'quarterly')).toBe('2026-Q2');
  });

  it('returns correct annual period key', () => {
    expect(service.getPeriodKey(new Date('2026-06-30'), 'annual')).toBe('2026');
  });
});
```

**Due Date Calculations**:

```typescript
describe('calculateDueDate', () => {
  it('clamps day 31 to February 28 in non-leap year', () => {
    const bill = { due_day: 31, frequency: 'monthly' };
    expect(service.calculateDueDate(bill, '2026-02')).toBe('2026-02-28');
  });

  it('handles leap year February correctly', () => {
    const bill = { due_day: 31, frequency: 'monthly' };
    expect(service.calculateDueDate(bill, '2024-02')).toBe('2024-02-29');
  });

  it('handles day 31 in 30-day months', () => {
    const bill = { due_day: 31, frequency: 'monthly' };
    expect(service.calculateDueDate(bill, '2026-04')).toBe('2026-04-30');
  });
});
```

**Overdue Detection**:

```typescript
describe('checkOverdue', () => {
  it('returns null if paid for current period', async () => {
    // Setup: bill paid for current period
    const result = await service.checkOverdue(paidBill);
    expect(result).toBeNull();
  });

  it('returns overdue info if past due date and unpaid', async () => {
    // Setup: bill due on 15th, today is 20th, not paid
    const result = await service.checkOverdue(unpaidBill);
    expect(result.isOverdue).toBe(true);
    expect(result.daysOverdue).toBe(5);
  });

  it('correctly handles month boundary', async () => {
    // Setup: bill due on 1st, today is Jan 29, paid in January
    const result = await service.checkOverdue(paidJanuaryBill);
    expect(result).toBeNull(); // NOT overdue
  });
});
```

### Integration Tests

Test the full payment flow:

```typescript
describe('Bill Payment Flow', () => {
  it('marks bill as paid and creates expense', async () => {
    const bill = await createTestBill({ amount: 100, frequency: 'monthly' });
    const periodKey = '2026-01';

    const result = await service.markAsPaid({
      billId: bill.id,
      periodKey,
      amountPaid: 100,
    });

    // Verify payment record
    const payment = await dbService.getBillPaymentByPeriod(bill.id, periodKey);
    expect(payment.status).toBe('paid');
    expect(payment.amount_paid).toBe(100);

    // Verify expense created
    const expense = await dbService.getExpenseById(result.expenseId);
    expect(expense.amount).toBe(100);
    expect(expense.description).toContain('January 2026');
  });

  it('handles unmark as paid (undo)', async () => {
    // ... mark as paid ...
    await service.unmarkAsPaid(bill.id, periodKey);

    // Verify payment reset
    const payment = await dbService.getBillPaymentByPeriod(bill.id, periodKey);
    expect(payment.status).toBe('pending');

    // Verify expense deleted
    const expense = await dbService.getExpenseById(expenseId);
    expect(expense).toBeNull();
  });
});
```

### Manual Testing Scenarios

1. **Monthly Bill Flow**:
   - Create bill due on 15th
   - Verify shows as pending
   - Mark as paid
   - Verify shows "Paid - [Month]"
   - Mock date to next month
   - Verify shows as pending again

2. **Day 31 Edge Case**:
   - Create bill due on 31st
   - Check due dates: Jan 31, Feb 28, Mar 31, Apr 30

3. **Overdue Detection**:
   - Create bill due on 1st
   - On the 15th, verify shows as overdue
   - Mark as paid
   - Verify not overdue

---

## Troubleshooting

### Bills Not Showing as Paid After Marking

**Symptom**: You mark a bill as paid, but it still shows as pending.

**Causes**:

1. **Wrong period marked**: You marked a different period than displayed
2. **Database sync issue**: Payment record not created
3. **Cache issue**: UI not refreshed

**Fix**:

```typescript
// Check payment record exists
const payment = await dbService.getBillPaymentByPeriod(billId, periodKey);
console.log('Payment record:', payment);

// Force reload
await loadBills();
```

### Incorrect Due Dates

**Symptom**: Due dates don't match expected values.

**Causes**:

1. **Start date is wrong**: Bill started on different date than expected
2. **Timezone issues**: Dates converted incorrectly
3. **Day 31 clamping**: Expected behavior for short months

**Fix**:

```typescript
// Check bill start_date
const bill = await dbService.getBillById(billId);
console.log('Start date:', bill.start_date);

// Recalculate due date
const dueDate = service.calculateDueDate(bill, periodKey);
console.log('Calculated due date:', dueDate);
```

### Duplicate Payment Records

**Symptom**: Multiple payment records for the same period.

**Cause**: Database constraint not enforced (shouldn't happen due to `UNIQUE(bill_id, period_key)`).

**Fix**:

```sql
-- Find duplicates
SELECT bill_id, period_key, COUNT(*) as count
FROM bill_payments
GROUP BY bill_id, period_key
HAVING count > 1;

-- Delete duplicates (keep most recent)
DELETE FROM bill_payments
WHERE id NOT IN (
  SELECT MAX(id)
  FROM bill_payments
  GROUP BY bill_id, period_key
);
```

### Performance Issues with Many Bills

**Symptom**: Bills List screen loads slowly with 100+ bills.

**Optimization**:

1. **Limit date range**: Only load current and next month
2. **Pagination**: Load bills in batches
3. **Caching**: Cache payment records for active periods
4. **Indexes**: Ensure indexes on `bill_id`, `period_key`, `due_date`

**Example**:

```typescript
// Before (slow)
const pending = await service.getPendingPayments({ accountId: 1 });

// After (fast)
const pending = await service.getPendingPayments({
  accountId: 1,
  startDate: startOfMonth(new Date()),
  endDate: endOfMonth(addMonths(new Date(), 1)),
});
```

---

## References

- **Design Document**: [`/openspec/changes/design-recurring-bills.md`](/Users/neptune/repos/agenticSdlc/openspec/changes/design-recurring-bills.md)
- **Implementation Plan**: [`/openspec/changes/tasks-recurring-bills.md`](/Users/neptune/repos/agenticSdlc/openspec/changes/tasks-recurring-bills.md)
- **Pattern Reference**: [`RecurringIncomeService.ts`](/Users/neptune/repos/agenticSdlc/src/services/RecurringIncomeService.ts)
- **Type Definitions**: [`/src/types/index.ts`](/Users/neptune/repos/agenticSdlc/src/types/index.ts)

---

**Last Updated**: 2026-01-30
**Author**: Technical Writer Agent
**Applies to**: Atlas Ledger v2.0+

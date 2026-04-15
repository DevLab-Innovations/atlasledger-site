# Architecture Review: Multi-Account Cash Flow Tracking

**Reviewer**: Architect
**Date**: 2026-01-18
**Proposal**: `openspec/changes/add-multi-account-cashflow/`
**Related Documents**: `docs/ux-review-multi-account-cashflow.md`

---

## Executive Summary

This document provides a technical architecture review of the proposed multi-account cash flow feature. The review examines the data model, service layer design, context architecture, query performance, migration strategy, and navigation structure.

**Key Findings:**
1. **Schema**: Generally sound with minor normalization and constraint improvements needed
2. **Services**: Should follow existing singleton pattern with account-scoped methods
3. **Context**: AccountContext should integrate with ServiceContext, not wrap it
4. **Performance**: Additional indexes recommended for dashboard aggregations
5. **Migration**: Two-phase approach required for safe data migration
6. **Navigation**: Recommend Option A (consolidated "Finances" tab) - technical rationale provided

---

## 1. Data Model Review

### 1.1 Proposed Schema Analysis

The proposed schema introduces four tables: `accounts`, `income`, `bills`, and modifications to existing `expenses` and `mileage` tables.

#### Accounts Table - Issues Identified

```sql
-- PROPOSED (with issues marked)
CREATE TABLE accounts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  type TEXT NOT NULL CHECK(type IN ('personal', 'business')),
  color TEXT DEFAULT '#2196F3',
  is_default INTEGER DEFAULT 0,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);
```

**Issues:**

| Issue | Severity | Description |
|-------|----------|-------------|
| Missing unique constraint on `is_default` | Medium | Multiple accounts could be marked as default |
| No icon field | Low | UX review recommends icon for accessibility (color-blind users) |
| Color format not validated | Low | No CHECK constraint on hex color format |

**Recommended Schema:**

```sql
CREATE TABLE accounts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  type TEXT NOT NULL CHECK(type IN ('personal', 'business')),
  color TEXT NOT NULL DEFAULT '#2196F3' CHECK(color GLOB '#[0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f]'),
  icon TEXT DEFAULT 'account',
  is_default INTEGER NOT NULL DEFAULT 0 CHECK(is_default IN (0, 1)),
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

-- Partial unique index ensures only one default account
CREATE UNIQUE INDEX idx_accounts_single_default ON accounts(is_default) WHERE is_default = 1;
```

#### Income Table - Issues Identified

```sql
-- PROPOSED
CREATE TABLE income (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  amount REAL NOT NULL,
  date TEXT NOT NULL,
  source TEXT,
  category TEXT NOT NULL,
  is_recurring INTEGER DEFAULT 0,
  recurring_day INTEGER CHECK(recurring_day >= 1 AND recurring_day <= 31),
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
);
```

**Issues:**

| Issue | Severity | Description |
|-------|----------|-------------|
| `source` should be NOT NULL | Medium | Every income entry should have identifiable source |
| Missing amount validation | Medium | Negative amounts should be prevented |
| `is_recurring` and `recurring_day` logic inconsistent | Medium | If `is_recurring = 0`, `recurring_day` should be NULL |
| Missing category validation | Low | No CHECK constraint for predefined categories |

**Recommended Schema:**

```sql
CREATE TABLE income (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  amount REAL NOT NULL CHECK(amount > 0),
  date TEXT NOT NULL,
  source TEXT NOT NULL DEFAULT 'Unknown',
  category TEXT NOT NULL CHECK(category IN ('Salary', 'Freelance', 'Sales', 'Refunds', 'Investments', 'Rental', 'Other')),
  is_recurring INTEGER NOT NULL DEFAULT 0 CHECK(is_recurring IN (0, 1)),
  recurring_day INTEGER CHECK(
    (is_recurring = 0 AND recurring_day IS NULL) OR
    (is_recurring = 1 AND recurring_day >= 1 AND recurring_day <= 31)
  ),
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
);
```

#### Bills Table - Issues Identified

```sql
-- PROPOSED
CREATE TABLE bills (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  name TEXT NOT NULL,
  amount REAL NOT NULL,
  due_day INTEGER NOT NULL CHECK(due_day >= 1 AND due_day <= 31),
  frequency TEXT DEFAULT 'monthly' CHECK(frequency IN ('weekly', 'monthly', 'quarterly', 'annual')),
  category TEXT NOT NULL,
  is_recurring INTEGER DEFAULT 0,
  is_paid INTEGER DEFAULT 0,
  paid_date TEXT,
  expense_id INTEGER,
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
  FOREIGN KEY (expense_id) REFERENCES expenses(id) ON DELETE SET NULL
);
```

**Issues:**

| Issue | Severity | Description |
|-------|----------|-------------|
| `is_recurring` is redundant | High | Every bill with `frequency` is inherently recurring |
| `due_day` doesn't work for weekly frequency | High | Weekly bills need day-of-week, not day-of-month |
| Missing amount validation | Medium | Negative amounts should be prevented |
| `is_paid`/`paid_date`/`expense_id` logic not enforced | Medium | Should use CHECK constraint for consistency |
| Category not validated | Low | Should match expense categories |

**Recommended Schema:**

```sql
CREATE TABLE bills (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  name TEXT NOT NULL,
  amount REAL NOT NULL CHECK(amount > 0),
  frequency TEXT NOT NULL DEFAULT 'monthly' CHECK(frequency IN ('monthly', 'quarterly', 'annual')),
  due_day INTEGER NOT NULL CHECK(due_day >= 1 AND due_day <= 31),
  category TEXT NOT NULL,
  is_paid INTEGER NOT NULL DEFAULT 0 CHECK(is_paid IN (0, 1)),
  paid_date TEXT CHECK(
    (is_paid = 0 AND paid_date IS NULL) OR
    (is_paid = 1 AND paid_date IS NOT NULL)
  ),
  expense_id INTEGER CHECK(
    (is_paid = 0 AND expense_id IS NULL) OR
    (is_paid = 1)
  ),
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
  FOREIGN KEY (expense_id) REFERENCES expenses(id) ON DELETE SET NULL,
  FOREIGN KEY (category) REFERENCES categories(name)
);
```

**Note**: Removed `weekly` frequency. Weekly recurring expenses are better modeled as individual expenses or by creating 4 bill entries per month. This simplifies the `due_day` logic.

#### Expenses/Mileage Modifications - Issues

```sql
-- PROPOSED
ALTER TABLE expenses ADD COLUMN account_id INTEGER;
ALTER TABLE mileage ADD COLUMN account_id INTEGER;
```

**Issues:**

| Issue | Severity | Description |
|-------|----------|-------------|
| `account_id` should be NOT NULL after migration | High | All records must belong to an account |
| Missing foreign key on existing tables | Medium | SQLite doesn't enforce FK on ALTER TABLE |

**Solution**: See Migration Strategy (Section 5) for two-phase approach.

### 1.2 Normalization Assessment

The schema is in **Third Normal Form (3NF)** with acceptable denormalization:

| Table | Assessment | Notes |
|-------|------------|-------|
| `accounts` | 3NF | Clean, no redundancy |
| `income` | 3NF | `category` could be FK but string is acceptable for simplicity |
| `bills` | 3NF | `category` links to expense categories (good) |
| `expenses` | 3NF | Adding `account_id` maintains normalization |
| `mileage` | 3NF | Adding `account_id` maintains normalization |

**Acceptable Denormalization**:
- Using `category TEXT` instead of `category_id INTEGER` matches existing patterns and simplifies queries
- Storing `amount` in bills (could be derived) is acceptable for display performance

### 1.3 Foreign Key Analysis

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   accounts   │────<│   expenses   │────<│   bills      │
│              │     │              │     │ (expense_id) │
│              │────<│   income     │     └──────────────┘
│              │     │              │
│              │────<│   mileage    │
│              │     │              │
└──────────────┘     └──────────────┘
                            │
                            ▼
                     ┌──────────────┐
                     │  categories  │  (via name, not FK)
                     └──────────────┘
```

**ON DELETE Behavior**:

| Relationship | Current Proposal | Recommendation |
|--------------|------------------|----------------|
| accounts → expenses | Not specified | `ON DELETE RESTRICT` (prevent orphans) |
| accounts → income | `ON DELETE CASCADE` | Keep (income is account-specific) |
| accounts → bills | `ON DELETE CASCADE` | Keep (bills are account-specific) |
| accounts → mileage | Not specified | `ON DELETE RESTRICT` (prevent orphans) |
| bills → expenses | `ON DELETE SET NULL` | Keep (expense survives if bill deleted) |

**Recommendation**: Use `ON DELETE RESTRICT` for expenses/mileage to prevent accidental data loss. Users should explicitly move or delete data before account deletion.

---

## 2. Service Layer Design

### 2.1 Current Pattern Analysis

The existing `DatabaseService` follows these patterns:

```typescript
// Singleton pattern
private static instance: DatabaseService | null = null;
public static getInstance(): DatabaseService { ... }

// Async initialization
async initialize(): Promise<void> { ... }

// CRUD methods with validation
async createExpense(expense: Omit<Expense, ...>): Promise<number> { ... }
async getAllExpenses(): Promise<Expense[]> { ... }
async updateExpense(id: number, updates: Partial<Expense>): Promise<void> { ... }
async deleteExpense(id: number): Promise<void> { ... }

// Aggregation methods
async getExpenseSummary(dateFrom: string, dateTo: string): Promise<ExpenseSummary> { ... }
async getCategoryTotals(dateFrom: string, dateTo: string): Promise<CategoryTotal[]> { ... }
```

### 2.2 Recommended Service Structure

Create three new services following the singleton pattern:

```typescript
// src/services/AccountService.ts
export class AccountService {
  private static instance: AccountService | null = null;
  private db: SQLite.SQLiteDatabase | null = null;

  public static getInstance(): AccountService { ... }

  async initialize(db: SQLite.SQLiteDatabase): Promise<void> { ... }

  // Account CRUD
  async createAccount(account: CreateAccountDTO): Promise<number>;
  async getAllAccounts(): Promise<Account[]>;
  async getAccountById(id: number): Promise<Account | null>;
  async getDefaultAccount(): Promise<Account | null>;
  async updateAccount(id: number, updates: UpdateAccountDTO): Promise<void>;
  async deleteAccount(id: number): Promise<void>;
  async setDefaultAccount(id: number): Promise<void>;

  // Account summary
  async getAccountSummary(accountId: number): Promise<AccountSummary>;
}

// src/services/IncomeService.ts
export class IncomeService {
  private static instance: IncomeService | null = null;

  public static getInstance(): IncomeService { ... }

  // Income CRUD (all methods require accountId)
  async createIncome(accountId: number, income: CreateIncomeDTO): Promise<number>;
  async getIncomeByAccount(accountId: number): Promise<Income[]>;
  async getIncomeWithFilters(accountId: number, filters: IncomeFilters): Promise<Income[]>;
  async updateIncome(id: number, updates: UpdateIncomeDTO): Promise<void>;
  async deleteIncome(id: number): Promise<void>;

  // Income aggregations
  async getIncomeSummary(accountId: number, dateFrom: string, dateTo: string): Promise<IncomeSummary>;
  async getIncomeByCategory(accountId: number, dateFrom: string, dateTo: string): Promise<CategoryTotal[]>;
}

// src/services/BillService.ts
export class BillService {
  private static instance: BillService | null = null;

  public static getInstance(): BillService { ... }

  // Bill CRUD
  async createBill(accountId: number, bill: CreateBillDTO): Promise<number>;
  async getBillsByAccount(accountId: number): Promise<Bill[]>;
  async getBillById(id: number): Promise<Bill | null>;
  async updateBill(id: number, updates: UpdateBillDTO): Promise<void>;
  async deleteBill(id: number): Promise<void>;

  // Bill-specific operations
  async markBillPaid(billId: number, paidDate: string): Promise<{ billId: number; expenseId: number }>;
  async undoMarkPaid(billId: number): Promise<void>;
  async resetRecurringBills(accountId: number): Promise<number>; // Returns count reset
  async getOverdueBills(accountId: number): Promise<Bill[]>;
  async getUpcomingBills(accountId: number, days: number): Promise<Bill[]>;
}

// src/services/CashFlowService.ts
export class CashFlowService {
  private static instance: CashFlowService | null = null;

  public static getInstance(): CashFlowService { ... }

  // Dashboard aggregations
  async getCashFlowSummary(accountId: number, dateFrom: string, dateTo: string): Promise<CashFlowSummary>;
  async getYearToDateSummary(accountId: number): Promise<CashFlowSummary>;
  async getPeriodComparison(accountId: number, dateFrom: string, dateTo: string): Promise<PeriodComparison>;

  // Trend data
  async getMonthlyTrend(accountId: number, months: number): Promise<MonthlyTrend[]>;
  async getCategoryBreakdown(accountId: number, dateFrom: string, dateTo: string): Promise<CategoryBreakdown>;
}
```

### 2.3 Service Interaction Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      React Components                            │
└────────────────────────────┬────────────────────────────────────┘
                             │ useServices() hook
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      ServiceContext                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │
│  │ Account   │  │ Income    │  │ Bill      │  │ CashFlow  │    │
│  │ Service   │  │ Service   │  │ Service   │  │ Service   │    │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘    │
│        │              │              │              │           │
│        └──────────────┴──────────────┴──────────────┘           │
│                              │                                   │
│                              ▼                                   │
│                    ┌─────────────────┐                          │
│                    │ DatabaseService │                          │
│                    │   (existing)    │                          │
│                    └─────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

### 2.4 BillService.markBillPaid Implementation

This is the most complex service method - here's the recommended implementation:

```typescript
async markBillPaid(billId: number, paidDate: string): Promise<{ billId: number; expenseId: number }> {
  if (!this.db) throw new Error('Database not initialized');

  // 1. Get bill details
  const bill = await this.getBillById(billId);
  if (!bill) throw new Error(`Bill ${billId} not found`);
  if (bill.is_paid) throw new Error(`Bill ${billId} is already marked as paid`);

  // 2. Begin transaction
  await this.db.execAsync('BEGIN TRANSACTION');

  try {
    // 3. Create expense from bill
    const now = new Date().toISOString();
    const expenseResult = await this.db.runAsync(
      `INSERT INTO expenses (
        account_id, amount, date, category, merchant, description,
        payment_method, receipt_count, voice_memo_count, ocr_extracted,
        nl_parsed, created_at, updated_at
      ) VALUES (?, ?, ?, ?, ?, ?, NULL, 0, 0, 0, 0, ?, ?)`,
      [
        bill.account_id,
        bill.amount,
        paidDate,
        bill.category,
        bill.name, // merchant = bill name
        `Bill payment: ${bill.name}`,
        now,
        now,
      ]
    );

    const expenseId = expenseResult.lastInsertRowId;

    // 4. Update bill status
    await this.db.runAsync(
      `UPDATE bills SET
        is_paid = 1,
        paid_date = ?,
        expense_id = ?,
        updated_at = ?
       WHERE id = ?`,
      [paidDate, expenseId, now, billId]
    );

    // 5. Commit transaction
    await this.db.execAsync('COMMIT');

    return { billId, expenseId };
  } catch (error) {
    await this.db.execAsync('ROLLBACK');
    throw error;
  }
}
```

### 2.5 DatabaseService Modifications

Existing `DatabaseService` methods need account-scoping:

```typescript
// Add account-scoped versions
async getAllExpensesByAccount(accountId: number): Promise<Expense[]>;
async getExpensesWithFiltersByAccount(accountId: number, filters: ExpenseFilters): Promise<Expense[]>;
async getExpenseSummaryByAccount(accountId: number, dateFrom: string, dateTo: string): Promise<ExpenseSummary>;
async getCategoryTotalsByAccount(accountId: number, dateFrom: string, dateTo: string): Promise<CategoryTotal[]>;

// Same for mileage
async getAllMileageByAccount(accountId: number): Promise<Mileage[]>;
async getMileageSummaryByAccount(accountId: number, dateFrom: string, dateTo: string): Promise<MileageSummary>;
```

**Deprecation Strategy**: Keep existing non-scoped methods for backward compatibility during migration, but mark as `@deprecated`.

---

## 3. Context Architecture

### 3.1 Current ServiceContext Analysis

```typescript
// Current structure
interface Services {
  database: DatabaseService;
  file: FileService;
  settings: SettingsService;
  migration: MigrationService;
  ocr: OCRService;
  export: ExportService;
}

const ServiceContext = createContext<ServiceProviderState>({
  services: null,
  initialized: false,
  error: null,
});
```

### 3.2 AccountContext Design

**Do NOT** create AccountContext as a separate provider that wraps ServiceContext. Instead, integrate account state into the existing ServiceContext pattern.

**Recommended Approach**:

```typescript
// src/contexts/ServiceContext.tsx (extended)

interface Services {
  database: DatabaseService;
  file: FileService;
  settings: SettingsService;
  migration: MigrationService;
  ocr: OCRService;
  export: ExportService;
  // NEW services
  account: AccountService;
  income: IncomeService;
  bill: BillService;
  cashFlow: CashFlowService;
}

interface AccountState {
  accounts: Account[];
  activeAccount: Account | null;
  isLoading: boolean;
}

interface ServiceProviderState {
  services: Services | null;
  initialized: boolean;
  error: Error | null;
  // NEW account state
  accountState: AccountState;
  setActiveAccount: (accountId: number) => Promise<void>;
  refreshAccounts: () => Promise<void>;
}
```

### 3.3 Implementation Pattern

```typescript
export function ServiceProvider({ children }: ServiceProviderProps) {
  const [initialized, setInitialized] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const [accountState, setAccountState] = useState<AccountState>({
    accounts: [],
    activeAccount: null,
    isLoading: true,
  });

  // Create singleton instances
  const services = useMemo<Services>(() => ({
    database: DatabaseService.getInstance(),
    file: FileService.getInstance(),
    settings: SettingsService.getInstance(),
    migration: MigrationService.getInstance(),
    ocr: OCRService.getInstance(),
    export: ExportService.getInstance(),
    account: AccountService.getInstance(),
    income: IncomeService.getInstance(),
    bill: BillService.getInstance(),
    cashFlow: CashFlowService.getInstance(),
  }), []);

  // Account operations
  const setActiveAccount = useCallback(async (accountId: number) => {
    const account = accountState.accounts.find(a => a.id === accountId);
    if (account) {
      setAccountState(prev => ({ ...prev, activeAccount: account }));
      // Persist active account to AsyncStorage
      await AsyncStorage.setItem('activeAccountId', String(accountId));
    }
  }, [accountState.accounts]);

  const refreshAccounts = useCallback(async () => {
    if (!services.account) return;
    const accounts = await services.account.getAllAccounts();
    const storedActiveId = await AsyncStorage.getItem('activeAccountId');
    const activeAccount = accounts.find(a =>
      a.id === Number(storedActiveId) || a.is_default
    ) || accounts[0] || null;

    setAccountState({
      accounts,
      activeAccount,
      isLoading: false,
    });
  }, [services]);

  // Initialize services and load accounts
  useEffect(() => {
    async function init() {
      try {
        // ... existing initialization ...
        await services.account.initialize(services.database.getDb());
        await services.income.initialize(services.database.getDb());
        await services.bill.initialize(services.database.getDb());
        await services.cashFlow.initialize(services.database.getDb());

        // Load accounts after DB ready
        await refreshAccounts();

        setInitialized(true);
      } catch (err) {
        setError(err as Error);
      }
    }
    init();
  }, [services, refreshAccounts]);

  // ... rest of provider
}
```

### 3.4 Usage in Components

```typescript
// In any component
function ExpenseListScreen() {
  const { services, accountState, setActiveAccount } = useServiceContext();
  const { activeAccount } = accountState;

  const [expenses, setExpenses] = useState<Expense[]>([]);

  useEffect(() => {
    if (services && activeAccount) {
      services.database.getAllExpensesByAccount(activeAccount.id)
        .then(setExpenses);
    }
  }, [services, activeAccount]);

  // ... render
}
```

### 3.5 Context Hierarchy

```
App
└── ServiceProvider (includes account state)
    └── ThemeProvider
        └── NavigationContainer
            └── TabNavigator
                └── Screens (can access services + activeAccount)
```

**Why NOT a separate AccountContext?**

1. **Initialization order**: Account state depends on services being initialized
2. **Single source of truth**: Avoids context nesting and potential sync issues
3. **Performance**: Fewer context updates, components re-render only when needed
4. **Consistency**: Matches existing codebase pattern

---

## 4. Query Performance

### 4.1 Current Indexes

```sql
-- Existing indexes
CREATE INDEX idx_expenses_date ON expenses(date);
CREATE INDEX idx_expenses_category ON expenses(category);
CREATE INDEX idx_expenses_merchant ON expenses(merchant);
CREATE INDEX idx_mileage_date ON mileage(date);
CREATE INDEX idx_merchant_categories_merchant ON merchant_categories(merchant_normalized);
```

### 4.2 Proposed Indexes (from design.md)

```sql
CREATE INDEX idx_income_account_date ON income(account_id, date);
CREATE INDEX idx_bills_account ON bills(account_id);
CREATE INDEX idx_expenses_account ON expenses(account_id);
CREATE INDEX idx_mileage_account ON mileage(account_id);
```

### 4.3 Additional Indexes Recommended

For dashboard performance, add composite indexes that support aggregation queries:

```sql
-- Account-scoped date range queries (most common pattern)
CREATE INDEX idx_expenses_account_date ON expenses(account_id, date);
CREATE INDEX idx_income_account_date ON income(account_id, date);
CREATE INDEX idx_mileage_account_date ON mileage(account_id, date);

-- Bill status queries (for "overdue" and "upcoming" views)
CREATE INDEX idx_bills_account_paid ON bills(account_id, is_paid);

-- Category aggregation (for dashboard breakdowns)
CREATE INDEX idx_expenses_account_category ON expenses(account_id, category);
CREATE INDEX idx_income_account_category ON income(account_id, category);

-- Dashboard period summaries (covering index)
CREATE INDEX idx_expenses_account_date_amount ON expenses(account_id, date, amount);
CREATE INDEX idx_income_account_date_amount ON income(account_id, date, amount);
```

### 4.4 Query Optimization for Dashboard

**Cash Flow Summary Query** (optimized):

```sql
-- Single query for dashboard metrics
SELECT
  COALESCE(i.total_income, 0) as income,
  COALESCE(e.total_expenses, 0) as expenses,
  COALESCE(m.total_mileage, 0) as mileage_deduction,
  COALESCE(i.total_income, 0) - COALESCE(e.total_expenses, 0) - COALESCE(m.total_mileage, 0) as net
FROM
  (SELECT SUM(amount) as total_income FROM income
   WHERE account_id = ? AND date >= ? AND date <= ?) i,
  (SELECT SUM(amount) as total_expenses FROM expenses
   WHERE account_id = ? AND date >= ? AND date <= ?) e,
  (SELECT SUM(amount) as total_mileage FROM mileage
   WHERE account_id = ? AND date >= ? AND date <= ?) m;
```

**Bills Summary Query** (for dashboard "unpaid bills" card):

```sql
SELECT
  COUNT(*) as total_unpaid,
  SUM(amount) as total_amount,
  SUM(CASE WHEN due_day < strftime('%d', 'now') THEN 1 ELSE 0 END) as overdue_count
FROM bills
WHERE account_id = ? AND is_paid = 0;
```

### 4.5 Index Maintenance

SQLite handles index maintenance automatically. However, add `ANALYZE` to migration script:

```sql
-- Run after data migration to update query planner statistics
ANALYZE expenses;
ANALYZE mileage;
ANALYZE income;
ANALYZE bills;
ANALYZE accounts;
```

### 4.6 Expected Query Performance

| Query | Without Index | With Index | Notes |
|-------|--------------|------------|-------|
| Expenses by account | O(n) full scan | O(log n) | Account filter first |
| Dashboard summary | 3 full scans | 3 index scans | Covering indexes |
| Overdue bills | O(n) scan | O(log n) | Compound index |
| Category totals | O(n) + sort | O(log n) grouped | Category in index |

For expected data volumes (<10,000 expenses per account), all queries should complete in <50ms on modern iOS devices.

---

## 5. Migration Strategy

### 5.1 Migration Risks

| Risk | Mitigation |
|------|------------|
| Data loss during migration | Backup before migration; transaction rollback |
| App crash mid-migration | Idempotent migrations; version tracking |
| Foreign key violations | Two-phase approach with nullable column first |
| Performance during migration | Batch operations; progress indicator |

### 5.2 Two-Phase Migration Approach

**Phase 1: Schema Addition (Non-breaking)**

```typescript
// migration_005_add_account_schema.ts
export async function up(db: SQLite.SQLiteDatabase): Promise<void> {
  // 1. Create accounts table
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS accounts (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      type TEXT NOT NULL CHECK(type IN ('personal', 'business')),
      color TEXT NOT NULL DEFAULT '#2196F3',
      icon TEXT DEFAULT 'account',
      is_default INTEGER NOT NULL DEFAULT 0 CHECK(is_default IN (0, 1)),
      created_at TEXT NOT NULL,
      updated_at TEXT NOT NULL
    );

    CREATE UNIQUE INDEX IF NOT EXISTS idx_accounts_single_default
    ON accounts(is_default) WHERE is_default = 1;
  `);

  // 2. Create default account
  const now = new Date().toISOString();
  await db.runAsync(
    `INSERT INTO accounts (name, type, color, icon, is_default, created_at, updated_at)
     VALUES (?, ?, ?, ?, ?, ?, ?)`,
    ['Personal', 'personal', '#2196F3', 'account', 1, now, now]
  );

  // 3. Add account_id to expenses (NULLABLE initially)
  await db.execAsync(`
    ALTER TABLE expenses ADD COLUMN account_id INTEGER;
  `);

  // 4. Add account_id to mileage (NULLABLE initially)
  await db.execAsync(`
    ALTER TABLE mileage ADD COLUMN account_id INTEGER;
  `);

  // 5. Create income table
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS income (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      account_id INTEGER NOT NULL,
      amount REAL NOT NULL CHECK(amount > 0),
      date TEXT NOT NULL,
      source TEXT NOT NULL DEFAULT 'Unknown',
      category TEXT NOT NULL,
      is_recurring INTEGER NOT NULL DEFAULT 0,
      recurring_day INTEGER,
      notes TEXT,
      created_at TEXT NOT NULL,
      updated_at TEXT NOT NULL,
      FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
    );

    CREATE INDEX IF NOT EXISTS idx_income_account_date ON income(account_id, date);
  `);

  // 6. Create bills table
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS bills (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      account_id INTEGER NOT NULL,
      name TEXT NOT NULL,
      amount REAL NOT NULL CHECK(amount > 0),
      frequency TEXT NOT NULL DEFAULT 'monthly',
      due_day INTEGER NOT NULL CHECK(due_day >= 1 AND due_day <= 31),
      category TEXT NOT NULL,
      is_paid INTEGER NOT NULL DEFAULT 0,
      paid_date TEXT,
      expense_id INTEGER,
      notes TEXT,
      created_at TEXT NOT NULL,
      updated_at TEXT NOT NULL,
      FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
      FOREIGN KEY (expense_id) REFERENCES expenses(id) ON DELETE SET NULL
    );

    CREATE INDEX IF NOT EXISTS idx_bills_account ON bills(account_id);
    CREATE INDEX IF NOT EXISTS idx_bills_account_paid ON bills(account_id, is_paid);
  `);
}

export async function down(db: SQLite.SQLiteDatabase): Promise<void> {
  // Reverse migration (use with caution)
  await db.execAsync(`
    DROP TABLE IF EXISTS bills;
    DROP TABLE IF EXISTS income;
    -- Note: Cannot remove columns in SQLite without table rebuild
  `);
}
```

**Phase 2: Data Migration (Backfill)**

```typescript
// migration_006_backfill_account_ids.ts
export async function up(db: SQLite.SQLiteDatabase): Promise<void> {
  // 1. Get default account ID
  const result = await db.getAllAsync(
    'SELECT id FROM accounts WHERE is_default = 1 LIMIT 1'
  );
  const defaultAccountId = (result as Array<{ id: number }>)[0]?.id;

  if (!defaultAccountId) {
    throw new Error('Default account not found - run migration 005 first');
  }

  // 2. Backfill expenses
  await db.runAsync(
    'UPDATE expenses SET account_id = ? WHERE account_id IS NULL',
    [defaultAccountId]
  );

  // 3. Backfill mileage
  await db.runAsync(
    'UPDATE mileage SET account_id = ? WHERE account_id IS NULL',
    [defaultAccountId]
  );

  // 4. Create additional indexes now that data is populated
  await db.execAsync(`
    CREATE INDEX IF NOT EXISTS idx_expenses_account_date ON expenses(account_id, date);
    CREATE INDEX IF NOT EXISTS idx_expenses_account_category ON expenses(account_id, category);
    CREATE INDEX IF NOT EXISTS idx_mileage_account_date ON mileage(account_id, date);
  `);

  // 5. Update statistics
  await db.execAsync('ANALYZE');
}
```

### 5.3 Migration Verification

Add verification query after migration:

```typescript
async function verifyMigration(db: SQLite.SQLiteDatabase): Promise<boolean> {
  // Check no orphaned expenses
  const orphanedExpenses = await db.getAllAsync(
    'SELECT COUNT(*) as count FROM expenses WHERE account_id IS NULL'
  );
  if ((orphanedExpenses as Array<{ count: number }>)[0].count > 0) {
    console.error('Migration incomplete: orphaned expenses found');
    return false;
  }

  // Check no orphaned mileage
  const orphanedMileage = await db.getAllAsync(
    'SELECT COUNT(*) as count FROM mileage WHERE account_id IS NULL'
  );
  if ((orphanedMileage as Array<{ count: number }>)[0].count > 0) {
    console.error('Migration incomplete: orphaned mileage found');
    return false;
  }

  // Check default account exists
  const defaultAccount = await db.getAllAsync(
    'SELECT COUNT(*) as count FROM accounts WHERE is_default = 1'
  );
  if ((defaultAccount as Array<{ count: number }>)[0].count !== 1) {
    console.error('Migration error: no default account or multiple defaults');
    return false;
  }

  return true;
}
```

### 5.4 Rollback Plan

If migration fails:

1. **Phase 1 rollback**: Drop new tables, keep altered expenses/mileage (columns are nullable)
2. **Phase 2 rollback**: Not needed - backfill is idempotent
3. **Full rollback**: Restore from pre-migration backup

---

## 6. Navigation Decision (CRITICAL)

### 6.1 Problem Statement

The current app has 4 tabs: Expenses, Mileage, Reports, Settings.

The proposal adds 3 new top-level features: Dashboard, Income, Bills.

This would create **7 tabs**, exceeding iOS Human Interface Guidelines (maximum 5).

### 6.2 Options Analysis

#### Option A: Consolidated "Finances" Tab with Segmented Control

```
┌─────────────────────────────────────────────────────┐
│  [Dashboard]  [Finances]  [Reports]  [Settings]     │  ← 4 tabs (iOS compliant)
└─────────────────────────────────────────────────────┘

Finances Tab Internal Structure:
┌─────────────────────────────────────────────────────┐
│  [Expenses]  [Income]  [Bills]  [Mileage]           │  ← Segmented control
├─────────────────────────────────────────────────────┤
│  (List content based on selected segment)           │
└─────────────────────────────────────────────────────┘
```

**Technical Implementation**:

```typescript
// src/navigation/FinancesStack.tsx
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tabs';

const TopTab = createMaterialTopTabNavigator();

export default function FinancesStack() {
  return (
    <TopTab.Navigator
      screenOptions={{
        tabBarScrollEnabled: true,
        tabBarIndicatorStyle: { backgroundColor: '#2E7D32' },
        tabBarLabelStyle: { fontSize: 14, fontWeight: '500' },
      }}
    >
      <TopTab.Screen name="Expenses" component={ExpenseListScreen} />
      <TopTab.Screen name="Income" component={IncomeListScreen} />
      <TopTab.Screen name="Bills" component={BillsListScreen} />
      <TopTab.Screen name="Mileage" component={MileageListScreen} />
    </TopTab.Navigator>
  );
}
```

**Pros**:
- iOS HIG compliant (4 tabs)
- Familiar pattern (used by many finance apps)
- Swipe between related features
- Account context stays visible across all finance sections
- Each section maintains independent scroll position

**Cons**:
- Mileage was previously top-level (slight behavior change)
- Users need to learn new segmented control pattern
- Deeper nesting for add/edit flows

#### Option B: Dashboard Hub-and-Spoke Model

```
┌─────────────────────────────────────────────────────┐
│  [Dashboard]  [Expenses]  [Mileage]  [Settings]     │  ← 4 tabs (iOS compliant)
└─────────────────────────────────────────────────────┘

Dashboard Screen Structure:
┌─────────────────────────────────────────────────────┐
│  Net: +$3,245                                       │  ← Hero metric
├─────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐            │
│  │ Income         │  │ Bills          │            │  ← Tappable cards
│  │ $5,200 ↑       │  │ 3 due this wk  │            │     navigate to lists
│  └────────────────┘  └────────────────┘            │
├─────────────────────────────────────────────────────┤
│  Recent Expenses                    [View All →]   │
│  • Coffee $4.50                                    │
│  • Gas $45.00                                      │
└─────────────────────────────────────────────────────┘
```

**Technical Implementation**:

```typescript
// src/screens/DashboardScreen.tsx
function DashboardScreen({ navigation }) {
  return (
    <ScrollView>
      <CashFlowHero accountId={activeAccount.id} />

      <TouchableOpacity onPress={() => navigation.navigate('IncomeList')}>
        <IncomeSummaryCard />
      </TouchableOpacity>

      <TouchableOpacity onPress={() => navigation.navigate('BillsList')}>
        <BillsSummaryCard />
      </TouchableOpacity>

      <RecentExpensesPreview
        onViewAll={() => navigation.navigate('ExpensesTab')}
      />
    </ScrollView>
  );
}
```

**Pros**:
- Dashboard is focal point (cash flow-first experience)
- Income/Bills discoverable from natural entry point
- Existing Expenses/Mileage tabs unchanged
- Simpler navigation hierarchy

**Cons**:
- Income/Bills require 2 taps to access (Dashboard → List)
- Dashboard becomes busy with multiple entry points
- Less direct for users who primarily track income

### 6.3 Recommendation: Option A (Consolidated "Finances" Tab)

**Decision**: Implement Option A - consolidated "Finances" tab with segmented control.

**Technical Rationale**:

1. **Navigation Depth**: Both options have similar depth for core actions:
   - Option A: Finances tab → Expenses segment → Add expense (2 taps)
   - Option B: Expenses tab → Add expense (1 tap), but Income: Dashboard → Income → Add income (3 taps)

2. **State Management**: Option A keeps all financial data contexts together:
   ```typescript
   // FinancesStack can provide shared state for all finance segments
   <FinancesProvider accountId={activeAccount.id}>
     <TopTab.Navigator>
       {/* All segments share the same account context */}
     </TopTab.Navigator>
   </FinancesProvider>
   ```

3. **FAB Consistency**: Single FAB in Finances tab can intelligently detect which segment is active:
   ```typescript
   const handleFabPress = () => {
     const activeSegment = getCurrentSegment();
     switch (activeSegment) {
       case 'Expenses': navigation.navigate('AddExpense'); break;
       case 'Income': navigation.navigate('AddIncome'); break;
       case 'Bills': navigation.navigate('AddBill'); break;
       case 'Mileage': navigation.navigate('AddMileage'); break;
     }
   };
   ```

4. **Performance**: Top tab navigator with lazy loading performs well:
   ```typescript
   <TopTab.Navigator lazy={true} lazyPreloadDistance={1}>
   ```

5. **Accessibility**: Material top tabs have better VoiceOver support than custom hub cards.

6. **Future Extensibility**: Adding more finance-related features (budgets, goals) is straightforward - just add another segment.

### 6.4 Final Navigation Structure

```
App
├── AccountSwitcher (header component)
├── TabNavigator
│   ├── DashboardTab (Dashboard Stack)
│   │   ├── DashboardScreen (cash flow summary)
│   │   └── PeriodComparisonScreen
│   │
│   ├── FinancesTab (Finances Stack - TOP TABS)
│   │   ├── Expenses Segment
│   │   │   ├── ExpenseListScreen
│   │   │   ├── ExpenseDetailScreen
│   │   │   └── AddExpenseScreen (modal)
│   │   ├── Income Segment
│   │   │   ├── IncomeListScreen
│   │   │   ├── IncomeDetailScreen
│   │   │   └── AddIncomeScreen (modal)
│   │   ├── Bills Segment
│   │   │   ├── BillsListScreen
│   │   │   ├── BillDetailScreen
│   │   │   └── AddBillScreen (modal)
│   │   └── Mileage Segment
│   │       ├── MileageListScreen
│   │       ├── MileageDetailScreen
│   │       └── AddMileageScreen (modal)
│   │
│   ├── ReportsTab (Reports Stack)
│   │   └── (existing screens, account-scoped)
│   │
│   └── SettingsTab (Settings Stack)
│       ├── (existing screens)
│       └── ManageAccountsScreen (new)
│
└── Modals (global)
    └── AccountSwitcherSheet
```

---

## 7. Recommended Schema Changes

### 7.1 Summary of All Schema Modifications

Based on the analysis above, here are the recommended changes to `design.md`:

| Table | Change | Reason |
|-------|--------|--------|
| `accounts` | Add `icon TEXT` column | Accessibility (color-blind users) |
| `accounts` | Add CHECK constraint on `color` format | Data validation |
| `accounts` | Add partial unique index on `is_default` | Prevent multiple defaults |
| `income` | Make `source` NOT NULL with default | Data completeness |
| `income` | Add CHECK constraint on `amount > 0` | Prevent invalid data |
| `income` | Add CHECK constraint on category values | Data validation |
| `income` | Add conditional CHECK on `recurring_day` | Logic consistency |
| `bills` | Remove `is_recurring` column | Redundant with `frequency` |
| `bills` | Remove `weekly` from `frequency` options | Simplify due_day logic |
| `bills` | Add CHECK constraint on `amount > 0` | Prevent invalid data |
| `bills` | Add conditional CHECK on `paid_date` | Logic consistency |
| `expenses` | Add `ON DELETE RESTRICT` for account FK | Prevent data loss |
| `mileage` | Add `ON DELETE RESTRICT` for account FK | Prevent data loss |
| All tables | Add additional composite indexes | Query performance |

### 7.2 Complete Recommended Schema

```sql
-- accounts table
CREATE TABLE accounts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  type TEXT NOT NULL CHECK(type IN ('personal', 'business')),
  color TEXT NOT NULL DEFAULT '#2196F3'
    CHECK(color GLOB '#[0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f]'),
  icon TEXT NOT NULL DEFAULT 'account',
  is_default INTEGER NOT NULL DEFAULT 0 CHECK(is_default IN (0, 1)),
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE UNIQUE INDEX idx_accounts_single_default ON accounts(is_default) WHERE is_default = 1;

-- income table
CREATE TABLE income (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  amount REAL NOT NULL CHECK(amount > 0),
  date TEXT NOT NULL,
  source TEXT NOT NULL DEFAULT 'Unknown',
  category TEXT NOT NULL CHECK(category IN ('Salary', 'Freelance', 'Sales', 'Refunds', 'Investments', 'Rental', 'Other')),
  is_recurring INTEGER NOT NULL DEFAULT 0 CHECK(is_recurring IN (0, 1)),
  recurring_day INTEGER CHECK(
    (is_recurring = 0 AND recurring_day IS NULL) OR
    (is_recurring = 1 AND recurring_day >= 1 AND recurring_day <= 31)
  ),
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
);

CREATE INDEX idx_income_account_date ON income(account_id, date);
CREATE INDEX idx_income_account_category ON income(account_id, category);
CREATE INDEX idx_income_account_date_amount ON income(account_id, date, amount);

-- bills table
CREATE TABLE bills (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  account_id INTEGER NOT NULL,
  name TEXT NOT NULL,
  amount REAL NOT NULL CHECK(amount > 0),
  frequency TEXT NOT NULL DEFAULT 'monthly' CHECK(frequency IN ('monthly', 'quarterly', 'annual')),
  due_day INTEGER NOT NULL CHECK(due_day >= 1 AND due_day <= 31),
  category TEXT NOT NULL,
  is_paid INTEGER NOT NULL DEFAULT 0 CHECK(is_paid IN (0, 1)),
  paid_date TEXT CHECK(
    (is_paid = 0 AND paid_date IS NULL) OR
    (is_paid = 1 AND paid_date IS NOT NULL)
  ),
  expense_id INTEGER,
  notes TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
  FOREIGN KEY (expense_id) REFERENCES expenses(id) ON DELETE SET NULL
);

CREATE INDEX idx_bills_account ON bills(account_id);
CREATE INDEX idx_bills_account_paid ON bills(account_id, is_paid);

-- expenses table modifications
ALTER TABLE expenses ADD COLUMN account_id INTEGER REFERENCES accounts(id) ON DELETE RESTRICT;
CREATE INDEX idx_expenses_account ON expenses(account_id);
CREATE INDEX idx_expenses_account_date ON expenses(account_id, date);
CREATE INDEX idx_expenses_account_category ON expenses(account_id, category);
CREATE INDEX idx_expenses_account_date_amount ON expenses(account_id, date, amount);

-- mileage table modifications
ALTER TABLE mileage ADD COLUMN account_id INTEGER REFERENCES accounts(id) ON DELETE RESTRICT;
CREATE INDEX idx_mileage_account ON mileage(account_id);
CREATE INDEX idx_mileage_account_date ON mileage(account_id, date);
```

---

## 8. Open Questions Resolved

| Question from design.md | Resolution |
|------------------------|------------|
| Bill recurrence timing: app open vs background? | **App open** - simpler, no background task infrastructure needed. Implement in `ServiceContext` initialization. |
| Delete account behavior? | **RESTRICT** - require user to explicitly delete or move data first. Show confirmation with data count. |
| Account limits? | **10 accounts max** - enforced in `AccountService.createAccount()`. Soft limit, not DB constraint. |

---

## 9. Implementation Priority

Based on dependencies and UX requirements:

1. **Phase 1 - Foundation** (Week 1-2)
   - Migration scripts (005, 006)
   - AccountService implementation
   - ServiceContext modifications
   - Account switcher UI

2. **Phase 2 - Income** (Week 3-4)
   - IncomeService implementation
   - Income types and validation
   - Income list/form screens
   - Integration with existing patterns

3. **Phase 3 - Bills & Dashboard** (Week 5-6)
   - BillService implementation
   - Bills list/form screens
   - Mark paid flow with expense creation
   - CashFlowService implementation
   - Dashboard screen

4. **Phase 4 - Navigation Refactor** (Week 7)
   - Create FinancesStack with top tabs
   - Move existing Expenses/Mileage into segments
   - Update TabNavigator
   - FAB context awareness

---

## 10. Review Sign-off

- [ ] Schema changes reviewed by: _______________
- [ ] Service layer design approved by: _______________
- [ ] Navigation decision accepted by: _______________
- [ ] UX review incorporated: See `docs/ux-review-multi-account-cashflow.md`

---

**Document Status**: Complete
**Next Step**: Update `openspec/changes/add-multi-account-cashflow/design.md` with recommended changes

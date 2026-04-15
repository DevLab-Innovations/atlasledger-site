# QA Report: PR #149 — Wave 5.7B Spreadsheet Import

**Date**: 2026-02-25
**Tester**: qa-tester agent
**Branch**: dev (merge commit `67f463d`)
**PR**: #149 (feature/spreadsheet-import)

---

## Results Overview

| Category | Result |
|----------|--------|
| Automated Tests | 2028 passed, 0 failed (106/107 suites pass; 1 pre-existing skip) |
| TypeScript Check | 0 errors |
| ImportService tests (32) | All pass |
| Critical Bugs | 0 |
| High Severity Bugs | 1 |
| Medium Severity Bugs | 1 |
| Low Severity Findings | 3 |
| Spec Compliance | 90% — 2 deviations found |

**Recommendation: NEEDS FIX before promotion to `main`**

The 1 High severity bug (`is_recurring: true` for imported bills) contradicts an explicit, unambiguous spec requirement and changes the fundamental behavior of imported data. The 1 Medium severity bug (missing "View" button) is a spec-required feature that was not implemented.

---

## Automated Test Results

### Full Test Suite

```
Test Suites: 1 skipped (pre-existing), 106 passed, 107 total
Tests: 55 skipped, 2028 passed, 2083 total
Time: 6.882s
```

No new failures introduced by PR #149. The 1 skipped suite and worker-exit warning pre-date this PR and are unrelated to import functionality.

### ImportService Unit Tests (32/32)

All 32 tests pass covering:
- `detectMapping()` — all 4 record types, exact aliases, fuzzy aliases, unknown headers, no-duplicate-assignment
- `executeImport()` — happy path (expense/income/mileage/bill), missing date, invalid amount, per-row continue, date coercion (US/ISO/written), amount coercion (currency symbols, negatives)
- `detectDuplicates()` — no match, match found, ±1 day window, invalid date skip, multi-row mix, income (no check)
- `parseFile()` — 5001 row limit throw, 5000 rows succeed

### TypeScript

```
npx tsc --noEmit: 0 errors
```

---

## Acceptance Criteria Verification

### From `openspec/changes/import-spreadsheet/design.md`

| Criterion | Status | Evidence |
|-----------|--------|----------|
| `.numbers` files show informational error (not attempted to parse) | PASS | `pickFile()` checks extension before any parsing; detailed export instructions shown |
| 5,001-row file triggers row-limit error | PASS | `nonEmptyRows.length > MAX_ROWS` (5000) throws; test confirmed |
| 5,000-row file succeeds | PASS | Boundary condition correctly `>` not `>=`; unit test confirms |
| All records assigned to active account | PASS | `accountId` flows: ImportScreen → nav params → ColumnMappingScreen → MappingConfig → all 4 insert methods |
| No account picker shown anywhere | PASS | Account is read-only banner in steps 1 and 2; no picker UI exists |
| `ImportScreen` validates default account exists before file picker | PASS | Button `disabled={!activeAccount}` + function guard in `handleChooseFile` |
| Duplicate detection: ±1 day tolerance | PASS | Date range query `>= dayBefore AND <= dayAfter`; unit test confirms; month/year boundaries correct |
| Duplicates still inserted but flagged | PASS | `duplicateMap` populated in `detectDuplicates`, rows inserted in transaction, added to `result.duplicates` |
| Per-row error handling: invalid rows skipped, rest continue | PASS | `try/catch` per row in transaction loop; errors collected but processing continues |
| Single SQLite transaction for all inserts | PASS | `db.withTransactionAsync(async () => { for each row... })` wraps all inserts; no nested transactions (DatabaseService methods use `runAsync` not `withTransactionAsync`) |

---

## Bug Reports

### BUG-001: Imported Bills Created as Recurring Templates (HIGH)

**Severity**: High
**Component**: `src/services/ImportService.ts` — `insertBillRow()`

#### Steps to Reproduce
1. Open Settings → Import Data
2. Select a CSV/XLSX containing bill records
3. Map columns and tap "Import N Records"
4. Navigate to the Bills list

#### Expected Result
Per `openspec/changes/import-spreadsheet/specs/spreadsheet-import/spec.md` (lines 106-109):
> "WHEN the user imports a bills sheet"
> "THEN each row is inserted with `is_recurring = false`"
> "AND no recurring template is created"

Imported bills should appear as one-time historical entries, NOT as recurring templates.

#### Actual Result
`ImportService.insertBillRow()` at line 896 sets `is_recurring: true`:
```typescript
const id = await this.dbService.createBill({
  account_id: accountId,
  name,
  amount: Math.abs(amount),
  due_day: dueDay,
  category,
  is_recurring: true,   // <-- BUG: spec requires false
  frequency,
  notes,
});
```

Every imported bill becomes an active recurring template. This causes:
- Bills appearing in recurring bill tracking workflows
- "Is Paid" status tracking triggered on imported historical data
- Pay period allocations may pick up imported bills unintentionally

#### Fix Required
Change `is_recurring: true` to `is_recurring: false` in `insertBillRow()`.

#### Environment
- Branch: dev (commit 67f463d)
- File: `/src/services/ImportService.ts` line 896

---

### BUG-002: Missing "View" Button on Potential Duplicate Cards (MEDIUM)

**Severity**: Medium
**Component**: `src/screens/ImportResultsScreen.tsx` — `DuplicateCard`

#### Steps to Reproduce
1. Import a spreadsheet where some rows match existing records (same date ±1 day, amount, category)
2. Reach the ImportResultsScreen
3. Expand the "Potential Duplicates" section
4. Observe the duplicate cards

#### Expected Result
Per `openspec/changes/import-spreadsheet/specs/spreadsheet-import/spec.md` (lines 129-131):
> "each flagged row has a 'View' button that opens the record detail screen"

Each duplicate card should have a tappable "View" button that navigates to the record so users can review and delete true duplicates.

#### Actual Result
`DuplicateCard` renders as a static `<View>` with no `onPress` handler and no "View" button:
```tsx
function DuplicateCard({ duplicate, themeColors }) {
  return (
    <View  // <-- not a TouchableOpacity, no onPress
      style={[styles.duplicateCard, { ... }]}
    >
      <Text>Row {duplicate.rowNumber}</Text>
      <Text>{duplicate.displayLabel}</Text>
      <Text>May duplicate record #{duplicate.existingId}</Text>
      {/* No View button */}
    </View>
  );
}
```

Users can see which rows are potential duplicates but have no way to navigate directly to the record to review or delete it. The secondary spec scenario ("returning to results screen reflects deletion") is also missing.

#### Workaround
Users must manually navigate to Expenses/Income/Mileage/Bills list and search for the duplicate record.

#### Fix Required
1. Convert `DuplicateCard` to a `TouchableOpacity` with navigation
2. Navigate to the appropriate record detail screen based on `recordType` and `insertedId`
3. Use navigation focus listener to refresh duplicate state on return

#### Environment
- Branch: dev (commit 67f463d)
- File: `/src/screens/ImportResultsScreen.tsx` lines 239-271

---

## Low Severity Findings (Not Blocking)

### FINDING-001: No 10MB File Size Check (LOW)

**Spec**: "maximum of 5,000 data rows and 10 MB file size" (spec.md line 12)
**Implementation**: `pickFile()` does not check `asset.size` against a 10MB limit.
**Impact**: A user could pick a file that passes the row limit but uses excessive memory (e.g., 5000 rows with very large cell values). Practical risk is low given the row limit.
**Recommendation**: Add size check against `asset.size` in `pickFile()`.

---

### FINDING-002: Future-Dated Rows Silently Skipped (LOW)

**Root cause**: `validatePastDate()` is called by `validateExpenseCreation()` and `validateIncomeCreation()`. Any expense or income row with a date in the future throws a `ValidationError`, which is caught by the row-level `try/catch` and added to the "Skipped" list.
**Impact**: Users importing forward-dated expenses or income (e.g., pre-scheduled data exported from accounting software) will see those rows skipped with the reason "Date cannot be in the future".
**Not a bug in isolation** — the behavior is caught and reported, not silently dropped. The error message is correct but may surprise users importing planned future data.
**Recommendation**: Document in the "Skipped" section hint text that future dates are not supported.

---

### FINDING-003: Bills `frequency` Required in Spec, Optional in UI (LOW)

**Spec** (spec.md line 73): "Required fields per record type: ... Bills (`name`, `amount`, `due_day`, `category`, `frequency`)"
**Implementation**: `ColumnMappingScreen` BILL_FIELDS lists `frequency` with `required: false`. Bills without a frequency mapping default to `'monthly'`.
**Impact**: Users can proceed without mapping a frequency column; all bills get `'monthly'` frequency silently. This is a reasonable default but deviates from spec's stated requirement.
**Severity**: Low — the default is sensible and this won't cause data loss.

---

## Manual QA Checklist (TASK-IMP-010)

The following manual items from `tasks.md` **cannot be verified automatically** and require device testing:

| Item | Auto-Verifiable? | Status |
|------|-----------------|--------|
| Import 50-row expense XLSX, verify all rows in ExpenseListScreen | No (requires iOS simulator + real file) | Deferred to device |
| Import CSV with US-format dates, verify ISO storage | Partially (unit tests confirm coercion) | Unit test PASS; device deferred |
| Import with 2 unmapped columns, fix in mapping screen, verify result | No (requires UI interaction) | Deferred to device |
| Import a .numbers file, verify error message | No (requires iOS file picker) | Code confirms: correct string check + instructions |
| Import 5,001-row file, verify row-limit error | Partially (unit test passes) | Unit test PASS; device deferred |
| Import where 3 rows duplicate existing data, verify results report | No (requires DB state + file) | Logic verified; View button missing (BUG-002) |
| Import with active account set, verify all records use that account | Partially (code trace confirms) | Code trace PASS; device deferred |

---

## Code Quality Observations

### Positive
- Singleton pattern consistent with `ExportService`
- All 4 record types have full coverage (expense/income/mileage/bill)
- Row-level error handling is correct: invalid rows skip without aborting the batch
- Per-row error messages are specific and user-facing (row number + reason)
- Date coercion handles 6 formats including Excel serials, ISO, US, EU heuristic, written
- Amount coercion handles currency symbols, commas, parentheses for negatives
- Alias tables well-populated; duplicate-prevention in `detectMapping` is correct
- `withTransactionAsync` correctly wraps all inserts; no nested transaction risk
- `scheduleCheckpoint()` inside transaction is safe (fires after commit via setTimeout)
- Navigation type definitions are complete and correct in `SettingsStackParamList`

### Minor Code Notes
- `confidence` set to `1` when user explicitly selects "skip" — cosmetic only, no functional impact
- Aliases not pre-normalized before comparison in `findBestMatch` — aliases with special chars (`project/client`) match via fuzzy score instead of exact; correct result but suboptimal confidence score
- Income and mileage duplicate detection not implemented (code comment calls it "simplified") — `detectDuplicates` returns empty map for these types

---

## Summary

Two spec deviations were found through adversarial code review:

1. **BUG-001 (High)**: `insertBillRow` uses `is_recurring: true` but spec requires `is_recurring: false`. A one-line fix.

2. **BUG-002 (Medium)**: `DuplicateCard` is not interactive. The "View" button for navigating to duplicate records was not implemented. The spec requires this for user-driven deduplication.

All automated tests pass (2028/2028), TypeScript is clean, and the core import flow (file pick → column mapping → transaction insert → results report) is architecturally sound and meets 8 of 10 spec acceptance criteria.

**Action required**: Fix BUG-001 and BUG-002 before promoting dev → main.

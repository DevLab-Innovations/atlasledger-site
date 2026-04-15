# Follow-Up Issues from QA Report (PRs #25 and #26)

Generated: 2026-01-18
QA Report: See `/Users/neptune/repos/agenticSdlc/docs/qa/QA_REPORT_PR25_PR26.md`

---

## Issue 1: Fix FinanceKitService Test Suite Failures

**Priority**: Medium
**Type**: Bug Fix
**Component**: Testing Infrastructure

### Description
6 tests in `src/__tests__/services/FinanceKitService.test.ts` are failing due to:
1. LogService mock not properly configured (2 tests)
2. Incorrect error message assertions (3 tests)
3. Platform availability check issue (1 test)

### Failing Tests
1. `Authorization › should request authorization successfully`
2. `Authorization › should handle authorization denial`
3. `Authorization › should throw error when requesting authorization on unavailable platform`
4. `Transaction Fetching › should fetch transactions successfully`
5. `Transaction Fetching › should throw error when FinanceKit unavailable`
6. (1 additional test - see full test output)

### Root Causes

#### Issue 1a: LogService.error undefined
```typescript
TypeError: Cannot read properties of undefined (reading 'error')
  at FinanceKitService.error (src/services/FinanceKitService.ts:176:18)
```

**Fix Required**:
- Update test setup to properly mock LogService
- Ensure LogService.error method is available in test environment

#### Issue 1b: Error Message Assertion Mismatch
```typescript
Expected substring: "not available"
Received message: "FinanceKit module is not installed. Install expo-finance-kit to enable transaction import."
```

**Fix Required**:
- Update test assertions to match actual error messages
- OR update error messages in code to match test expectations

### Steps to Reproduce
```bash
npm test -- src/__tests__/services/FinanceKitService.test.ts
```

### Expected Behavior
All FinanceKitService tests should pass with proper mocking and correct assertions.

### Suggested Fix
1. Review and update test file setup/teardown
2. Mock LogService properly in test environment
3. Update error message assertions to match implementation
4. Verify all 6 tests pass after fix

### Files Involved
- `src/__tests__/services/FinanceKitService.test.ts`
- `src/services/FinanceKitService.ts`
- Test setup/configuration files

---

## Issue 2: Fix ExpenseListScreen "Pull to Refresh" Test

**Priority**: Medium
**Type**: Bug Fix
**Component**: Testing Infrastructure

### Description
1 test in `src/__tests__/screens/ExpenseListScreen.test.tsx` is failing:
- Test: "Refresh › should pull to refresh"
- Error: `Unable to find an element with testID: expense-list`

### Failing Test Details
```typescript
Unable to find an element with testID: expense-list

  at Object.getByTestId (src/__tests__/screens/ExpenseListScreen.test.tsx:375:24)
```

### Root Cause
The test attempts to find an element with `testID="expense-list"` but the component is rendering with a different structure or the testID is not being applied correctly in the test environment.

### Steps to Reproduce
```bash
npm test -- src/__tests__/screens/ExpenseListScreen.test.tsx
```

### Expected Behavior
The test should successfully find the FlatList component and trigger the refresh event.

### Suggested Fix
1. Review ExpenseListScreen component rendering in test environment
2. Verify `testID="expense-list"` is correctly applied to the FlatList component
3. Check if component conditional rendering is affecting test
4. Update test to match actual component structure
5. Verify test passes after fix

### Files Involved
- `src/__tests__/screens/ExpenseListScreen.test.tsx`
- `src/screens/ExpenseListScreen.tsx`

---

## Issue 3: Investigate Test Count Discrepancy

**Priority**: Low
**Type**: Investigation

### Description
Test count comparison between main and dev branches shows:
- main: 544 tests passed, 32 skipped
- dev: 549 tests passed, 4 skipped

This is a +5 test increase (expected from PR #25) but the skipped count changed from 32 to 4.

### Questions to Answer
1. Why did 28 previously skipped tests start running?
2. Are these tests now passing or were they removed?
3. Is this intentional or a side effect of test configuration changes?

### Investigation Steps
1. Compare test configuration between main and dev
2. Identify which tests changed from skipped to running
3. Verify this is expected behavior
4. Document findings

### Impact
Low - Tests are passing, but understanding the change is important for test suite maintenance.

---

## Summary

| Issue | Priority | Type | Failing Tests | Estimated Effort |
|-------|----------|------|---------------|------------------|
| #1 - FinanceKitService | Medium | Bug Fix | 6 tests | 2-3 hours |
| #2 - ExpenseListScreen | Medium | Bug Fix | 1 test | 1-2 hours |
| #3 - Test Count Change | Low | Investigation | N/A | 1 hour |

**Total Estimated Effort**: 4-6 hours

---

## Recommendations

1. Create GitHub issues for each of the above problems
2. Assign to appropriate developer(s)
3. Target for next sprint or maintenance window
4. These issues do NOT block the promotion of PRs #25 and #26 to main

---

**Document Prepared By**: qa-tester agent
**Date**: 2026-01-18

# Manual Test Checklist: PR #42 - Premium Feature Gating

**Date**: 2026-01-21
**Tester**: _____________
**Device**: iOS Simulator (iPhone 16 Pro)
**Build**: dev branch (commit 57e6108)

---

## Instructions

1. Launch the app on iOS Simulator
2. Complete each test scenario in order
3. Mark PASS/FAIL for each scenario
4. Document any issues found
5. Take screenshots of failures

---

## Test Scenarios

### 1. Premium Badge Display
**Objective**: Verify premium badges appear on locked features

- [ ] Launch app as free user
- [ ] Navigate to Money tab
- [ ] Observe Expenses card: NO premium badge
- [ ] Observe Income card: PREMIUM badge visible in top-right
- [ ] Observe Bills card: PREMIUM badge visible in top-right
- [ ] Verify badges are properly aligned with chevron icons

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 2. Income Tab Feature Gate
**Objective**: Free user tapping Income tab shows upgrade flow

- [ ] As free user, tap Income card in Money Hub
- [ ] Verify navigation to Settings > Paywall screen
- [ ] Verify Premium tier is highlighted
- [ ] Verify pricing information is displayed
- [ ] Tap back button to return to Money Hub

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 3. Bills Tab Feature Gate
**Objective**: Free user tapping Bills tab shows upgrade flow

- [ ] As free user, tap Bills card in Money Hub
- [ ] Verify navigation to Settings > Paywall screen
- [ ] Verify Premium tier is highlighted
- [ ] Tap back button to return to Money Hub

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 4. FAB Menu Lock Icons
**Objective**: Verify locked features show lock icons in FAB menu

- [ ] On Money Hub screen, tap FAB (bottom-right corner)
- [ ] Observe "Add Expense" action: cash-minus icon (NO lock)
- [ ] Observe "Add Income" action: LOCK icon
- [ ] Observe "Add Bill" action: LOCK icon
- [ ] Tap "Add Income" (locked)
- [ ] Verify navigation to Paywall screen
- [ ] Return to Money Hub
- [ ] Tap FAB again
- [ ] Tap "Add Bill" (locked)
- [ ] Verify navigation to Paywall screen

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 5. Account Creation - Free User (1st Account)
**Objective**: Free user can create first account

- [ ] Navigate to Settings > Accounts
- [ ] Tap "Create Account" button
- [ ] Enter name: "Personal Testing"
- [ ] Select type: Personal
- [ ] Choose color: Blue
- [ ] Tap Save
- [ ] Verify account is created successfully
- [ ] Verify no error message shown
- [ ] Verify account appears in list

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 6. Account Creation - Free User (2nd Account - Blocked)
**Objective**: Free user is blocked from creating second account

- [ ] On Settings > Accounts screen (with 1 existing account)
- [ ] Tap "Create Account" button
- [ ] Enter name: "Business Testing"
- [ ] Select type: Business
- [ ] Choose color: Green
- [ ] Tap Save
- [ ] Verify error alert is displayed
- [ ] Verify error message mentions account limit (1/1)
- [ ] Verify error message includes "Upgrade to Premium"
- [ ] Tap OK on alert
- [ ] Verify account is NOT created
- [ ] Return to Accounts list
- [ ] Verify only 1 account exists

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 7. Account Creation - Premium User (2nd Account)
**Objective**: Premium user can create second account

**Setup**: Simulate premium subscription (may require dev mode toggle)

- [ ] Upgrade to Premium tier
- [ ] Navigate to Settings > Accounts
- [ ] Verify 1 existing account
- [ ] Tap "Create Account" button
- [ ] Enter name: "Premium Account"
- [ ] Select type: Business
- [ ] Choose color: Orange
- [ ] Tap Save
- [ ] Verify account is created successfully
- [ ] Verify no error message shown
- [ ] Verify 2 accounts now exist in list

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 8. Regression - Expenses Tab
**Objective**: Expenses functionality unchanged

- [ ] On Money Hub screen, tap Expenses card
- [ ] Verify Expenses list screen loads
- [ ] Tap FAB to add new expense
- [ ] Create test expense: "Test Expense", $25.00
- [ ] Save expense
- [ ] Verify expense appears in list
- [ ] Tap expense to view details
- [ ] Edit expense amount to $30.00
- [ ] Save changes
- [ ] Return to Expenses list
- [ ] Swipe to delete expense
- [ ] Confirm deletion
- [ ] Verify expense is removed

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 9. Regression - Dashboard Display
**Objective**: Dashboard displays correctly with locked features

- [ ] Navigate to Money tab
- [ ] Verify all three cards are visible
- [ ] Verify Expenses card shows actual data
- [ ] Verify Income card shows $0.00 (or actual data if exists)
- [ ] Verify Bills card shows status and amounts
- [ ] Verify layout is clean and consistent
- [ ] Verify no visual glitches or overlaps

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

### 10. Accessibility - VoiceOver
**Objective**: Verify accessibility labels for locked features

**Setup**: Enable VoiceOver on simulator (Cmd+F5)

- [ ] Navigate to Money Hub
- [ ] Swipe to Income card
- [ ] Listen to VoiceOver label
- [ ] Verify label includes "Premium feature"
- [ ] Swipe to Bills card
- [ ] Listen to VoiceOver label
- [ ] Verify label includes "Premium feature"
- [ ] Open FAB menu
- [ ] Swipe to "Add Income" action
- [ ] Verify label indicates "Premium feature"
- [ ] Swipe to "Add Bill" action
- [ ] Verify label indicates "Premium feature"

**Result**: [ ] PASS [ ] FAIL
**Notes**: _______________________________________________

---

## Summary

**Total Scenarios**: 10
**Passed**: _____
**Failed**: _____
**Pass Rate**: _____%

**Critical Issues Found**: _____
**High Issues Found**: _____
**Medium Issues Found**: _____
**Low Issues Found**: _____

**Overall Assessment**: [ ] PASS [ ] FAIL [ ] NEEDS REVIEW

**Recommendation**:
[ ] Ready for production release
[ ] Requires minor fixes
[ ] Requires major fixes
[ ] Block release

---

## Issues Log

### Issue #1
**Title**: _______________________________________________
**Severity**: [ ] Critical [ ] High [ ] Medium [ ] Low
**Description**: _______________________________________________
**Steps to Reproduce**: _______________________________________________
**Expected**: _______________________________________________
**Actual**: _______________________________________________
**Screenshot**: _______________________________________________

### Issue #2
**Title**: _______________________________________________
**Severity**: [ ] Critical [ ] High [ ] Medium [ ] Low
**Description**: _______________________________________________
**Steps to Reproduce**: _______________________________________________
**Expected**: _______________________________________________
**Actual**: _______________________________________________
**Screenshot**: _______________________________________________

---

**Tester Signature**: _______________ **Date**: _______________

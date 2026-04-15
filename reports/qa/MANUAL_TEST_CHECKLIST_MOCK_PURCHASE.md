# Manual Test Checklist: Mock Card Purchase Flow

**Date**: 2026-01-19
**Build**: TestFlight mode (TESTFLIGHT_MODE = true)
**Tester**: _______________

---

## Pre-Test Setup ✅

- [ ] iOS Simulator is running
- [ ] Expo app loaded (should auto-reload after SettingsScreen change)
- [ ] Navigate to Settings tab in the app
- [ ] Verify "Subscription" section is visible (above "Organization")

---

## Test Execution Checklist

### ✅ Test 1: Navigate to PaywallScreen
- [ ] Tap "Settings" tab at bottom
- [ ] Scroll to "Subscription" section
- [ ] Tap "Upgrade to Premium"
- [ ] **VERIFY**: PaywallScreen opens as modal (no header)
- [ ] **VERIFY**: "Choose Your Plan" title visible
- [ ] **VERIFY**: Three tier cards: Premium, Pro, Team
- [ ] **VERIFY**: Monthly/Annual billing toggle at top

---

### ✅ Test 2: Valid Card Purchase - Premium Tier
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] **VERIFY**: Mock card dialog appears
- [ ] **VERIFY**: Title says "TestFlight Purchase"
- [ ] **VERIFY**: Description mentions "simulated purchase"
- [ ] **VERIFY**: Tip shows "Use 4242 4242 4242 4242"
- [ ] **VERIFY**: "TESTFLIGHT MODE" badge visible
- [ ] Enter: `4242 4242 4242 4242`
- [ ] **VERIFY**: Card number auto-formats with spaces
- [ ] Tap "Confirm Purchase"
- [ ] **VERIFY**: Success alert: "Welcome to Atlas Premium!"
- [ ] **VERIFY**: Navigates back after dismissing alert

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 3: Invalid Card - Too Short
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Enter: `4242 4242 42` (only 10 digits)
- [ ] Tap "Confirm Purchase"
- [ ] **VERIFY**: Alert appears: "Invalid Card"
- [ ] **VERIFY**: Alert message mentions "valid 16-digit card number"
- [ ] **VERIFY**: Tip in alert: "Use 4242 4242 4242 4242"
- [ ] **VERIFY**: Dialog stays open (can try again)
- [ ] Tap "OK" to dismiss alert
- [ ] Tap "Cancel" to close dialog

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 4: Invalid Card - Non-Numeric
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Try to type letters: `abcd efgh`
- [ ] **VERIFY**: Only numbers appear in input field
- [ ] **VERIFY**: Letters are automatically filtered out
- [ ] Tap "Cancel"

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 5: Card Number Formatting
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Type slowly without spaces: `4242424242424242`
- [ ] **VERIFY**: As you type, spaces auto-insert every 4 digits
- [ ] **VERIFY**: Final display: `4242 4242 4242 4242`
- [ ] **VERIFY**: Cannot type more than 16 digits
- [ ] Tap "Cancel"

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 6: Cancel Button
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Enter partial card number: `4242 42`
- [ ] Tap "Cancel"
- [ ] **VERIFY**: Dialog closes immediately
- [ ] **VERIFY**: No alert appears
- [ ] **VERIFY**: Back on PaywallScreen

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 7: Dismiss by Tapping Outside
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Tap on the dark semi-transparent area OUTSIDE the white dialog box
- [ ] **VERIFY**: Dialog closes
- [ ] **VERIFY**: Back on PaywallScreen

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 8: Pro Tier Purchase
- [ ] From PaywallScreen, tap "Get Pro"
- [ ] **VERIFY**: Same mock dialog appears
- [ ] Enter: `4242 4242 4242 4242`
- [ ] Tap "Confirm Purchase"
- [ ] **VERIFY**: Success alert: "Welcome to Atlas Pro!"
- [ ] **VERIFY**: Navigates back

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 9: Team Tier Purchase
- [ ] From PaywallScreen, tap "Get Team"
- [ ] **VERIFY**: Same mock dialog appears
- [ ] Enter: `4242 4242 4242 4242`
- [ ] Tap "Confirm Purchase"
- [ ] **VERIFY**: Success alert: "Welcome to Atlas Team!"
- [ ] **VERIFY**: Navigates back

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 10: Billing Period Toggle
- [ ] From PaywallScreen, observe "Monthly" is selected
- [ ] Tap "Annual - Save 17%"
- [ ] **VERIFY**: All tier prices change to annual pricing
- [ ] **VERIFY**: "Save $XX" text appears under prices
- [ ] Tap "Monthly"
- [ ] **VERIFY**: Prices revert to monthly
- [ ] **VERIFY**: Savings text disappears

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 11: TESTFLIGHT MODE Badge
- [ ] From PaywallScreen, tap any "Get [Tier]" button
- [ ] **VERIFY**: Mock dialog appears
- [ ] Scroll to bottom of dialog
- [ ] **VERIFY**: "TESTFLIGHT MODE" badge is visible
- [ ] **VERIFY**: Badge text is yellow/warning color
- [ ] **VERIFY**: Badge is centered

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### ✅ Test 12: Keyboard and Auto-Focus
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] **VERIFY**: Dialog opens
- [ ] **VERIFY**: Card input field is auto-focused (keyboard appears)
- [ ] **VERIFY**: Numeric keyboard appears (number pad, not QWERTY)
- [ ] Tap "Cancel"

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

## Edge Cases

### Edge Case 1: Rapid Tapping
- [ ] From PaywallScreen, rapidly tap "Get Premium" 5+ times
- [ ] **VERIFY**: Only ONE dialog opens
- [ ] **VERIFY**: No duplicate dialogs or errors

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### Edge Case 2: Alternative Valid Cards
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Enter: `1111 1111 1111 1111`
- [ ] Tap "Confirm Purchase"
- [ ] **VERIFY**: Accepted (mock accepts any 16-digit number)
- [ ] **VERIFY**: Success alert appears

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

### Edge Case 3: Loading State
- [ ] From PaywallScreen, tap "Get Premium"
- [ ] Enter: `4242 4242 4242 4242`
- [ ] Tap "Confirm Purchase"
- [ ] **VERIFY**: Button shows loading spinner briefly
- [ ] **VERIFY**: Button is disabled during loading
- [ ] **VERIFY**: After loading, success alert appears

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

## Visual Verification Checklist

### Mock Dialog Appearance
- [ ] Title: "TestFlight Purchase" (large, bold, centered)
- [ ] Description text (gray, centered)
- [ ] Tip text in blue/primary color
- [ ] Card input label: "Card Number"
- [ ] Card input has border and background
- [ ] Cancel button (outlined)
- [ ] Confirm Purchase button (filled/contained)
- [ ] TESTFLIGHT MODE badge (yellow, centered, bottom)

### PaywallScreen Appearance
- [ ] Clean modal presentation (no header bar)
- [ ] Close button (X) in top-right corner
- [ ] Three tier cards with clear visual separation
- [ ] Each tier has pricing, feature list, and CTA button
- [ ] Billing toggle is clearly labeled
- [ ] All text is readable and properly sized

---

## Accessibility Testing (Optional)

### With VoiceOver Enabled
- [ ] Enable VoiceOver: Settings → Accessibility → VoiceOver
- [ ] Navigate to mock card dialog
- [ ] **VERIFY**: All buttons are announced
- [ ] **VERIFY**: Card input has label "Card number input"
- [ ] **VERIFY**: Card input has hint "Enter a 16-digit test card number"
- [ ] **VERIFY**: All elements are focusable

**Result**: ⬜ PASS  ⬜ FAIL
**Notes**: _______________________________________

---

## Bug Reports

### Bugs Found During Testing

**Bug #1**: _______________________________________________
- **Severity**: ⬜ Critical  ⬜ High  ⬜ Medium  ⬜ Low
- **Description**: ________________________________________
- **Steps to Reproduce**: __________________________________
- **Expected**: ___________________________________________
- **Actual**: _____________________________________________

**Bug #2**: _______________________________________________
- **Severity**: ⬜ Critical  ⬜ High  ⬜ Medium  ⬜ Low
- **Description**: ________________________________________
- **Steps to Reproduce**: __________________________________
- **Expected**: ___________________________________________
- **Actual**: _____________________________________________

---

## Summary

**Total Tests Executed**: ___ / 15
**Tests Passed**: ___
**Tests Failed**: ___
**Bugs Found**: ___

**Overall Assessment**:
⬜ PASS - Ready for TestFlight release
⬜ PASS WITH MINOR ISSUES - Can proceed with notes
⬜ FAIL - Blocking issues found, needs fixes

**Additional Notes**:
_______________________________________________________
_______________________________________________________
_______________________________________________________

**Tester Signature**: _______________  **Date**: __________

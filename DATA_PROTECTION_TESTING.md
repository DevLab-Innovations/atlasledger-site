# Data Protection Testing Guide

## Overview

This document outlines the testing procedures for iOS Data Protection capability enabled in Wave 0.7A. The app uses "Protected Until First User Authentication" level, which encrypts all app data at rest while allowing background operations after first device unlock.

## Protection Level Configuration

**Level:** NSFileProtectionCompleteUntilFirstUserAuthentication

**Location:** `/ios/ExpenseTracker/ExpenseTracker.entitlements`

**Entitlement Key:** `com.apple.developer.default-data-protection`

## What Gets Protected

All app data is encrypted at the file system level:
- SQLite database (expenses, amounts, merchants, categories)
- Receipt images stored locally
- Mileage location history
- Sync queue with pending changes
- FinanceKit imported transactions (when implemented)
- User preferences and settings

## Test Plan

### Task 0.7A.4: Data Inaccessible When Device Locked (Before First Unlock)

**Objective:** Verify that app data is encrypted and inaccessible when device is powered off and hasn't been unlocked yet.

**Prerequisites:**
- Install app on physical iOS device (cannot test on simulator)
- Add test expense data including receipts
- Ensure device has a passcode set

**Test Steps:**
1. Power off the device completely
2. Power on the device (DO NOT unlock with passcode)
3. Wait for device to boot to lock screen
4. Attempt to access app data through:
   - iTunes backup extraction (data should be encrypted)
   - File system access via tools (should fail)
5. Unlock device with passcode
6. Launch app
7. Verify all data is accessible and intact

**Expected Results:**
- ✅ Data cannot be accessed before first unlock
- ✅ After unlock, app launches normally
- ✅ All expenses, receipts, and data are intact

**Notes:**
- This protection prevents data theft if device is stolen while powered off
- Data remains encrypted until user authenticates after boot

---

### Task 0.7A.5: Background Sync Works After First Unlock

**Objective:** Verify background sync operations can access data after first device unlock.

**Prerequisites:**
- App installed with sync configured (when backend is implemented)
- Device locked but has been unlocked at least once since boot

**Test Steps:**
1. Ensure device is unlocked and app is running
2. Create/modify test expenses
3. Force app to background (swipe up)
4. Lock device (press power button)
5. Wait 1-2 minutes for background sync to trigger
6. Unlock device
7. Check sync logs/status

**Expected Results:**
- ✅ Background sync completes successfully while device is locked
- ✅ Changes are uploaded to sync service
- ✅ No sync failures due to data protection errors

**Alternate Test (Manual Verification):**
Since backend sync isn't implemented yet:
1. Verify app can be backgrounded and resumed without data loss
2. Test background task registration (if implemented)
3. Verify app state restoration after device lock/unlock

**Notes:**
- "Complete Until First User Authentication" allows background operations after initial unlock
- This is why we chose this level over "Complete" protection

---

### Task 0.7A.6: Push Notifications Work Correctly

**Objective:** Verify push notifications can be delivered and processed when device is locked (after first unlock).

**Prerequisites:**
- Push notification capability enabled
- Device locked but unlocked at least once since boot
- Test notification sending capability

**Test Steps:**
1. Ensure device is unlocked and app has notification permission
2. Lock device (press power button)
3. Send test push notification
4. Verify notification appears on lock screen
5. Tap notification
6. Verify app opens to correct screen with data

**Expected Results:**
- ✅ Notification delivered successfully to locked device
- ✅ Notification displays correct content
- ✅ Tapping notification opens app correctly
- ✅ App can access data when opened from notification

**Current Status:**
- Push notifications not yet implemented
- Defer this test until notification system is built

---

### Task 0.7A.7: Document Data Protection in Privacy Policy

**Objective:** Update app documentation to reflect data protection implementation.

**Required Documentation:**
1. Update Privacy Policy to include:
   - "All financial data is encrypted at rest using iOS Data Protection"
   - "Data is protected when device is locked or powered off"
   - "Encryption keys are tied to device passcode"

2. Update App Store description to include:
   - Security features bullet point
   - "Bank-level encryption for your expense data"

3. Update README.md with security section:
   - Data protection level and rationale
   - What data is protected
   - User requirements (device passcode required)

**Deliverables:**
- [x] Privacy policy created (`/docs/PRIVACY_POLICY.md`)
- [x] App Store description draft (`/docs/appstore/SECURITY_DESCRIPTION.md`)
- [x] README.md security section
- [x] User-facing help/FAQ about security (`/docs/SECURITY_FAQ.md`)

---

### Task 0.7A.8: Verify Capability Shows in Apple Developer Portal

**Objective:** Confirm Data Protection capability is properly registered with Apple.

**Test Steps:**
1. Log into Apple Developer portal
2. Navigate to App ID (com.devlabinnovations.atlasledger)
3. Check Capabilities section
4. Verify "Data Protection" is enabled

**Expected Results:**
- ✅ Data Protection capability listed and enabled
- ✅ Entitlement matches app configuration

**Notes:**
- This is auto-synced when app is built with Xcode
- Required for App Store submission
- If missing, re-sync capabilities in Xcode (Signing & Capabilities tab)

---

### Task 0.7A.9: Verify App Passes App Store Security Review

**Objective:** Ensure data protection implementation meets App Store requirements.

**Pre-Submission Checklist:**
- [ ] Data Protection entitlement present in build
- [ ] No conflicting entitlements (e.g., excessive permissions)
- [ ] Privacy policy mentions data protection
- [ ] App Info.plist has required usage descriptions
- [ ] No hardcoded credentials or secrets in code

**Test Steps:**
1. Create TestFlight build with production entitlements
2. Submit for internal testing
3. Verify no capability errors in App Store Connect
4. Test on physical device via TestFlight
5. Review any Apple feedback/rejections

**Expected Results:**
- ✅ Build uploads successfully to App Store Connect
- ✅ No entitlement errors during processing
- ✅ TestFlight distribution works
- ✅ App Store review accepts security implementation

**Notes:**
- First submission may take 24-48 hours for review
- Data Protection is a standard capability, low risk of rejection
- Ensure privacy policy URL is accessible

---

### Task 0.7A.10: Verify Background Operations Work Correctly

**Objective:** Comprehensive test that all background features function with data protection enabled.

**Test Matrix:**

| Feature | Device State | Expected Result |
|---------|--------------|-----------------|
| App Background Refresh | Locked (after unlock) | ✅ Works |
| Receipt Upload | Locked (after unlock) | ✅ Works |
| FinanceKit Import | Locked (after unlock) | ✅ Works |
| Database Access | Locked (before unlock) | ❌ Fails gracefully |
| Database Access | Unlocked | ✅ Works |
| App Launch | Before first unlock | ❌ Shows error/placeholder |
| App Launch | After unlock | ✅ Works |

**Test Procedure:**
1. **Background Database Access:**
   - Trigger background task while locked (after first unlock)
   - Verify database queries succeed
   - Check no file protection errors in logs

2. **First Boot Scenario:**
   - Power off device
   - Power on (don't unlock)
   - Try to launch app from notification
   - Verify graceful error handling or wait for unlock

3. **Normal Operation:**
   - Lock/unlock device multiple times
   - Verify seamless app operation
   - Check no performance degradation

**Expected Results:**
- ✅ All background operations work after first unlock
- ✅ App handles "before first unlock" gracefully
- ✅ No crashes or data corruption
- ✅ Performance is unaffected

**Failure Scenarios to Test:**
- Device reboot without unlock
- Extended locked period (hours)
- Low battery/power mode

---

## Testing Tools

### Xcode Instruments
Use Instruments to monitor file system access:
```bash
# Profile app with System Trace
# Watch for file protection errors
```

### Console.app Filtering
Monitor device logs for protection errors:
```
# Filter by process: ExpenseLedger
# Search for: "NSFileProtection", "data protection", "unlock"
```

### Manual Testing Device Requirements
- Physical iPhone (iOS 13+)
- Device passcode enabled (required for data protection)
- Development build with entitlements
- Test data: expenses, receipts, mileage entries

---

## Rollback Procedure

If data protection causes issues:

1. **Disable in Entitlements:**
   ```xml
   <!-- Remove or comment out -->
   <key>com.apple.developer.default-data-protection</key>
   <string>NSFileProtectionCompleteUntilFirstUserAuthentication</string>
   ```

2. **Clean Build:**
   ```bash
   cd ios
   rm -rf build
   pod cache clean --all
   pod install
   xcodebuild clean
   ```

3. **Rebuild and Test:**
   - Verify app works without protection
   - Document any issues caused by protection

**Note:** Data protection should NOT cause issues at this level. If problems occur, likely root cause is:
- Background task configuration error
- File access during "before first unlock" state
- Third-party library incompatibility

---

## Verification Checklist

After completing all tests:

- [ ] Data encrypted when device powered off (0.7A.4)
- [ ] Background sync works (0.7A.5)
- [ ] Push notifications work (0.7A.6) - Deferred until push notifications implemented
- [x] Privacy policy updated (0.7A.7) - COMPLETE
- [ ] Developer portal shows capability (0.7A.8)
- [ ] TestFlight build submitted (0.7A.9)
- [ ] Background operations tested (0.7A.10)
- [ ] No crashes or data loss observed
- [ ] Performance impact measured (should be negligible)

---

## Implementation Status

**Configuration Status:** ✅ **COMPLETE**
- Entitlement file includes data protection
- Protection level: NSFileProtectionCompleteUntilFirstUserAuthentication
- Xcode project links entitlements correctly
- No code changes required (OS-level encryption)

**Next Steps:**
1. Execute test plan on physical device
2. Document test results
3. Update privacy policy
4. Submit TestFlight build for validation
5. Hand off to QA for comprehensive testing

---

## References

- [Apple Data Protection Documentation](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/encrypting_your_app_s_files)
- [iOS Security Guide](https://support.apple.com/guide/security/welcome/web)
- Project Entitlements: `/ios/ExpenseTracker/ExpenseTracker.entitlements`
- Xcode Project: `/ios/ExpenseTracker.xcodeproj`

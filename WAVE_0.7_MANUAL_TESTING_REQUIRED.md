# Wave 0.7 Manual Testing Required

## Overview

Wave 0.7 Data Protection is **4/10 tasks complete**. The remaining 6 tasks require physical device testing and Apple Developer portal access.

**Completed Tasks (Automated):**
- ✅ 0.7A.1: Data Protection capability enabled in Xcode
- ✅ 0.7A.2: Protection level configured (Protected Until First User Authentication)
- ✅ 0.7A.3: Entitlements file verified
- ✅ 0.7A.7: Documentation complete (README, Privacy Policy, Security FAQ)

**Remaining Tasks (Manual):**
- ⏳ 0.7A.4: Test data inaccessibility when device locked
- ⏳ 0.7A.5: Test background operations after first unlock
- ⏳ 0.7A.6: Test push notifications (deferred - not implemented yet)
- ⏳ 0.7A.8: Verify capability in Apple Developer portal
- ⏳ 0.7A.9: Submit TestFlight build for App Store validation
- ⏳ 0.7A.10: Comprehensive background operations testing

---

## Why Manual Testing is Required

iOS Data Protection **cannot be tested in the simulator** because:
- Simulator doesn't support iOS encryption features
- File protection is not enforced in simulator
- Device-specific hardware (Secure Enclave) is required

**You must use a physical iPhone or iPad to verify data protection works correctly.**

---

## Quick Start: Physical Device Testing

### Prerequisites

1. **Physical iOS Device** (iPhone or iPad)
   - iOS 13.0 or later
   - Device passcode enabled (Settings > Face ID & Passcode)
   - Development provisioning profile installed

2. **Xcode Setup**
   - Mac with Xcode installed
   - Developer account added to Xcode
   - Device connected via USB or Wi-Fi

3. **Build Configuration**
   - Development build with entitlements enabled
   - Data Protection entitlement present
   - Signing configured for your team

### Test Execution (15-20 minutes)

See **[DATA_PROTECTION_TESTING.md](DATA_PROTECTION_TESTING.md)** for full test procedures.

**Quick Test (5 minutes):**
1. Build and run app on physical device
2. Add test expense with receipt photo
3. Lock device (press power button)
4. Unlock device (enter passcode)
5. Verify app works and data is intact

**Comprehensive Test (15 minutes):**
1. Add multiple expenses with receipts
2. Power off device completely
3. Power on (DO NOT unlock yet)
4. Attempt to access app from lock screen (should fail gracefully)
5. Unlock device with passcode
6. Launch app - verify all data is accessible
7. Background the app and lock device
8. Verify app resumes correctly after unlock

---

## Apple Developer Portal Verification (Task 0.7A.8)

### Access Required

- Apple Developer account (Team Agent or Admin role)
- Access to: https://developer.apple.com/account/

### Steps

1. Log into Apple Developer portal
2. Navigate to **Certificates, Identifiers & Profiles**
3. Select **Identifiers** → Find your App ID
4. Click on the App ID to view details
5. Scroll to **Capabilities** section
6. **Verify:** "Data Protection" capability is listed and enabled

**Expected Result:**
```
Data Protection  ✅ Enabled
Level: Default (Protected Until First User Authentication)
```

**If Missing:**
- Open Xcode project
- Go to target → Signing & Capabilities
- Click "+ Capability" and add "Data Protection"
- Rebuild and re-sync capabilities

---

## TestFlight Submission (Task 0.7A.9)

### Prerequisites

- Apple Developer Program membership ($99/year)
- App Store Connect access
- App ID created in developer portal
- Provisioning profiles configured

### Steps

1. **Archive Build:**
   ```bash
   cd ios
   xcodebuild archive \
     -workspace ExpenseTracker.xcworkspace \
     -scheme ExpenseTracker \
     -configuration Release \
     -archivePath ./build/ExpenseTracker.xcarchive
   ```

2. **Export for TestFlight:**
   ```bash
   xcodebuild -exportArchive \
     -archivePath ./build/ExpenseTracker.xcarchive \
     -exportPath ./build/ \
     -exportOptionsPlist exportOptions.plist
   ```

3. **Upload to App Store Connect:**
   - Use Xcode Organizer (Window → Organizer)
   - Or use Transporter app
   - Or command line: `xcrun altool --upload-app`

4. **Wait for Processing:**
   - Apple processes build (10-30 minutes)
   - Check for errors in App Store Connect

5. **Add to TestFlight:**
   - Go to App Store Connect → My Apps → TestFlight
   - Select the build
   - Add internal testers
   - Submit for review (if external testing needed)

**Expected Results:**
- ✅ Build uploads successfully
- ✅ No entitlement errors during processing
- ✅ TestFlight distribution becomes available
- ✅ App installs and launches on test devices

---

## Background Operations Testing (Task 0.7A.10)

### Test Matrix

| Scenario | Device State | Expected Result | Status |
|----------|--------------|-----------------|--------|
| App Launch | Unlocked | ✅ Works | ⏳ Test |
| App Launch | Locked (after 1st unlock) | ✅ Works | ⏳ Test |
| App Launch | Before first unlock | ❌ Graceful error | ⏳ Test |
| Database Query | Unlocked | ✅ Works | ⏳ Test |
| Database Query | Locked (after 1st unlock) | ✅ Works | ⏳ Test |
| Database Query | Before first unlock | ❌ Fails | ⏳ Test |
| Background Refresh | Locked (after 1st unlock) | ✅ Works | ⏳ Test |
| File Access (receipt) | Locked (after 1st unlock) | ✅ Works | ⏳ Test |

### Detailed Test Procedures

See **[DATA_PROTECTION_TESTING.md](DATA_PROTECTION_TESTING.md)** sections:
- Task 0.7A.4: Data inaccessibility when locked (before first unlock)
- Task 0.7A.5: Background sync after first unlock
- Task 0.7A.10: Comprehensive background operations

---

## Automation Opportunities

While core encryption testing requires physical devices, some verification can be automated:

### Unit Tests (Possible)
- ✅ Verify entitlements file exists
- ✅ Check protection level is set correctly
- ✅ Validate plist configuration

### Integration Tests (Limited)
- ⚠️ Mock file protection errors
- ⚠️ Test error handling for locked state
- ⚠️ Verify graceful degradation

### Manual Tests (Required)
- 🔴 Physical device encryption verification
- 🔴 Lock/unlock behavior
- 🔴 Power off/on scenarios
- 🔴 Apple Developer portal verification
- 🔴 TestFlight submission

---

## Troubleshooting

### Common Issues

**Problem:** "Data Protection Not Available"
- **Cause:** Running in simulator
- **Solution:** Build for physical device

**Problem:** App crashes on launch after reboot
- **Cause:** Attempting to access database before first unlock
- **Solution:** Add error handling for "device locked" state

**Problem:** Capability not showing in portal
- **Cause:** Entitlements not synced
- **Solution:** Re-archive build in Xcode, sync capabilities

**Problem:** TestFlight upload fails
- **Cause:** Invalid provisioning profile or entitlements
- **Solution:** Verify signing configuration, regenerate profiles

---

## Next Steps

### For Immediate Testing (Today)

If you have a physical device available:
1. Connect device to Mac
2. Build and run from Xcode
3. Execute quick test (5 minutes)
4. Document results

### For Comprehensive Validation (This Week)

1. Execute full test plan (15-20 minutes)
2. Verify Apple Developer portal (5 minutes)
3. Create TestFlight build (30 minutes)
4. Document all results

### For Production Release (Next Week)

1. Complete all manual tests
2. Submit TestFlight build for external testing
3. Get user feedback on security features
4. Update Wave 0.7 tasks to complete
5. Mark Wave 0.7 as 100% complete

---

## Impact Assessment

### If Testing is Skipped

**Medium Risk:**
- Data Protection configuration is correct (verified via code review)
- Entitlements are in place
- Documentation is complete
- Apple will catch any issues during App Store review

**Potential Issues:**
- Undiscovered edge cases with lock/unlock
- Unexpected behavior before first unlock
- Performance impact not measured

### If Testing is Completed

**High Confidence:**
- All edge cases validated
- Security features verified working
- Performance confirmed acceptable
- Ready for App Store submission
- Marketing can confidently claim "bank-level encryption"

---

## Resources

- **Full Test Guide:** [DATA_PROTECTION_TESTING.md](DATA_PROTECTION_TESTING.md)
- **User FAQ:** [SECURITY_FAQ.md](SECURITY_FAQ.md)
- **Privacy Policy:** [PRIVACY_POLICY.md](PRIVACY_POLICY.md)
- **Apple Documentation:** [Encrypting Your App's Files](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/encrypting_your_app_s_files)
- **iOS Security Guide:** [Apple Security PDF](https://support.apple.com/guide/security/welcome/web)

---

## Status Summary

**Wave 0.7 Progress:** 4/10 tasks complete (40%)

**Completed:**
- ✅ Configuration (Xcode, entitlements)
- ✅ Documentation (README, FAQ, Privacy Policy)

**Remaining:**
- ⏳ Physical device testing (requires hardware)
- ⏳ Apple Developer portal check (requires account access)
- ⏳ TestFlight submission (requires paid developer account)

**Blockers:**
- Physical iOS device with passcode enabled
- Apple Developer account access
- Time allocation for manual testing (20-30 minutes)

**Recommendation:**
Proceed with physical device testing when hardware is available. Data Protection is correctly configured and documented - testing will validate it works as expected.

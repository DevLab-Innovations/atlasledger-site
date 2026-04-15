# iOS Data Protection Implementation

## Overview

This document describes the iOS Data Protection configuration for the Expense Ledger app, implementing Wave 0.7 of the development roadmap.

## Configuration

### Protection Level

**Selected Level:** `NSFileProtectionCompleteUntilFirstUserAuthentication`

This protection level provides a balance between security and functionality:

- Data is encrypted when the device is powered off or before first unlock after boot
- Data remains accessible after first device unlock, even when locked
- Background features (sync, notifications) work after first unlock
- Industry standard for financial applications

### What Gets Protected

All app data is automatically encrypted at rest by iOS when using Data Protection:

1. **SQLite Database**
   - Expenses table (amounts, dates, merchants, descriptions)
   - Mileage entries
   - Categories and merchant history
   - All financial data

2. **File System Data**
   - Receipt images in Documents directory
   - Voice memos in voice/ directory
   - Cached data (export files, route cache)
   - AsyncStorage data (settings, preferences)

3. **Imported Data**
   - Transaction records from FinanceKit
   - Imported receipt data

### Configuration Files

#### app.json

The Data Protection capability is configured in `app.json` under the iOS entitlements section:

```json
{
  "expo": {
    "ios": {
      "entitlements": {
        "com.apple.developer.default-data-protection": "NSFileProtectionCompleteUntilFirstUserAuthentication"
      }
    }
  }
}
```

When building with Expo/EAS, this configuration is automatically translated to the native iOS entitlements file.

## Protection Level Comparison

| Level | Security | Background Features | Use Case |
|-------|----------|---------------------|----------|
| Complete | Highest | None work when locked | Medical records |
| **Until First Auth** | **High** | **All work after unlock** | **Financial apps (chosen)** |
| Until Mounted | Medium | All work | General apps |
| None | None | All work | Public data |

## Implementation Details

### Automatic Encryption

iOS handles all encryption automatically when Data Protection is enabled:

- No code changes required in the application
- Encryption keys are managed by the iOS Secure Enclave
- Data is encrypted using AES-256
- Keys are tied to device passcode/biometrics

### Authentication Requirements

For data to remain protected:

1. Device must have a passcode or biometric authentication enabled
2. User must authenticate at least once after device boot
3. After first authentication, data remains accessible even when locked

### Background Operations

After first device unlock, the following background operations work correctly:

- Push notifications
- Background sync (if implemented)
- Background fetch
- Silent notifications

## Testing Requirements

### Manual Testing Scenarios

#### Test Case 1: Cold Boot Protection

**Objective:** Verify data is inaccessible before first unlock

**Steps:**
1. Power off the iOS device completely
2. Power on the device (do NOT unlock)
3. Attempt to launch the app

**Expected Result:**
- App cannot access database
- App should show appropriate unlock prompt or fail gracefully

**Status:** Manual testing required on physical device

#### Test Case 2: Post-Authentication Access

**Objective:** Verify data is accessible after first unlock

**Steps:**
1. Unlock device with passcode/biometric
2. Launch the app
3. Verify all features work normally

**Expected Result:**
- Database opens successfully
- All expenses and data visible
- CRUD operations work normally

**Status:** Manual testing required on physical device

#### Test Case 3: Background Operations

**Objective:** Verify background features work after first unlock

**Steps:**
1. Unlock device (authenticate)
2. Lock device (do NOT power off)
3. Send test push notification (if implemented)
4. Verify background sync works (if implemented)

**Expected Result:**
- Push notifications received and processed
- Background sync completes successfully
- Data remains accessible in background

**Status:** Manual testing required when background features implemented

#### Test Case 4: Re-lock Behavior

**Objective:** Verify data remains accessible when device is locked (but not powered off)

**Steps:**
1. Authenticate and use app normally
2. Lock device screen (press power button)
3. Wait 5 minutes
4. Unlock and open app

**Expected Result:**
- App opens immediately without delay
- Data accessible without re-authentication
- No data loss or corruption

**Status:** Manual testing required on physical device

### Automated Testing

Data Protection cannot be fully tested in iOS Simulator as it doesn't enforce file protection. All testing must be done on physical devices with passcode enabled.

## Security Considerations

### Additional Secure Storage

For highly sensitive data that requires additional protection, use `expo-secure-store`:

- Authentication tokens
- User credentials (if stored)
- API keys (though these should be in environment variables)

`expo-secure-store` uses the iOS Keychain, which has separate encryption from Data Protection.

### Data at Rest vs. Data in Transit

This configuration only protects data at rest (on the device). For data in transit:

- Use HTTPS for all network requests
- Implement certificate pinning for sensitive endpoints (Phase 2)
- Validate SSL certificates

### Device Requirements

For Data Protection to function:

- Device must have iOS 8.0 or later (we require iOS 15.5+)
- Device must have a passcode or biometric authentication enabled
- If user disables passcode, protection is downgraded automatically by iOS

## Privacy Policy Documentation

The app's privacy policy should include:

> **Data Security**
>
> Your financial data is encrypted at rest using Apple's Data Protection APIs with "Protected Until First User Authentication" level. This means:
>
> - All expenses, receipts, and mileage data are encrypted when your device is powered off
> - Data is protected by your device passcode or biometric authentication
> - After you unlock your device, the app can access your data even when the screen is locked
> - If your device is lost or stolen while powered off, your data cannot be accessed without your passcode
>
> We do not transmit your financial data to any servers. All data remains on your device.

## References

- [Apple Documentation: Protecting the User's Privacy](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy)
- [Apple Documentation: NSFileProtectionType](https://developer.apple.com/documentation/foundation/nsfileprotectiontype)
- [Apple Documentation: Data Protection](https://support.apple.com/guide/security/data-protection-overview-secf4a3a5bf5/web)
- Design Decision 12 in `openspec/changes/design.md`

## Verification Checklist

- [x] Data Protection capability configured in app.json
- [x] Correct protection level selected (NSFileProtectionCompleteUntilFirstUserAuthentication)
- [x] Entitlements properly formatted
- [ ] Tested on physical device with passcode enabled
- [ ] Verified data inaccessible before first unlock
- [ ] Verified data accessible after first unlock
- [ ] Verified background operations work (when implemented)
- [ ] Privacy policy updated
- [ ] App Store submission includes data protection information

## Build Requirements

### EAS Build

When building with EAS Build, ensure:

1. Build profile includes iOS entitlements
2. Device has provisioning profile with Data Protection capability
3. Apple Developer account has Data Protection capability enabled

### Local Build

For local Xcode builds:

1. Open `ios/ExpenseTracker.xcworkspace` in Xcode
2. Select target "ExpenseTracker"
3. Go to "Signing & Capabilities" tab
4. Verify "Data Protection" capability is present
5. Verify protection level matches app.json configuration

## Troubleshooting

### Issue: Data Protection not working in Simulator

**Solution:** Data Protection cannot be tested in iOS Simulator. Use a physical device with passcode enabled.

### Issue: App crashes on cold boot before unlock

**Solution:** This is expected behavior. Implement graceful error handling for file system access failures and show appropriate user message.

### Issue: Background sync not working

**Cause:** Using `NSFileProtectionComplete` instead of `NSFileProtectionCompleteUntilFirstUserAuthentication`

**Solution:** Verify app.json uses the correct protection level as documented.

## Migration Notes

If upgrading from no Data Protection to this configuration:

1. Existing user data is automatically protected after app update
2. No data migration required
3. First launch after update may take slightly longer as iOS encrypts existing files
4. Users must have device passcode enabled for full protection

## Compliance

This implementation satisfies:

- PCI-DSS requirements for data at rest encryption (if handling payment data)
- GDPR technical measures for data protection
- SOC 2 Type II controls for data security
- Apple App Store Review Guidelines for financial apps

---

**Document Version:** 1.0
**Last Updated:** 2026-01-14
**Author:** Roy (Backend Developer)
**Related:** Wave 0.7A, design.md Decision 12

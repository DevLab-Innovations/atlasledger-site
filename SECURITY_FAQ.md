# Security FAQ

Common questions about Atlas Ledger's security and data protection features.

---

## General Security

### Is my financial data secure?

Yes. Atlas Ledger uses **iOS Data Protection** to encrypt all your financial data at rest. This is the same encryption technology used by banking apps and is built into iOS at the operating system level.

### What if someone steals my phone?

If your device is stolen while powered off or locked:
- All your expense data is encrypted and inaccessible
- The thief cannot access your information without your device passcode
- Your data is protected even with physical access to the device

If your device is stolen while unlocked:
- Lock your device remotely using Find My iPhone
- Once locked, all data becomes encrypted again
- Use Find My to erase the device if needed

### Do I need to set up anything special?

No extra setup required! Data protection works automatically if:
- You have a device passcode or Face ID/Touch ID enabled
- You're using iOS 13.0 or later

**Important:** iOS Data Protection requires a passcode. If you don't have one set, your data is not encrypted.

---

## Data Storage

### Where is my data stored?

All data is stored **locally on your device only**:
- SQLite database in the app's sandbox
- Receipt photos in the app's document directory
- No cloud storage
- No external servers

### Does Atlas Ledger send my data anywhere?

**No.** Atlas Ledger is 100% local-first:
- We don't collect your data
- We don't track your usage
- We don't have servers that store your information
- Your data never leaves your device

### What happens if I delete the app?

When you delete Atlas Ledger:
- All local data is permanently deleted
- Receipt photos are removed
- There are no cloud backups to recover

**Recommendation:** Export your data to Excel before deleting the app if you need to keep records.

---

## Encryption Details

### What level of encryption do you use?

We use **NSFileProtectionCompleteUntilFirstUserAuthentication**, which means:
- Data is encrypted using AES-256 (military-grade encryption)
- Encryption keys are derived from your device passcode
- Data is protected when device is powered off or before first unlock
- Background operations work normally after you unlock your device once

### Can Apple access my data?

No. The encryption is tied to your device's hardware and passcode. Even Apple cannot decrypt your data without your passcode.

### What about backups?

**iCloud Backups:**
- If iCloud Backup is enabled, your encrypted data is included
- iCloud backups are also encrypted by Apple
- Restoration requires your Apple ID and device passcode

**iTunes Backups:**
- Encrypted backups preserve data protection
- Non-encrypted iTunes backups may not preserve full protection

---

## Comparison to Cloud Apps

### Why is local-first more secure than cloud storage?

**Local-First Advantages:**
- ✅ No data breach risk (no servers to hack)
- ✅ No third-party access to your data
- ✅ You control your data completely
- ✅ Works offline always
- ✅ No subscription required for data storage

**Cloud Storage Risks:**
- ❌ Company servers can be breached
- ❌ Third parties may access data (legal requests, analytics)
- ❌ Dependent on company staying in business
- ❌ Requires internet connection
- ❌ Often requires subscription

### What if I want to sync between devices?

Atlas Ledger currently does not support cloud sync to maintain maximum privacy and security.

**Alternatives:**
- Export to Excel and share via AirDrop
- Use iCloud backup to transfer data when setting up a new device
- Manually migrate using iTunes backup

**Future Consideration:** End-to-end encrypted cloud sync may be added as an optional feature, ensuring only you can decrypt your data.

---

## Technical Questions

### What encryption algorithm is used?

iOS Data Protection uses AES-256 (Advanced Encryption Standard with 256-bit keys):
- Industry standard encryption
- Used by governments and military
- Considered unbreakable with current technology

### How are encryption keys generated?

- Keys are derived from your device passcode using PBKDF2 (Password-Based Key Derivation Function 2)
- Hardware UID (unique to your device) is combined with your passcode
- Keys exist only in secure memory, never written to storage
- Keys are erased when device is locked or powered off

### Does encryption slow down the app?

No. iOS Data Protection has **zero performance impact** because:
- Encryption/decryption happens at the file system level
- Hardware acceleration (AES instructions in CPU)
- Transparent to the app and user
- No noticeable difference in speed

---

## Testing & Verification

### How can I verify my data is encrypted?

1. Add some test expenses with receipts
2. Power off your device completely
3. Power on (do not unlock)
4. Try to access data via iTunes or third-party tools
5. Data should be inaccessible until you unlock with passcode

### Does encryption work in the simulator?

No. The iOS Simulator does not support Data Protection. You must test on a physical device to verify encryption.

---

## Best Practices

### How can I maximize my data security?

1. **Use a strong passcode:**
   - At least 6 digits (preferably alphanumeric)
   - Don't share your passcode
   - Change it if you suspect it's compromised

2. **Enable biometric auth:**
   - Face ID or Touch ID adds convenience without sacrificing security
   - Biometric data never leaves your device

3. **Keep iOS updated:**
   - Security patches are released regularly
   - Update to latest iOS for best protection

4. **Enable Find My:**
   - Allows remote lock if device is lost
   - Can erase device remotely if needed

5. **Regular exports:**
   - Export your data monthly for backup
   - Store exports in a secure location (encrypted drive, password manager)

### What should I NOT do?

- ❌ Don't jailbreak your device (weakens security)
- ❌ Don't share your device passcode
- ❌ Don't use "1234" or simple passcodes
- ❌ Don't leave your device unlocked in public
- ❌ Don't create non-encrypted iTunes backups

---

## Troubleshooting

### App says "Data protection not available"

This happens when:
- You're running in the iOS Simulator (encryption not supported)
- Device passcode is not set (go to Settings > Face ID & Passcode)
- iOS version is too old (need iOS 13.0+)

**Solution:** Enable a passcode on your physical device.

### Can't access app after reboot

If you see this right after device boot:
- This is normal! Data Protection is working correctly
- Unlock your device with your passcode
- App will work normally after first unlock

### Lost my passcode, can I recover data?

Unfortunately, no. If you lose your device passcode:
- Your data is permanently encrypted and inaccessible
- Even Apple cannot recover it
- This is by design for maximum security

**Prevention:**
- Remember your passcode
- Set up Face ID/Touch ID as backup
- Export your data regularly

---

## Additional Resources

- [iOS Security Guide](https://support.apple.com/guide/security/welcome/web)
- [Apple Data Protection Documentation](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/encrypting_your_app_s_files)
- [Privacy Policy](PRIVACY_POLICY.md)
- [Data Protection Testing Guide](DATA_PROTECTION_TESTING.md) (for developers)

---

## Contact

If you have security concerns or questions not covered here:
- Open an issue on GitHub
- Review our open-source code
- Check for security updates in release notes

**Security Disclosure:** If you discover a security vulnerability, please report it responsibly via GitHub Issues with a "Security" label.

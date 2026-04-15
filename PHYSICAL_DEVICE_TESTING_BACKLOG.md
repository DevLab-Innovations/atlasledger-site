# Physical Device Testing Backlog

This document tracks all features that require manual testing on a physical iOS device. These tests cannot be performed in the simulator due to hardware dependencies.

**Last Updated:** 2026-02-01

---

## 🔴 High Priority - Blocking Production Release

### Wave 0.7: Data Protection (6 tests, ~75 minutes)

#### 0.7A.4: Data Inaccessibility When Device Locked (15 min)
**What:** Verify data is encrypted and inaccessible before first unlock after boot

**Test Steps:**
1. Power off device completely
2. Power on (DO NOT unlock with passcode)
3. Attempt to access app data (should fail)
4. Unlock device
5. Launch app - verify all data intact

**Expected:** Data inaccessible before first unlock, accessible after

**Why Physical Device:** Simulator doesn't support iOS Data Protection encryption

**Status:** ⏳ Pending

---

#### 0.7A.5: Background Sync After First Unlock (10 min)
**What:** Verify background operations work when device is locked (after first unlock)

**Test Steps:**
1. Add test expenses
2. Background the app
3. Lock device
4. Wait 1-2 minutes
5. Unlock and verify app state restored correctly

**Expected:** Background operations succeed while locked

**Why Physical Device:** Background task execution differs on device vs simulator

**Status:** ⏳ Pending

---

#### 0.7A.6: Push Notifications (Deferred)
**What:** Verify notifications work when device is locked

**Status:** 🔵 Deferred - Push notifications not implemented yet

---

#### 0.7A.8: Apple Developer Portal Verification (5 min)
**What:** Confirm Data Protection capability is registered with Apple

**Test Steps:**
1. Log into https://developer.apple.com/account/
2. Navigate to App ID
3. Verify "Data Protection" capability enabled

**Expected:** Capability shows as enabled in portal

**Why Required:** Manual portal access needed

**Status:** ⏳ Pending

---

#### 0.7A.9: TestFlight Build Submission (30 min)
**What:** Verify app passes App Store security review

**Test Steps:**
1. Archive release build
2. Upload to App Store Connect
3. Wait for processing
4. Verify no entitlement errors
5. Install via TestFlight
6. Test on device

**Expected:** Build uploads successfully, installs via TestFlight

**Why Physical Device:** TestFlight only works on real devices

**Status:** ⏳ Pending

---

#### 0.7A.10: Comprehensive Background Operations (15 min)
**What:** Test all background features with data protection

**Test Matrix:**
- App launch (unlocked) → ✅ Works
- App launch (locked after 1st unlock) → ✅ Works
- App launch (before first unlock) → ❌ Graceful error
- Database query (locked after 1st unlock) → ✅ Works
- File access receipts (locked) → ✅ Works

**Expected:** All operations work correctly or fail gracefully

**Why Physical Device:** Background behavior differs on device

**Status:** ⏳ Pending

---

## 🔴 High Priority - Recent Builds Requiring Testing

### Build 17/18: WAL Checkpoint & Data Persistence Fix

#### SQLite WAL Data Persistence Test (15 min)
**What:** Verify data is not lost when app is closed and reopened (WAL checkpoint fix)

**Background:** Build 17 added SQLite WAL checkpointing to prevent data loss when the app terminates before writes are flushed from the WAL file to the main database.

**Test Steps:**
1. Add 3-5 new expenses with various amounts
2. Add 2 income entries
3. Mark 1 bill as paid
4. Force close the app immediately (swipe up from app switcher)
5. Reopen app
6. Verify ALL data persists:
   - All expenses present with correct amounts
   - All income entries present
   - Bill still shows as paid
7. Repeat test 3 times to ensure consistency

**Expected:** 100% data persistence - no data loss on app restart

**Why Physical Device:** WAL behavior differs on real device vs simulator

**Status:** ⏳ Pending (Build 17/18 on TestFlight)

---

#### Background/Inactive Checkpoint Test (10 min)
**What:** Verify checkpoint triggers when app goes to background

**Test Steps:**
1. Add new expense
2. Press home button (background app)
3. Wait 5 seconds
4. Force kill app
5. Reopen - verify expense persists

**Expected:** Data saved even after force kill from background

**Status:** ⏳ Pending

---

### Build 18: Speech Recognition Privacy Description

#### Speech Recognition Permission Test (5 min)
**What:** Verify speech recognition permission prompt appears with correct description

**Test Steps:**
1. Fresh install Build 18
2. Navigate to expense form
3. Tap dictation/voice button (if available)
4. Verify permission prompt shows:
   - "Atlas uses on-device speech recognition to transcribe voice memos and enable hands-free voice commands."
5. Grant permission
6. Verify speech recognition works

**Expected:** Permission prompt appears with user-friendly description

**Status:** ⏳ Pending (Build 18 on TestFlight)

---

## 🟡 Medium Priority - Feature Validation

### Wave 4.5: Receipt Flow Enhancement (Code Complete)

#### Receipt-First Flow End-to-End Test (15 min)
**What:** Verify receipt scanning and OCR auto-fill works on real hardware

**Status:** ✅ Code Complete | ⏳ Physical Device Testing Pending

**Test Steps:**
1. Create new expense
2. Tap "Scan Receipt" button
3. Capture receipt photo with real camera in various lighting conditions:
   - Good lighting (bright indoor/outdoor)
   - Low lighting (dim room)
   - Mixed lighting (shadow on receipt)
4. Verify OCR processing completes
5. Check auto-filled fields:
   - Amount extracted correctly
   - Merchant name detected
   - Date extracted (if present)
   - Payment method extracted (VISA/MC/AMEX → Credit)
   - Category suggested based on merchant keywords
6. Save expense with receipt attached
7. Verify receipt thumbnail appears in expense list

**Expected Results:**
- ✅ Camera captures high-quality receipt images
- ✅ OCR processes within 5 seconds
- ✅ Amount extraction >90% accuracy on clear receipts
- ✅ Merchant name extracted correctly
- ✅ Payment method mapping works (card logos → payment type)
- ✅ Category auto-suggested based on merchant
- ✅ Receipt saved and accessible in expense detail

**Alternative Flow Test:**
1. Create expense manually (no receipt)
2. Fill in amount, merchant, category
3. Save expense
4. Open expense detail
5. Tap "Add Receipt"
6. Capture photo
7. Verify receipt attached to existing expense

**Why Physical Device:**
- Simulator camera uses sample images, not real camera
- Need to test OCR accuracy with actual receipt photos
- Camera quality and lighting conditions vary on device
- End-to-end flow validation with real hardware

**Status:** ⏳ Pending

---

### Wave 0.5: Accessibility (VoiceOver Testing)

#### VoiceOver Navigation Test (20 min)
**What:** Verify all screens are accessible with VoiceOver

**Test Steps:**
1. Enable VoiceOver on device (Settings > Accessibility)
2. Navigate through all screens using VoiceOver gestures
3. Verify all interactive elements have proper labels
4. Test form validation announcements
5. Test loading state announcements

**Expected:** All screens navigable and understandable with VoiceOver

**Why Physical Device:** VoiceOver behavior more accurate on device

**Status:** ⏳ Pending (optional - automated tests cover most)

---

### Wave 0.6: Form Polish (Haptic Feedback)

#### Haptic Feedback Verification (10 min)
**What:** Verify haptic feedback feels appropriate on real device

**Test Steps:**
1. Test bulk selection mode entry (heavy haptic)
2. Test item selection (medium haptic)
3. Test delete action (heavy haptic)
4. Test undo action (light haptic)
5. Test form submission (medium haptic)

**Expected:** Haptics feel natural and appropriate for each action

**Why Physical Device:** Simulator cannot produce haptic feedback

**Status:** ⏳ Pending

---

## 🟢 Low Priority - Nice to Have

### Camera & OCR (Already Tested, but could re-verify)

#### Camera Capture Quality (5 min)
**What:** Verify camera captures high-quality receipt images

**Test Steps:**
1. Capture receipts in various lighting conditions
2. Verify image quality is sufficient for OCR
3. Test flash functionality

**Expected:** Clear, readable receipt images

**Why Physical Device:** Simulator doesn't have real camera

**Status:** ✅ Likely working (already in production)

---

#### OCR Accuracy on Device (10 min)
**What:** Verify OCR works well with real camera photos

**Test Steps:**
1. Capture 5-10 different receipts
2. Verify amounts extracted correctly
3. Verify dates extracted correctly
4. Verify merchants extracted correctly

**Expected:** >90% accuracy on clear receipts

**Why Physical Device:** Different image quality than simulator

**Status:** ✅ Likely working (OCR has 113 passing tests)

---

## 📋 Testing Session Planning

### Quick Session (30 minutes)
**Focus:** Critical Wave 0.7 validation

1. 0.7A.4: Data encryption test (15 min)
2. 0.7A.5: Background operations (10 min)
3. 0.7A.8: Developer portal check (5 min)

**Output:** Wave 0.7 marked complete

---

### Comprehensive Session (2.5 hours)
**Focus:** All physical device validations

1. Wave 0.7 tests (40 min)
2. Wave 4.5 receipt flow test (15 min)
3. Wave 6 AI Assistance tests (30 min)
4. VoiceOver navigation (20 min)
5. Haptic feedback (10 min)
6. Camera/OCR re-verification (10 min)
7. TestFlight build submission (30 min - includes processing wait)

**Output:** Complete physical device testing backlog cleared

---

### Pre-Production Session (2 hours)
**Focus:** Full App Store submission preparation

1. All Wave 0.7 tests (75 min)
2. TestFlight submission and testing (30 min)
3. VoiceOver comprehensive audit (20 min)
4. Haptic feedback verification (10 min)
5. Performance testing (memory, battery) (20 min)
6. Screenshot capture for App Store (15 min)

**Output:** Ready for App Store submission

---

## 🔧 Testing Setup Requirements

### Hardware
- [ ] Physical iPhone or iPad (iOS 13.0+)
- [ ] Device passcode or Face ID/Touch ID enabled
- [ ] USB cable or Wi-Fi sync enabled
- [ ] Sufficient storage for test data and receipts

### Software
- [ ] Mac with Xcode installed
- [ ] Development provisioning profile
- [ ] Apple Developer account (paid membership for TestFlight)
- [ ] Device registered in developer portal

### Test Data
- [ ] 10-20 sample expenses with varying amounts
- [ ] 5-10 receipt photos (various merchants)
- [ ] 5 mileage entries with different distances
- [ ] Test account setup (if multi-account feature exists)

---

## 📊 Testing Status Dashboard

| Category | Tests | Completed | Status |
|----------|-------|-----------|--------|
| **Wave 0.7 (Data Protection)** | 6 | 0 | 🔴 Blocked |
| **Wave 4.5 (Receipt Flow)** | 1 | 0 | 🟡 Code Complete |
| **Wave 0.5 (Accessibility)** | 1 | 0 | 🟡 Optional |
| **Wave 0.6 (Haptic Feedback)** | 1 | 0 | 🟡 Optional |
| **Wave 6 (AI Assistance)** | 13 | 0 | 🟡 Optional |
| **Camera/OCR** | 2 | 0 | 🟢 Low Priority |
| **Total** | 24 | 0 | 0% |

---

## 🚀 Automation Opportunities

Some tests could eventually be automated:

**Possible with XCUITest:**
- ✅ VoiceOver navigation (accessibility identifier checks)
- ✅ Haptic feedback timing (can verify calls are made)
- ⚠️ Camera capture flow (can test UI, not actual camera)

**Not Automatable:**
- ❌ Data Protection encryption (requires device reboot)
- ❌ TestFlight submission (manual process)
- ❌ Apple Developer portal (external system)
- ❌ Actual haptic "feel" (subjective)

---

## 📝 Notes

- **Simulator Limitations:** iOS Simulator cannot test Data Protection, real camera, haptic feedback, or TestFlight
- **Timing:** Physical device tests should be batched into single sessions for efficiency
- **Priority:** Wave 0.7 is blocking production release; other tests are nice-to-have
- **Alternative:** Some validation will happen during App Store review process
- **Confidence:** High confidence in implementation due to automated test coverage (1,587 tests passing)

---

## ✅ Completion Criteria

**Minimum for Production (Wave 0.7 only):**
- [ ] Data encryption verified (0.7A.4)
- [ ] Background operations tested (0.7A.5, 0.7A.10)
- [ ] Developer portal verified (0.7A.8)
- [ ] TestFlight build submitted (0.7A.9)

**Complete Validation (All tests):**
- [ ] All Wave 0.7 tests passed
- [ ] VoiceOver navigation verified
- [ ] Haptic feedback tested
- [ ] Camera/OCR re-validated
- [ ] Performance benchmarks recorded

**Ready for App Store:**
- [ ] All minimum tests passed
- [ ] No crashes observed
- [ ] Privacy policy accessible
- [ ] Screenshots captured
- [ ] App Store metadata complete

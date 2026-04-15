# App Store Screenshot Completion Guide

**Status**: 2 of 5 screenshots completed
**Completion Time**: ~15 minutes (manual)
**Last Updated**: 2026-01-18

---

## Prerequisites

✅ **Sample data is already loaded in the database**
- 10 expense entries with realistic business expenses
- 5 mileage entries with business trips
- Data is persistent in simulator at: iPhone 16 Pro (Device ID: E362FB45-46B0-45A3-9185-3847F289C855)

✅ **App is ready to run**
- Expo dev server can be started with: `npx expo start`
- Simulator can be booted with: `xcrun simctl boot "iPhone 16 Pro"`

---

## Completed Screenshots ✅

### 01_home_dashboard.png
- **Status**: ✅ DONE
- **Location**: `docs/appstore/screenshots/01_home_dashboard.png`
- **Size**: 239 KB
- **Content**: Main expense list with sample data, search bar, filter chips, FAB button

### 03_expense_detail.png
- **Status**: ✅ DONE
- **Location**: `docs/appstore/screenshots/03_expense_detail.png`
- **Size**: 206 KB
- **Content**: Detailed view of $45.99 Starbucks expense with all fields

---

## Remaining Screenshots (Manual Capture Required)

### 02_add_expense.png - Add Expense Form ⏳

**How to capture**:
1. Boot simulator: `xcrun simctl boot "iPhone 16 Pro"`
2. Start Expo: `npx expo start` (in project root)
3. Open Expo Go on simulator and select "Expense Ledger"
4. Wait for app to load to expense list screen
5. **Tap the purple "+" FAB button** in bottom right
6. Once add expense form opens, take screenshot:
   ```bash
   xcrun simctl io booted screenshot docs/appstore/screenshots/02_add_expense.png
   ```

**What should be visible**:
- Empty expense form with all input fields
- Amount input at top
- Date picker
- Category dropdown
- Merchant input
- Description input
- Payment method dropdown
- Camera/receipt button
- Save button

**Tips**:
- Form should be empty (new expense mode)
- All fields should be clearly visible
- No keyboard should be showing
- If keyboard appears, tap outside fields to dismiss

---

### 04_settings.png - Settings Screen ⏳

**How to capture**:
1. From any screen in the app
2. **Tap the "Settings" tab** (gear icon, far right in bottom tab bar)
3. Wait for settings screen to load
4. Take screenshot:
   ```bash
   xcrun simctl io booted screenshot docs/appstore/screenshots/04_settings.png
   ```

**What should be visible**:
- App version and build number
- Settings sections:
  - Categories management
  - Backup & Export
  - Mileage rate configuration
  - Default currency
  - About section
- Clean, organized list layout

---

### Optional: Additional Recommended Screenshots

#### 05_mileage_list.png (Recommended)

**How to capture**:
1. **Tap the "Mileage" tab** (car icon, second from left in tab bar)
2. Wait for mileage list to load (5 entries should be visible)
3. Take screenshot:
   ```bash
   xcrun simctl io booted screenshot docs/appstore/screenshots/05_mileage_list.png
   ```

**What should be visible**:
- List of 5 mileage entries with distances and purposes
- Total mileage amount calculations
- Dates of trips
- FAB button for adding new mileage entry

---

#### 06_reports_screen.png (Recommended)

**How to capture**:
1. **Tap the "Reports" tab** (chart icon, third from left in tab bar)
2. Wait for reports to load
3. Take screenshot:
   ```bash
   xcrun simctl io booted screenshot docs/appstore/screenshots/06_reports_screen.png
   ```

**What should be visible**:
- Summary statistics (total expenses, expense count, etc.)
- Charts or graphs showing spending by category
- Date range selector
- Export options

---

## Screen Recording: Receipt Capture Flow 🎥

### receipt_capture_demo.mp4 (15-30 seconds)

**Prerequisites**:
- Have a sample receipt image ready (PDF or photo)
- Can use test receipts from `e2e/fixtures/` if available

**How to record**:

1. **Start recording**:
   ```bash
   xcrun simctl io booted recordVideo docs/appstore/screenshots/receipt_capture_demo.mp4
   ```

2. **Perform these actions on simulator**:
   - Open Add Expense form (tap + FAB)
   - Tap the camera/receipt button
   - Select "Choose from Library" (or take photo if camera works)
   - Select a sample receipt image
   - Wait for OCR processing (loading indicator)
   - Watch as expense fields auto-fill with extracted data
   - Tap "Save" to save the expense

3. **Stop recording**: Press `Ctrl+C` in terminal where recording started

**Recording tips**:
- Keep actions smooth and deliberate (not too fast)
- Show clear user flow
- Wait 1-2 seconds on auto-filled form so viewer can see the data
- Total time should be 15-30 seconds
- No need for audio

**Fallback if OCR doesn't work in simulator**:
- Record just the receipt selection process
- Or use physical device for recording (more reliable)

---

## Quick Start Commands

### Boot Everything
```bash
# From project root
cd /Users/neptune/repos/agenticSdlc

# Boot simulator
xcrun simctl boot "iPhone 16 Pro"
open -a Simulator

# Start Expo dev server
npx expo start

# In Simulator: Open Expo Go > Tap "Expense Ledger"
```

### Take Screenshot
```bash
# General screenshot command
xcrun simctl io booted screenshot <output-path>.png

# Example
xcrun simctl io booted screenshot docs/appstore/screenshots/04_settings.png
```

### Record Video
```bash
# Start recording
xcrun simctl io booted recordVideo docs/appstore/screenshots/receipt_capture_demo.mp4

# Stop recording: Press Ctrl+C
```

---

## Device Information

**Simulator**: iPhone 16 Pro
**iOS Version**: 18.2
**Resolution**: 1206x2622 pixels (native)
**Device ID**: E362FB45-46B0-45A3-9185-3847F289C855

**Database Location** (sample data is here):
```
/Users/neptune/Library/Developer/CoreSimulator/Devices/E362FB45-46B0-45A3-9185-3847F289C855/data/Containers/Data/Application/96E9FA06-6A92-40CF-A8CA-4F5CD8A8E92E/Documents/SQLite/expense_tracker.db
```

---

## Screenshot Requirements for App Store

### Specifications
- **Resolution**: 1206x2622 (iPhone 16 Pro native) ✅
- **Format**: PNG ✅
- **Color Space**: sRGB
- **File Size**: Under 2 MB each ✅
- **Quantity**: 5-10 screenshots recommended

### Composition Guidelines
- ✅ Show realistic, professional sample data (already loaded)
- ✅ Clean UI with no errors or placeholder text
- ✅ Avoid showing personal information
- ✅ Consistent time (status bar should show same time - currently 9:23-9:26)
- ✅ Full battery indicator (already showing)
- ✅ Good Wi-Fi signal (already showing)

---

## Troubleshooting

### App Won't Load Sample Data
**Symptom**: Expense list shows "Add your first expense" empty state

**Solution**:
The data is in the database but app may need restart:
```bash
# Terminate app
xcrun simctl terminate booted com.devlabinnovations.atlasledger

# Relaunch
xcrun simctl launch booted com.devlabinnovations.atlasledger
```

### Simulator Running Slow
**Solution**:
```bash
# Shutdown all simulators
xcrun simctl shutdown all

# Boot only the one you need
xcrun simctl boot "iPhone 16 Pro"
```

### Can't Find Expo App in Simulator
**Solution**:
1. Make sure Expo dev server is running (`npx expo start`)
2. Open Expo Go app (may need to install from App Store)
3. Shake device (Ctrl+Cmd+Z) to open Expo menu
4. Or scan QR code from terminal

### Screenshot Command Fails
**Check**:
```bash
# Verify simulator is booted
xcrun simctl list devices booted

# Should show iPhone 16 Pro in Booted state
```

---

## Final Checklist

Before submitting to App Store Connect:

- [ ] All 5+ screenshots captured in correct resolution
- [ ] Screenshots show clean UI with realistic data
- [ ] No developer warnings or debug info visible
- [ ] Screen recording captured (optional but recommended)
- [ ] Files saved to `docs/appstore/screenshots/`
- [ ] Screenshots reviewed for quality and content
- [ ] Consider rebuilding as production build (no Expo Go) for final submission

---

## Next Steps After Screenshots Complete

1. Review all screenshots for quality
2. Optionally add text overlays or marketing copy (App Store allows this)
3. Upload to App Store Connect in Screenshots section
4. Add localized screenshots for other languages if needed
5. Complete App Store listing with description, keywords, etc.

---

**Questions?** See `/Users/neptune/repos/agenticSdlc/QA_REPORT_PR24.md` for detailed QA report.

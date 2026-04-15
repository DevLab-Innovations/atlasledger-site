# Visual Testing Checklist: PR #43 Purple/Coral Theme
**Date**: 2026-01-22
**Tester**: QA Tester Agent
**Branch**: dev (post PR #43 merge)
**Feature**: New Atlas Purple/Coral brand theme

---

## Executive Summary

This document provides a comprehensive visual testing checklist for the new Purple/Coral theme implementation. All screens and components must be verified for correct color application, accessibility compliance, and visual consistency.

---

## Test Environment

- **Device**: iOS Simulator (iPhone 15 Pro)
- **iOS Version**: 17.x
- **Branch**: dev
- **Commit**: 7a0d889 (PR #43 merged)
- **Test Date**: 2026-01-22

---

## Color Palette Reference (from theme/index.ts)

### Primary Colors
- **Primary Purple**: #7C3AED (Vibrant violet)
- **Primary Light**: #A78BFA (Light violet for hover)
- **Primary Dark**: #5B21B6 (Deep purple for active)
- **Primary Background**: #F5F3FF (Ultra-light violet)

### Secondary Colors
- **Secondary Orange**: #F97316 (Vibrant orange/coral)
- **Secondary Light**: #FB923C (Light coral)
- **Secondary Dark**: #C2410C (Deep burnt orange)
- **Secondary Background**: #FFF7ED (Ultra-light peach)

### Semantic Colors
- **Success**: #059669 (Emerald green)
- **Warning**: #D97706 (Amber)
- **Error**: #DC2626 (Red)
- **Info**: #0891B2 (Teal)

---

## Screen-by-Screen Visual Testing

### 1. Dashboard/Home Screen (DashboardScreen.tsx)

**Component**: DashboardScreen.tsx

**Test Cases**:
- [ ] Screen background uses warm stone gray (#FAFAF9)
- [ ] Card surfaces are white (#FFFFFF) with proper elevation
- [ ] Cash flow summary uses purple/orange/teal for metrics
- [ ] Chart colors follow chartColors array (purple, orange, teal sequence)
- [ ] Buttons use primary purple with white text
- [ ] Tab navigation shows active tab in purple
- [ ] Skeleton loaders use stone-200/stone-100 colors
- [ ] Text hierarchy: primary text (#1C1917), secondary text (#57534E)

**Expected Results**:
- Purple is prominent in navigation and primary actions
- Orange accents appear on CTA buttons or highlights
- Charts are colorful and use the full palette
- All text is readable with sufficient contrast

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Dashboard overview
- Chart detail view
- Empty state (if applicable)

---

### 2. Money Hub Screen (MoneyHubScreen.tsx)

**Component**: MoneyHubScreen.tsx

**Test Cases**:
- [ ] Three cards (Income, Bills, Expenses) render with proper colors
- [ ] Card borders and dividers use stone-200 (#E7E5E4)
- [ ] Icons and text have correct color hierarchy
- [ ] Add buttons use purple primary color
- [ ] Premium badge/indicator uses appropriate styling
- [ ] Account switcher (if visible) uses theme colors

**Expected Results**:
- Consistent card styling across all three sections
- Purple for primary actions
- Clear visual hierarchy

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full Money Hub view
- Account switcher (if applicable)

---

### 3. Expense List Screen (ExpenseListScreen.tsx)

**Component**: ExpenseListScreen.tsx

**Test Cases**:
- [ ] Filter chips use purple for active state
- [ ] Search bar uses proper border colors (default: stone-200, focused: purple)
- [ ] Expense list items have readable text on white cards
- [ ] Category badges/chips use theme colors appropriately
- [ ] FAB (Floating Action Button) uses primary purple
- [ ] Empty state illustration and text use theme colors
- [ ] Pull-to-refresh indicator uses purple
- [ ] Delete/undo snackbar uses appropriate colors

**Expected Results**:
- Active filters clearly indicated with purple
- Clean, readable list with proper spacing
- Purple FAB stands out for primary action

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- List with multiple expenses
- Active filter chips
- Empty state
- Search active state

---

### 4. Expense Form Screen (ExpenseFormScreen.tsx)

**Component**: ExpenseFormScreen.tsx

**Test Cases**:
- [ ] Form inputs use stone-200 border by default
- [ ] Focused inputs show purple border (#7C3AED)
- [ ] Error states show red border (#DC2626)
- [ ] Save button uses primary purple with white text
- [ ] Cancel button uses appropriate text/outline style
- [ ] Keyboard toolbar (if visible) uses theme colors
- [ ] Date picker shows purple accent
- [ ] Currency input displays correctly
- [ ] Draft auto-save indicator (if visible) uses info color
- [ ] Camera button/icon uses theme colors

**Expected Results**:
- Clear focus states with purple
- Error states are immediately visible in red
- Form is easy to navigate with good contrast

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Empty form (default state)
- Focused input field
- Form with validation errors
- Date picker modal
- Category selector

---

### 5. Expense Detail Screen (ExpenseDetailScreen.tsx)

**Component**: ExpenseDetailScreen.tsx

**Test Cases**:
- [ ] Header background uses appropriate theme color
- [ ] Amount displays prominently with good contrast
- [ ] Edit/Delete buttons use theme colors (purple for edit, red for delete)
- [ ] Category badge uses theme-appropriate color
- [ ] Receipt image (if present) displays correctly
- [ ] Dividers use stone-100 (#F5F5F4)
- [ ] Text hierarchy follows theme (primary, secondary, tertiary)

**Expected Results**:
- Clean, professional detail view
- Clear action buttons with appropriate colors
- Receipt images display without color distortion

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full detail view
- Detail with receipt image
- Delete confirmation modal

---

### 6. Mileage List Screen (MileageListScreen.tsx)

**Component**: MileageListScreen.tsx

**Test Cases**:
- [ ] Similar to Expense List but for mileage entries
- [ ] Filter chips use purple for active state
- [ ] FAB mini animation uses theme colors
- [ ] List items use white cards on stone-50 background
- [ ] Category/distance badges readable

**Expected Results**:
- Consistent with Expense List styling
- Purple accents for primary actions and filters

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- List with mileage entries
- Active filters

---

### 7. Mileage Form Screen (MileageFormScreen.tsx)

**Component**: MileageFormScreen.tsx

**Test Cases**:
- [ ] Form styling consistent with ExpenseFormScreen
- [ ] Purple focus states on inputs
- [ ] Map integration (if visible) doesn't conflict with theme
- [ ] Distance calculator uses theme colors

**Expected Results**:
- Consistent form UX with expense forms
- Clear focus and error states

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Empty form
- Form with validation errors

---

### 8. Reports Screen (ReportsScreen.tsx)

**Component**: ReportsScreen.tsx

**Test Cases**:
- [ ] Period selector uses purple for active period
- [ ] Charts use chartColors array (purple, orange, teal, etc.)
- [ ] Export button uses secondary orange color for prominence
- [ ] Period comparison card uses info/warning colors appropriately
- [ ] Empty state for no data uses theme colors
- [ ] Skeleton loaders during data fetch

**Expected Results**:
- Vibrant, colorful charts using full palette
- Orange export button stands out as secondary CTA
- Data visualizations are clear and distinguishable

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full reports view with charts
- Period comparison card
- Export preview modal
- Empty state

---

### 9. Settings Screen (SettingsScreen.tsx)

**Component**: SettingsScreen.tsx

**Test Cases**:
- [ ] Section headers use secondary text color (#57534E)
- [ ] List items use white surface on stone-50 background
- [ ] Switches/toggles use purple for active state
- [ ] Chevron icons use appropriate gray
- [ ] Dividers use stone-100
- [ ] Destructive actions (e.g., "Delete All Data") use red
- [ ] Premium badge uses orange/purple gradient or appropriate color

**Expected Results**:
- Clean, organized settings list
- Purple accents for active states
- Red for destructive actions

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full settings list
- Toggle in active state
- Destructive action confirmation

---

### 10. Income List/Form Screens

**Components**: IncomeListScreen.tsx, IncomeFormScreen.tsx

**Test Cases**:
- [ ] Consistent styling with Expense screens
- [ ] Purple primary actions
- [ ] Forms use same input styling
- [ ] FAB uses purple
- [ ] Income-specific icon colors match theme

**Expected Results**:
- Consistent with other list/form screens
- Theme applied uniformly

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Income list
- Income form

---

### 11. Bills List/Form Screens

**Components**: BillsListScreen.tsx, BillFormScreen.tsx

**Test Cases**:
- [ ] Bill cards show due dates with appropriate warning colors
- [ ] "Mark as Paid" button uses success green
- [ ] Overdue bills show error/warning indicators
- [ ] Recurring bill indicator uses info color
- [ ] Form inputs consistent with other forms

**Expected Results**:
- Clear visual indicators for bill status (paid, due soon, overdue)
- Consistent form UX

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Bills list with various statuses
- Bill form
- "Mark as Paid" confirmation

---

### 12. Categories Screen (CategoriesScreen.tsx)

**Component**: CategoriesScreen.tsx

**Test Cases**:
- [ ] Category chips/cards use theme colors
- [ ] Custom categories show with appropriate colors
- [ ] Add category button uses purple
- [ ] Category icons display correctly

**Expected Results**:
- Categories are visually distinct
- Purple for primary actions

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full category list

---

### 13. Onboarding Screen (OnboardingScreen.tsx)

**Component**: OnboardingScreen.tsx

**Test Cases**:
- [ ] Onboarding slides use purple/orange brand colors
- [ ] "Get Started" button uses primary purple
- [ ] Skip button uses text color
- [ ] Pagination dots use purple for active page
- [ ] Illustrations/icons don't clash with new theme

**Expected Results**:
- Welcoming, branded first experience
- Purple establishes brand identity immediately

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- First onboarding slide
- Final slide with CTA

---

### 14. Lock Screen (LockScreen.tsx)

**Component**: LockScreen.tsx

**Test Cases**:
- [ ] PIN pad uses theme colors
- [ ] Active PIN indicators use purple
- [ ] Biometric prompt uses theme colors
- [ ] Error messages use error red
- [ ] Background uses surface/background color

**Expected Results**:
- Clean, secure-looking lock screen
- Purple accents maintain brand identity

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Lock screen with PIN pad
- Biometric prompt

---

### 15. Paywall Screen (PaywallScreen.tsx)

**Component**: PaywallScreen.tsx

**Test Cases**:
- [ ] Premium feature cards use gradient or purple/orange accents
- [ ] "Upgrade to Premium" button uses secondary orange for prominence
- [ ] Feature list checkmarks use success green
- [ ] Pricing information readable with good contrast
- [ ] Close/dismiss button clearly visible

**Expected Results**:
- Compelling, premium-looking upgrade screen
- Orange CTA stands out
- Purple reinforces premium brand

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full paywall screen
- Feature list
- Pricing options

---

### 16. Account Management Screens

**Components**: CreateAccountScreen.tsx, EditAccountScreen.tsx

**Test Cases**:
- [ ] Color picker component displays correctly
- [ ] Account switcher uses theme colors
- [ ] Save button uses primary purple
- [ ] Delete account uses error red
- [ ] Account icons/avatars use custom colors with proper contrast

**Expected Results**:
- Clean account management UI
- Color picker integrates well with theme

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Create account screen
- Edit account screen
- Color picker modal

---

### 17. Camera Screen (CameraScreen.tsx)

**Component**: CameraScreen.tsx

**Test Cases**:
- [ ] Camera controls (capture, cancel) use theme colors
- [ ] Flash/settings icons use appropriate gray or white
- [ ] OCR processing indicator uses purple or info color
- [ ] Overlay/guidelines don't clash with camera view

**Expected Results**:
- Functional camera with unobtrusive theme colors
- Clear control buttons

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Camera view with controls

---

### 18. Image Viewer Screen (ImageViewerScreen.tsx)

**Component**: ImageViewerScreen.tsx

**Test Cases**:
- [ ] Background uses dark backdrop
- [ ] Close button clearly visible on dark background
- [ ] Zoom controls (if any) use theme colors
- [ ] Image displays without color distortion

**Expected Results**:
- Clean image viewing experience
- Theme doesn't interfere with image viewing

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Full-screen image view

---

### 19. Privacy Policy Screen (PrivacyPolicyScreen.tsx)

**Component**: PrivacyPolicyScreen.tsx

**Test Cases**:
- [ ] Text uses primary text color with good contrast
- [ ] Links (if any) use purple
- [ ] Background uses surface color
- [ ] Scrolling doesn't reveal color issues

**Expected Results**:
- Readable legal text
- Consistent with app theme

**Status**: ⏳ Pending Manual Verification

**Screenshot Required**:
- Top of privacy policy
- Scrolled view

---

## Component-Level Visual Testing

### Shared Components

#### BackupReminderBanner.tsx
- [ ] Banner uses warning amber (#D97706) background
- [ ] Warning background (#FEF3C7) for lighter background
- [ ] "Export Now" button uses appropriate color
- [ ] "Dismiss" button uses text color
- [ ] Icon displays correctly

#### ColorPicker.tsx
- [ ] Color swatches display correctly
- [ ] Selected color shows purple border/indicator
- [ ] Custom colors can be selected

#### SearchBar.tsx
- [ ] Default border: stone-200
- [ ] Focused border: purple
- [ ] Clear button uses appropriate gray
- [ ] Search icon uses gray

#### FilterModal.tsx
- [ ] Modal background uses surface color
- [ ] Apply button uses purple
- [ ] Clear filters uses text color
- [ ] Chips use purple for selected state

#### SkeletonLoader.tsx
- [ ] Base color: stone-200 (#E7E5E4)
- [ ] Highlight color: stone-100 (#F5F5F4)
- [ ] Shimmer effect uses white with opacity
- [ ] Animations smooth and not jarring

#### AccountSwitcher.tsx
- [ ] Account cards use white surface
- [ ] Active account indicated with purple border
- [ ] Account colors display correctly
- [ ] Add account button uses purple

#### ErrorBoundary.tsx
- [ ] Error screen uses error red for icon/message
- [ ] "Reload App" button uses purple
- [ ] Background uses surface color

#### PINPad.tsx
- [ ] Number buttons use appropriate colors
- [ ] Active/pressed state uses purple ripple
- [ ] Delete button uses appropriate icon color
- [ ] PIN dots use purple when filled

#### ExportPreviewModal.tsx
- [ ] Modal uses surface background
- [ ] Export button uses secondary orange
- [ ] Cancel button uses text color
- [ ] Preview content readable

#### PeriodComparisonCard.tsx
- [ ] Up/down indicators use success/error colors
- [ ] Card uses white surface
- [ ] Text hierarchy correct

---

## Accessibility Testing (WCAG 2.1 AA Compliance)

### Contrast Ratios to Verify

| Foreground | Background | Expected Ratio | Standard | Status |
|------------|------------|----------------|----------|--------|
| Primary Purple (#7C3AED) | White | 4.6:1 | AA ✓ | ⏳ |
| Secondary Orange (#F97316) | White | 3.5:1 | AA Large Text ✓ | ⏳ |
| Text (#1C1917) | White | 19.3:1 | AAA ✓ | ⏳ |
| Text Secondary (#57534E) | White | 7.8:1 | AAA ✓ | ⏳ |
| Success (#059669) | White | 4.5:1 | AA ✓ | ⏳ |
| Warning (#D97706) | White | 4.7:1 | AA ✓ | ⏳ |
| Error (#DC2626) | White | 4.6:1 | AA ✓ | ⏳ |
| White | Primary Purple | 4.6:1 | AA ✓ | ⏳ |
| White | Secondary Orange | 3.5:1 | AA Large Text ✓ | ⏳ |

### Manual Accessibility Checks

#### Text Readability
- [ ] All body text meets AA contrast (4.5:1 minimum)
- [ ] Large text (18pt+ or 14pt+ bold) meets AA (3:1 minimum)
- [ ] Secondary orange NOT used for small body text
- [ ] Text remains readable on all backgrounds

#### Focus Indicators
- [ ] All interactive elements show purple focus border
- [ ] Focus borders are at least 2px thick
- [ ] Focus state clearly distinguishable from default state
- [ ] Tab order is logical and visible

#### Button Contrast
- [ ] Primary purple buttons have white text (4.6:1 contrast)
- [ ] Secondary orange buttons have white text (3.5:1 contrast)
- [ ] Disabled buttons have sufficient contrast to indicate state
- [ ] Buttons are easily tappable (minimum 44x44pt)

#### Color-Blind Accessibility
Test with iOS Accessibility > Display > Color Filters:
- [ ] Protanopia (red-blind) - purple/orange still distinguishable
- [ ] Deuteranopia (green-blind) - success/error states clear
- [ ] Tritanopia (blue-blind) - info/warning states clear
- [ ] Monochromacy (grayscale) - elements distinguishable by more than just color

#### Dynamic Type
Test at different text sizes (Settings > Display & Brightness > Text Size):
- [ ] Minimum size: Text remains readable
- [ ] Maximum size: Layout doesn't break, text doesn't truncate unexpectedly
- [ ] Colors maintain contrast at all text sizes

#### VoiceOver Testing
Enable VoiceOver and verify:
- [ ] Color changes are accompanied by text/label changes
- [ ] Interactive elements announce correctly
- [ ] Focus indicators are visible for VoiceOver users
- [ ] Color-coded information has text alternatives

---

## Edge Case Testing

### Long Text / Overflow
- [ ] Long expense descriptions don't break layout
- [ ] Long category names remain readable
- [ ] Colors remain correct when text wraps

### Empty States
- [ ] Empty expense list uses appropriate colors/illustrations
- [ ] Empty report screen uses theme colors
- [ ] No data charts show appropriate message

### Error States
- [ ] Network error messages use error red
- [ ] Form validation errors use error red
- [ ] Inline error messages readable

### Loading States
- [ ] Skeleton loaders use stone-200/stone-100
- [ ] Loading spinners use purple
- [ ] Pull-to-refresh indicator uses purple

### Dark Mode (Future-Proofing)
Note: Dark mode not implemented yet, but verify light mode is consistent:
- [ ] All screens use light mode colors consistently
- [ ] No dark mode colors bleeding through

---

## Interactive State Testing

### Button States
For all button types, verify:
- [ ] **Default**: Primary purple or secondary orange
- [ ] **Pressed**: Darker shade (primaryDark #5B21B6 or secondaryDark #C2410C)
- [ ] **Disabled**: Stone-400 (#A8A29E) with reduced opacity
- [ ] **Ripple**: Purple ripple (rgba(124, 58, 237, 0.12)) on primary, orange ripple on secondary

### Input States
For all text inputs, verify:
- [ ] **Default**: Stone-200 border (#E7E5E4)
- [ ] **Focused**: Purple border (#7C3AED), thicker (2px)
- [ ] **Error**: Red border (#DC2626)
- [ ] **Disabled**: Stone-300 background, stone-400 text
- [ ] **Placeholder**: Stone-500 (#78716C)

### List Item States
- [ ] **Default**: White surface
- [ ] **Pressed**: Overlay with rgba(0, 0, 0, 0.04)
- [ ] **Selected**: Purple border or background
- [ ] **Swipe actions**: Use appropriate semantic colors (green for success, red for delete)

---

## Regression Testing

Verify that existing functionality still works correctly:

### Core Functionality
- [ ] Add new expense (full flow)
- [ ] Edit existing expense
- [ ] Delete expense with undo
- [ ] Add mileage entry
- [ ] View expense details
- [ ] Filter expenses by category
- [ ] Search expenses
- [ ] View reports and charts
- [ ] Export data (CSV/Excel)
- [ ] Take photo with OCR
- [ ] Set up biometric lock
- [ ] Change settings
- [ ] Create/edit account (Premium)
- [ ] Add income entry (Premium)
- [ ] Add bill and mark as paid (Premium)

### Navigation
- [ ] Tab navigation works correctly
- [ ] Back navigation preserves state
- [ ] Deep linking (if applicable)
- [ ] Modal dismissal works

### Data Persistence
- [ ] Data saves correctly
- [ ] Draft auto-save works
- [ ] Settings persist
- [ ] Search query persists when navigating

### Performance
- [ ] Scrolling is smooth
- [ ] Animations don't stutter
- [ ] App launches quickly
- [ ] No color-related performance issues

---

## Known Issues / Limitations

### Pre-Existing Issues (Not Theme-Related)
- Some test warnings about `act(...)` wrapping (pre-existing, not blocking)
- Settings service console errors in tests (mocked behavior, not production issue)

### Theme-Specific Issues to Watch For
- [ ] Any color combinations that fail contrast checks
- [ ] Any screens with hardcoded old colors
- [ ] Any third-party components not respecting theme

---

## Testing Tools

### Manual Testing
- iOS Simulator (iPhone 15 Pro)
- Accessibility Inspector (Xcode)
- VoiceOver (iOS)
- Color Filters (iOS Settings)

### Automated Testing
- Jest unit tests (718 tests passing)
- TypeScript compilation
- ESLint checks

### Contrast Checking
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- macOS Color Contrast Analyser app
- Chrome DevTools Accessibility tab

---

## Sign-Off Checklist

Before approving for production:

- [ ] All automated tests pass (718/718)
- [ ] No new TypeScript errors
- [ ] All screens visually verified on simulator
- [ ] Accessibility contrast requirements met
- [ ] VoiceOver testing completed
- [ ] Color-blind simulation tested
- [ ] Dynamic Type tested (min and max sizes)
- [ ] No regressions in core functionality
- [ ] No hardcoded old colors remaining
- [ ] Theme is consistently applied across all screens
- [ ] Performance is acceptable
- [ ] Screenshots documented

---

## Next Steps After Verification

1. **If all tests pass**:
   - Create final QA report
   - Approve for promotion to `main`
   - Notify DevOps for production release

2. **If issues found**:
   - Document bugs with severity ratings
   - Create bug reports with reproduction steps
   - Hand off to dev team for fixes
   - Re-test after fixes applied

---

## Notes

This checklist will be used during manual verification on the iOS simulator. Results will be documented in the final QA report.

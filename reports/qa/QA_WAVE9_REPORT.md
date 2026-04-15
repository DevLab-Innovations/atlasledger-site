# Wave 9 QA Validation Report — Atlas Ledger

**Date**: 2026-03-08
**Tester**: qa-tester agent
**Branch**: main
**Wave**: 9 (validating Wave 8.5 completeness)

---

## Test Summary

| Category | Result |
|----------|--------|
| Automated test suite | PASS |
| Accessibility basics | PASS with notes |
| Screen/header consistency | PASS |
| Data integrity (ThemeContext + SettingsService) | PASS with notes |
| Accent color reactivity (frozen StyleSheet) | FAIL — 10 occurrences in 9 files |

### Automated Test Results
- **Total Suites**: 165 (164 passed, 1 skipped)
- **Total Tests**: 3414 (3359 passed, 55 skipped, 0 failed)
- **Skipped Suite**: `src/__tests__/services/FinanceKitService.test.ts` — intentionally skipped with `describe.skip` because FinanceKit requires a native iOS bridge unavailable in Jest. This is expected and pre-existing.
- **Worker exit warning**: Jest reports one worker failed to exit gracefully. Root cause is `AudioService.ts` using `setTimeout`-based delays (L286, L372) that hold the event loop open after tests complete. This is a pre-existing test hygiene issue, not a new regression. It does not affect production behavior.

**Verdict: No new test failures introduced by Wave 8.5.**

---

## Bug Reports

---

### Bug 1: Frozen accent color in 10 StyleSheet.create() entries across 9 files

**Severity**: Medium (visible UX inconsistency for users who change accent color)
**Component**: Multiple screens and components

#### Description
`StyleSheet.create()` is evaluated once at module load time. When the static `colors` import from `@/theme` is used inside `StyleSheet.create()`, the colors are frozen at the default Violet accent. Changing the accent color at runtime (via `ThemeContext.setAccentColor`) correctly updates all _inline_ style references to `themeColors.primary`, but the `StyleSheet.create()` definitions remain Violet regardless.

This creates a split appearance: some primary-colored elements update to the new accent while nearby elements stay Violet.

#### Steps to Reproduce
1. Open the app and navigate to Settings
2. Change the accent color from Violet to any other preset (e.g., Ocean or Ember)
3. Navigate to the following screens and observe the inconsistency

#### Affected Files and Elements

| File | Line | Element | Frozen Color Usage |
|------|------|---------|-------------------|
| `src/screens/ExpenseFormScreen.tsx` | 1840 | Category indicator chip left border | `borderLeftColor: colors.primary` |
| `src/screens/CategoriesScreen.tsx` | 446 | FAB button background | `backgroundColor: colors.primary` |
| `src/screens/MileageFormScreen.tsx` | 1271 | "Tracked" badge background | `backgroundColor: colors.primary` |
| `src/screens/IncomeListScreen.tsx` | 1177–1178 | Selected income item highlight | `backgroundColor: colors.primaryLight`, `borderLeftColor: colors.primary` |
| `src/screens/PayPeriodBillsScreen.tsx` | 575 | Count chip and pending card left border | `backgroundColor: colors.primary` |
| `src/screens/PaywallScreen.tsx` | 744, 765 | Current plan banner border + tier card border | `borderLeftColor/borderColor: colors.primary` |
| `src/screens/ReportResultScreen.tsx` | 308 | FAB button background | `backgroundColor: colors.primary` (no ThemeContext used at all in this file) |
| `src/components/PaycheckAllocationWizard.tsx` | 405, 452 | Total row top borders | `borderTopColor: colors.primary` |
| `src/components/PendingIncomeSection.tsx` | 666, 686 | Count chip background + pending card border | `backgroundColor/borderLeftColor: colors.primary` |
| `src/components/UpgradePrompt.tsx` | 491 | Primary CTA button background | `backgroundColor: colors.primary` |

#### Expected Result
All primary-colored elements update immediately when the user changes the accent color.

#### Actual Result
Elements using `colors.primary` inside `StyleSheet.create()` remain Violet even after changing the accent. Elements using inline `{ color: themeColors.primary }` correctly update.

#### Fix Required
For each occurrence: move the frozen `colors.primary` reference out of `StyleSheet.create()` and into an inline style object that reads from `themeColors` obtained via `useTheme()`.

Pattern:
```tsx
// BEFORE (frozen):
const styles = StyleSheet.create({
  fab: { backgroundColor: colors.primary },
});
// ...
<FAB style={styles.fab} />

// AFTER (reactive):
const styles = StyleSheet.create({
  fab: { /* no color here */ },
});
const { colors: themeColors } = useTheme();
// ...
<FAB style={[styles.fab, { backgroundColor: themeColors.primary }]} />
```

**Special case**: `ReportResultScreen.tsx` does not import or call `useTheme()` at all. It needs `useTheme()` added before the FAB style can be made reactive.

#### Priority within the fix
High-traffic screens (users see daily): `ExpenseFormScreen`, `CategoriesScreen`, `MileageFormScreen`, `IncomeListScreen`.
Moderate: `PayPeriodBillsScreen`, `PendingIncomeSection`, `PaycheckAllocationWizard`.
Low-traffic: `ReportResultScreen`, `PaywallScreen`, `UpgradePrompt`.

---

### Bug 2: VoiceRecorder and DictateButton use static colors.primary inline (not in StyleSheet)

**Severity**: Low (cosmetic, inline style so less severe than StyleSheet freeze)
**Component**: `src/components/VoiceRecorder.tsx`, `src/components/DictateButton.tsx`

#### Description
These components import `colors` statically from `@/theme` and use `colors.primary` in inline style props (not StyleSheet). While inline styles _can_ be reactive if they reference a live context value, here they reference the frozen static import. The recording indicator dot and listening dot will not update to a custom accent color.

Lines:
- `VoiceRecorder.tsx:457,459,485,494,523,595` — recording dot, text, and buttons use `colors.primary`
- `DictateButton.tsx:223,242,243` — listening dot and text use `colors.primary`

Fix: import `useTheme()` (or use the Paper `theme.colors.primary` which already reflects accent via `paperTheme` override) instead of the static `colors` import for these properties.

---

## Findings — No Bug (Cleared)

### Header Pattern Compliance
All screens correctly use `headerShown: false` for their own custom headers. The two exceptions in `RootNavigator.tsx` (lines 99 and 108) are for `OnboardingPrivacyPolicy` and `OnboardingTermsOfService` — these are intentionally using the native React Navigation header because they are full-screen modal pushes that simply need a back button and title. There are no custom header actions, so the iOS `UIBarButtonItem` pill issue does not apply. Cleared.

### ThemeContext Data Integrity
`ThemeContext.tsx` loads preferences in `loadPreferences()` with a try/catch that falls through gracefully — on AsyncStorage failure it logs a warning but still sets `isLoading: false`, so the app renders with defaults. No data loss possible.

`setAccentColor()` applies the change in memory first (synchronous) then persists async. If persistence fails, the in-session experience is preserved and a warning is logged. The accent reverts to default on next cold start, but no crash or data corruption occurs. This is acceptable behavior documented in the code.

`updateSettings()` in `SettingsService.ts` re-throws errors — this is intentional so callers can handle failures. The only caller for `accent_color` (`ThemeContext.setAccentColor`) already catches and warns, so the exception is handled.

`null` accent color survives `JSON.stringify`/`JSON.parse` round-trips correctly (verified by running the JSON round-trip in Node: `null` is serialized and deserialized as `null`, not dropped).

### AccentColorPicker Accessibility
The component has thorough accessibility coverage:
- Preset swatches: `accessibilityRole="radio"`, `accessibilityState.selected`, descriptive `accessibilityLabel`
- Swatch row: `accessibilityRole="radiogroup"`
- Custom color row: `accessibilityRole="button"`, dynamic label including current hex value
- Reset link: `accessibilityRole="button"`, `accessibilityLabel`
- Color wheel Modal: `accessibilityViewIsModal={true}`, backdrop has close label
- Hex input: `accessibilityLabel` + `accessibilityHint` with example
- Apply button: `accessibilityState.disabled` reflects enabled state
- Touch targets: `TOUCH_TARGET = 44` const used for swatch touchables; custom row has `minHeight: 44`; reset has `hitSlop` of 8pt on all sides

No accessibility issues found.

### FeatureCard / PageIndicator (Onboarding) — Cleared
Initially flagged as potentially frozen — confirmed NOT frozen. Both components call `useTheme()` and use the returned `colors` object inline. The `colors` variable here is the live context value, not the static import. Cleared.

### SettingsScreen color picker integration
`SettingsScreen` uses `useTheme()` throughout and passes `accentColor` and `setAccentColor` from context to `AccentColorPicker`. No frozen colors in the picker entry point. Cleared.

### FinanceKitService test suite skipped
`describe.skip` at line 63 is intentional — the test requires a native iOS bridge (`expo-apple-authentication` patterns) not available in Jest. Pre-existing. Not a regression.

### Test worker leak
Worker exit warning is from `AudioService.ts` using `setTimeout` in async helper methods. Pre-existing, reproducible before Wave 8.5. Not a new regression.

---

## Accessibility Summary

| Area | Status | Notes |
|------|--------|-------|
| AccentColorPicker | PASS | Full radio group, labels, hints, disabled states |
| Onboarding components | PASS | accessibilityRole, labels, touch targets |
| Touch targets (44pt) | PASS | AccentColorPicker enforces 44pt constant |
| Header patterns | PASS | Custom headers on all action screens |
| Modal accessibility | PASS | accessibilityViewIsModal on color sheet |

---

## Go / No-Go Recommendation

**GO for Wave 9 with one condition: Bug 1 fix should be tracked.**

The automated test suite is clean (zero failures, one pre-existing intentional skip). The Wave 8.5 accent color feature works correctly for the majority of elements. The `ThemeContext` architecture is sound.

However, Bug 1 (frozen StyleSheet colors) represents an **incomplete Wave 8.5 implementation** — 10 elements across 9 files still use the Violet default regardless of the user's accent preference. Users who change their accent color will notice these inconsistencies because they are on high-traffic screens (ExpenseForm, Categories, MileageForm).

**Recommended action before Wave 9 closes:**
1. Create a `fix/wave-85-frozen-stylesheet-colors` branch
2. Fix all 10 frozen `colors.primary` references listed in Bug 1
3. Fix `VoiceRecorder` and `DictateButton` inline frozen refs (Bug 2)
4. Submit for code review then merge to dev
5. This QA report serves as the test plan — verify each element updates correctly after accent change

If the product decision is to accept the cosmetic inconsistency for now and deliver Wave 9 features instead, these can be tracked as tech debt, but users who find the color picker will notice immediately.

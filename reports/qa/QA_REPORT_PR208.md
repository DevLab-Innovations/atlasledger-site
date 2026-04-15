# QA Report: PR #208 — feature/onboarding-polish

**Date**: 2026-03-10
**Tester**: qa-tester
**Branch**: dev (post-merge, commit 933dd85)
**PR**: https://github.com/buzybee83/ExpenseTracker/pull/208

---

## Results Overview

| Metric | Count |
|--------|-------|
| Total Scenarios Tested | 22 |
| Passed | 20 |
| Failed | 2 |
| Blocked | 0 |

**Automated test suite**: 3,494 passed / 28 failed (pre-existing regression introduced by parallel Wave 10 / 8.5 work merged alongside PR #208 — see Bug 2 below)

---

## Recommendation

**Needs fixes before promotion to main.**

Two issues require resolution:

1. **Bug 1 (High)**: `PRIVACY_SLIDE_INDEX` hardcoded to `2` but privacy slide is at index `3` in the 7-slide `OnboardingScreen`. Legal footnote (Privacy Policy / Terms of Service links) appears on the wrong slide.
2. **Bug 2 (Medium)**: 28 test failures in `Wave96EGestureExtensions.test.tsx` — local ThemeContext mock missing `chartColors`. Triggered by the dev branch merge combining PR #207's test file with parallel Wave 10/8.5 work. Not directly caused by PR #208 code, but it is a regression that exists on `dev` post-merge and must be fixed before `main` promotion.

---

## Bug Reports

---

### Bug 1: Legal Footnote Shows on Wrong Slide

**Severity**: High
**Component**: `src/components/onboarding/OnboardingSlideshow.tsx` + `src/screens/OnboardingScreen.tsx`

#### Steps to Reproduce
1. Launch the app for the first time (onboarding not completed)
2. Reach slide 3 of 7 ("Made for You" — Color Themes, iCloud Sync, Tax-Ready Export)
3. Observe the bottom navigation area

#### Expected Result
The Privacy Policy and Terms of Service links should appear **only** on slide 4 of 7 ("Your Data Stays Private"), since that is the privacy-focused slide.

#### Actual Result
The Privacy Policy and Terms of Service links appear on **slide 3 of 7** ("Made for You"), which has no privacy context. The links do NOT appear on slide 4 where they belong.

#### Root Cause
`PRIVACY_SLIDE_INDEX = 2` is hardcoded in `OnboardingSlideshow.tsx` line 28. At the time this constant was written, the onboarding had fewer slides. `OnboardingScreen.tsx` now passes 7 slides:

```
[0] Slide1  — Track Expenses Effortlessly
[1] Slide2  — Capture Expenses Instantly
[2] Slide2b — Made for You          ← PRIVACY_SLIDE_INDEX points here (WRONG)
[3] Slide3  — Your Data Stays Private  ← Correct privacy slide (MISSING footnote)
[4] Slide3b — No Account, No Problem
[5] Slide4  — Ready to Start Tracking?
[6] Slide5  — You're ready to go!
```

The constant was not updated when the additional slides (Slide2b, Slide3b) were split out.

#### Evidence
- `src/components/onboarding/OnboardingSlideshow.tsx` line 28: `const PRIVACY_SLIDE_INDEX = 2;`
- `src/screens/OnboardingScreen.tsx` line 247: `const slides = [Slide1, Slide2, Slide2b, Slide3, Slide3b, Slide4, Slide5];`
- "Your Data Stays Private" title appears in `Slide3` at array index 3

#### Fix Required
Change `PRIVACY_SLIDE_INDEX = 2` to `PRIVACY_SLIDE_INDEX = 3` in `OnboardingSlideshow.tsx`.

Update the `OnboardingSlideshow.test.tsx` suite: the 4-slide test harness uses slides indexed 0–3, so the legal footnote tests that navigate to "Go to page 3" (index 2) are testing the correct index for that 4-slide fixture. The unit tests themselves are fine. The integration test in `OnboardingScreen.test.tsx` should be extended to navigate to page 4 (index 3) and assert the footnote appears there.

---

### Bug 2: Wave96EGestureExtensions Tests Fail on dev — chartColors Missing from Local Mock

**Severity**: Medium
**Component**: `src/__tests__/components/charts/Wave96EGestureExtensions.test.tsx`

#### Steps to Reproduce
```bash
git checkout dev
npm test -- --testPathPattern="Wave96EGestureExtensions" --no-coverage
```

#### Expected Result
35 tests pass (as they did on commit `9c94996` before the dev merge).

#### Actual Result
28 tests fail with:

```
TypeError: Cannot read properties of undefined (reading '0')
  at src/theme/chartTheme.ts:56:34
  → colors.chartColors[0]
```

#### Root Cause
`Wave96EGestureExtensions.test.tsx` declares a local `jest.mock('@/contexts/ThemeContext')` at lines 63–81 that overrides the global setup in `jest.setup.js`. The local mock's `colors` object does not include `chartColors`. The global `jest.setup.js` mock (line 691) does include `chartColors`, but local mocks shadow global ones per Jest module-registry rules.

`chartTheme.ts` line 56 calls `colors.chartColors[0]`, which is `undefined[0]` when the local mock is active, causing the TypeError.

This regression surfaced because:
- PR #207 introduced `Wave96EGestureExtensions.test.tsx` (with the incomplete local mock)
- PR #207 also introduced the test file's tested components in isolation, where `chartTheme.ts` did not yet reference `chartColors[0]` in the path exercised by those tests
- Parallel dev work (Wave 9.5 charts, Wave 8.5 color customization) merged into dev before PR #208 and changed `chartTheme.ts` / `SpendingTrendChart.tsx` in ways that now exercise the `chartColors[0]` code path during those tests
- When all branches converged in the dev merge of PR #208, the incompatible mock was exposed

Confirmed: Wave96E tests pass at commit `9c94996` (pre-PR#208), fail at commit `933dd85` (post-PR#208 merge). The root deficiency is in the Wave96E test file, not in PR #208 code.

#### Fix Required
Add `chartColors` to the local ThemeContext mock in `Wave96EGestureExtensions.test.tsx`:

```ts
jest.mock('@/contexts/ThemeContext', () => ({
  useTheme: () => ({
    colors: {
      // ... existing keys ...
      chartColors: ['#4F46E5', '#F97316', '#0891B2'],
    },
    isDark: false,
  }),
}));
```

---

## Passed Scenarios

### ExpenseMockup.tsx

| # | Scenario | Result |
|---|----------|--------|
| 1 | Component renders 3 expense rows (coffee, cart-outline, car-outline icons) | PASS |
| 2 | Each row has correct label / category / amount text | PASS |
| 3 | `AccessibilityInfo.isReduceMotionEnabled()` called on mount | PASS |
| 4 | Reduce-motion: `translateY` initializes to 0, opacity uses `withTiming duration:0` | PASS |
| 5 | Each row accessible with `accessibilityLabel` = "{label}, {category}, {amount}" | PASS |
| 6 | Stagger delays: 0ms, 180ms, 360ms for rows 0–2 | PASS |
| 7 | Icon pill uses `colors.primaryBg` background, `colors.primary` icon tint | PASS |
| 8 | Dead code: `reduceMotionRef` is set but never read (cosmetic-only, Low severity) | NOTE |

### OnboardingSlideshow.tsx

| # | Scenario | Result |
|---|----------|--------|
| 9 | Skip button visible on non-last slides, hidden on last | PASS |
| 10 | Next button → "Get Started" on final slide | PASS |
| 11 | Back button absent on slide 1, present on slide 2+ | PASS |
| 12 | `Gesture.Race` used (not `Simultaneous`) — only one fling direction wins | PASS |
| 13 | Transition: `slideTranslateX` set to ±40 then eases to 0 over 260ms cubic | PASS |
| 14 | Slide animation: opacity 0→1 over 220ms | PASS |
| 15 | Accessibility announcement fired on each slide change | PASS |
| 16 | Privacy footnote on slide index 2 of 4 (unit test fixture) | PASS — but see Bug 1 for real-app index |
| 17 | Legal footnote hidden when no handlers provided | PASS |
| 18 | `goToSlide` direction detection uses `currentSlideShared.value` (worklet-safe) | PASS |

### AppleSignInButton.tsx

| # | Scenario | Result |
|---|----------|--------|
| 19 | `useCallback` declared before conditional returns (Rules of Hooks compliant) | PASS |
| 20 | Returns `null` on Android/web (`Platform.OS !== 'ios'` guard after hook) | PASS |
| 21 | `NativeModules.ExpoAppleAuthentication` check evaluated at module load | PASS |
| 22 | Fallback Pressable renders when native module absent; `accessibilityRole="button"` | PASS |
| 23 | Fallback press calls `signInAsync` — throws, caught by `onError` (graceful degradation) | PASS — documented behavior |

### StorageOnboardingScreen.tsx

| # | Scenario | Result |
|---|----------|--------|
| 24 | `cardGroup.gap = 12` separates the two option cards | PASS |
| 25 | Default selection is `local_only` | PASS |
| 26 | Cloud card tap when not signed in → shows inline Apple sign-in UI | PASS |
| 27 | Sign-in success → `enableSync()` → `finishOnboarding()` even if enableSync throws | PASS |
| 28 | "Decide later" → `setStoragePreference('local_only')` → navigate | PASS |
| 29 | `accessibilityRole="radiogroup"` on card container; `role="radio"` on each card | PASS |
| 30 | `STORAGE_ONBOARDING_KEY` written before `navigation.replace('MainTabs')` | PASS |
| 31 | `isCommitting` guard prevents double-submit on Continue button | PASS |

---

## Notes for Developer

### Low Severity / Cosmetic

- **`reduceMotionRef` dead code** in `ExpenseMockup.tsx` (lines 82, 87): The ref is assigned but never consumed. Only `reduceMotion` state is used. Can be cleaned up in a follow-up.

- **`goingForward` direction calculation** in `OnboardingSlideshow.tsx` line 85 reads `currentSlideShared.value` before `setCurrentSlide` updates it. If the user navigates faster than a React render cycle, the slide-in direction could be incorrect (wrong ±40 enter position). The navigation itself is unaffected; only the animation direction could be cosmetically wrong. Extremely unlikely in practice given 260ms transition.

---

## Automated Test Suite Summary

```
Test Suites: 1 failed, 1 skipped, 172 passed, 173 of 174 total
Tests:       28 failed, 55 skipped, 3494 passed, 3577 total
```

The 28 failures are exclusively in `Wave96EGestureExtensions.test.tsx` (Bug 2 above). All PR #208 tests pass (OnboardingSlideshow: 19/19, OnboardingScreen: 5/5).

The 55 skipped tests are pre-existing skips unrelated to this PR.

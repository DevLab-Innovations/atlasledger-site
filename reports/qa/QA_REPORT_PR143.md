# QA Report: PR #143 — feature/dashboard-compactness-optimization

**Date**: 2026-02-24
**Tester**: qa-tester agent
**Branch Tested**: dev (HEAD: 6b4c16e)
**Feature Branch**: feature/dashboard-compactness-optimization (HEAD: daf04f8)
**Build**: Atlas Ledger v1.3.0

---

## Results Overview

- Total Scenarios: 18
- Passed: 0
- **BLOCKED: 18** — Feature branch was never merged into dev

### Recommendation

**[ ] BLOCKED — Feature branch NOT merged to dev. Cannot perform QA validation.**

---

## CRITICAL FINDING: Incomplete Merge

### The PR #143 feature branch has NOT been merged into the `dev` branch.

All 12 commits from `feature/dashboard-compactness-optimization` are absent from `dev`:

```
daf04f8 fix: remove debug console.logs and fix misplaced const in SettingsScreen
42c1502 feat: reduce CategoryDonutChart size for compact dashboard
c29ace1 feat: add subtext lines to half-width summary widgets for consistent height
183874e fix: prevent UNIQUE constraint error on concurrent widget initialization
1a7c360 fix: prevent nested SQLite transactions on tier change
d3c7218 fix: detect and handle tier changes that occur before Dashboard mount
3090ebd chore: remove debug logging from widget grid and context
7cf636e fix: clear widget_config table on factory reset and wipe data
d167356 fix: remove incorrect free tier gridSpan override in useVisibleWidgets
e8b0063 fix: clear dashboard guide flag on factory reset and wipe data
d6ebf59 fix: move WidgetProvider to app level to support SettingsScreen access
dbdc043 feat: add 'Reset Dashboard Layout' developer option for testing tier-specific layouts
```

Verification command:

```bash
git log --oneline feature/dashboard-compactness-optimization ^dev
# Returns 12 commits (should return 0 if merged)
```

The `dev` branch HEAD is at `6b4c16e` — a commit from the **previous wave** (tier-specific widget layouts), which predates all PR #143 work.

---

## Automated Test Results

**Test Suite**: `npm test -- --watchAll=false`

```
Test Suites: 1 failed, 1 skipped, 102 passed, 103 of 104 total
Tests:       1 failed, 55 skipped, 1944 passed, 2000 total
```

### Failing Test (Pre-existing, caused by incomplete merge)

**File**: `src/__tests__/screens/SettingsScreen.test.tsx:273`

**Test**: `SettingsScreen > About Section > should display app version from package.json`

**Root Cause**: The feature branch updated this test to match version `1.3.0` (current `package.json`) and added a missing `WidgetContext` mock. Since the feature branch was not merged, `dev` still has the old test asserting version `1.2.0` but `package.json` is at `1.3.0`.

```
Expected: /1\.2\.0/
Actual rendered: "1.3.0"
```

The feature branch fix for this test (in commit `daf04f8`) is:
- Updates regex to `/1\.3\.0/`
- Adds missing `jest.mock('@/contexts/WidgetContext', ...)` to prevent test breakage from SettingsScreen now calling `useWidgets()`

---

## Static Code Analysis: Defects Found on `dev` (vs. Feature Branch Spec)

The following bugs exist on `dev` because the PR was not merged. These would all be found by manual UI testing.

---

### Bug #1 — CRITICAL: `CategoryDonutChart.tsx` Still Uses Old 280×280px Dimensions

**Severity**: High
**Component**: `src/components/charts/CategoryDonutChart.tsx`

**Steps to Reproduce**:
1. Open the dev branch
2. Navigate to Dashboard on any premium/pro tier account
3. Observe the Category Spending widget

**Expected Result**: Chart renders at 240×240px with outerRadius=103, innerRadius=68, skeleton height 240px, legend marginTop=`spacing.sm`

**Actual Result** (on `dev` HEAD):
```
Line 238: <SkeletonCard height={280} />   // should be 240
Line 254: const chartSize = 280;           // should be 240
Line 257: const outerRadius = 120;         // should be 103
Line 258: const innerRadius = 80;          // should be 68
Line 455: width: 280, height: 280          // chartContainer — should be 240, 240
Line 463: width: 280, height: 280          // chartLayer — should be 240, 240
Line 494: marginTop: spacing.md,           // legend — should be spacing.sm
```

**Impact**: The dashboard compactness goal (-40px vertical space savings) is completely absent. The chart is larger than specified, shadow layers do not fit within the container bounds, and the skeleton placeholder mismatches the actual chart height.

---

### Bug #2 — HIGH: `ExpenseSummaryWidget.tsx` Missing "N expenses" Subtext in Half-Width Mode

**Severity**: High
**Component**: `src/widgets/cards/ExpenseSummaryWidget.tsx`

**Steps to Reproduce**:
1. On a Premium/Pro account, set Expense Summary widget to half-width
2. Pair with Income Summary widget on the same row

**Expected Result**: Expense card shows a subtext line `"N expense(s)"` below the amount, giving equal height to both cards.

**Actual Result** (on `dev`): Only the amount text is rendered in half-width mode. No subtext. The `countSubtext` style does not exist. The Income card (which also lacks the subtext on dev) will differ in height from the Expense card when income is zero vs. non-zero.

**Missing code** (from feature branch commit `c29ace1`):
```tsx
<Text
  variant="bodySmall"
  style={[styles.countSubtext, { color: themeColors.textSecondary }]}
>
  {loading ? '' : `${data.count} ${data.count === 1 ? 'expense' : 'expenses'}`}
</Text>
```

---

### Bug #3 — HIGH: `IncomeSummaryWidget.tsx` Missing "net +/-$X.XX" Subtext in Half-Width Mode

**Severity**: High
**Component**: `src/widgets/cards/IncomeSummaryWidget.tsx`

**Steps to Reproduce**:
1. On a Premium/Pro account, set Income Summary widget to half-width
2. Pair with Expense Summary widget on the same row

**Expected Result**: Income card shows `"net +$X.XX"` or `"net -$X.XX"` in success/warning color below the amount.

**Actual Result** (on `dev`): Only the amount text is rendered in half-width mode. No net subtext. The `subtext` style does not exist. Half-width paired cards have mismatched heights.

**Missing code** (from feature branch commit `c29ace1`):
```tsx
<Text
  variant="bodySmall"
  style={[styles.subtext, { color: loading ? 'transparent' : netColor }]}
>
  {loading ? ' ' : `net ${data.net >= 0 ? '+' : ''}${formatCurrency(data.net)}`}
</Text>
```

---

### Bug #4 — CRITICAL: Race Condition in `WidgetContext.tsx` — `isInitializingRef` Guard Missing

**Severity**: Critical
**Component**: `src/contexts/WidgetContext.tsx`

**Steps to Reproduce**:
1. Install fresh app (or clear data)
2. Launch the app
3. Observe console output

**Expected Result**: Dashboard initializes cleanly. No UNIQUE constraint errors.

**Actual Result** (on `dev`): The `isInitializingRef` guard is completely absent from `WidgetContext.tsx`. On `dev`, the init function has no re-entrancy protection:

```tsx
// MISSING on dev — this guard is not present:
if (isInitializingRef.current) return;
isInitializingRef.current = true;
```

Additionally, the `finally` block that resets the flag (`isInitializingRef.current = false`) is absent. The tier-change `useEffect` also lacks the `isInitializingRef.current` check in its guard condition.

**Consequence**: When `userTier` changes during initialization (a common race on first app launch with subscription restore), two concurrent `init()` calls execute simultaneously, both calling `syncWithRegistry()`, which both attempt `INSERT INTO widget_config` for the same widget types, producing a SQLite `UNIQUE constraint failed: widget_config.widget_type` crash.

---

### Bug #5 — HIGH: `WidgetContext.tsx` Missing `WIDGET_TIER_KEY` Persistence Logic

**Severity**: High
**Component**: `src/contexts/WidgetContext.tsx`

**Steps to Reproduce**:
1. Upgrade from Free to Premium
2. Kill and relaunch the app
3. Observe dashboard layout

**Expected Result**: Dashboard detects the stored tier differs from current tier and resets to tier-specific defaults.

**Actual Result** (on `dev`): The entire `WIDGET_TIER_KEY` persistence system is absent. The `dev` version of `init()` only calls `syncWithRegistry()` unconditionally with no stored-tier comparison. This means:
- Tier changes that happen while the app is closed are never detected at startup
- The tier key is never written to AsyncStorage
- On tier upgrade/downgrade between sessions, the dashboard silently shows the wrong layout

**Missing logic** (from commit `d3c7218`):
```tsx
const storedTier = await AsyncStorage.getItem(WIDGET_TIER_KEY);
const currentTierForDefaults = userTier === 'team' ? 'pro' : userTier;

if (storedTier && storedTier !== userTier) {
  // Tier changed while app was closed
  await repositoryRef.current.resetToDefaults(currentTierForDefaults);
  await AsyncStorage.setItem(WIDGET_TIER_KEY, userTier);
  await AsyncStorage.removeItem(GUIDE_SHOWN_KEY);
} else if (!storedTier) {
  await repositoryRef.current.syncWithRegistry(currentTierForDefaults);
  await AsyncStorage.setItem(WIDGET_TIER_KEY, userTier);
} else {
  await repositoryRef.current.syncWithRegistry(currentTierForDefaults);
}
```

---

### Bug #6 — HIGH: `WidgetProvider` Still Wraps Only `WidgetDashboard`, Not App-Level

**Severity**: High
**Component**: `App.tsx`, `src/components/WidgetDashboard.tsx`

**Steps to Reproduce**:
1. Open Settings screen
2. Navigate to any section that calls `resetToDefaults` (e.g., Factory Reset)

**Expected Result**: `resetToDefaults` call in `SettingsScreen` operates correctly because `WidgetProvider` is at app level.

**Actual Result** (on `dev`): `WidgetProvider` still wraps only `WidgetDashboard` (inside the dashboard screen), not at the App root. Meanwhile, `SettingsScreen` on the feature branch imports `useWidgets()` and calls `resetDashboardToDefaults` — but on `dev`, `SettingsScreen` does not have the `useWidgets()` call, so this particular crash isn't triggered yet. However, once that SettingsScreen change is merged without also moving `WidgetProvider` to App level, calling `useWidgets()` from `SettingsScreen` will throw "useWidgets must be used within a WidgetProvider".

**Missing in `App.tsx`** (from commit `d6ebf59`):
```tsx
import { WidgetProvider } from './src/contexts/WidgetContext';
// ...
<WidgetProvider>
  <NavigationContainer theme={navigationTheme}>
    <StatusBar style={isDark ? 'light' : 'dark'} />
    <RootNavigator />
  </NavigationContainer>
</WidgetProvider>
```

**Missing in `WidgetDashboard.tsx`** — removal of local `<WidgetProvider>` wrapper (which would cause a duplicate provider nesting if App.tsx change is applied but WidgetDashboard.tsx is not cleaned up).

---

### Bug #7 — MEDIUM: `DatabaseService.ts` Missing `widget_config` in Factory Reset Tables

**Severity**: Medium
**Component**: `src/services/DatabaseService.ts`

**Steps to Reproduce**:
1. Customize dashboard widget layout
2. Go to Settings → Factory Reset (or Wipe Data)
3. Re-open the dashboard

**Expected Result**: Dashboard resets to tier-specific defaults after factory reset.

**Actual Result** (on `dev`): The `widget_config` table is not in either the "wipe data" or "factory reset" table lists. Customized widget configurations persist through resets, meaning a user who factory resets still sees their old customized dashboard layout instead of the tier-appropriate defaults.

**Missing** (from commit `7cf636e`):
```ts
'widget_config', // Reset dashboard layout to tier-specific defaults
```
in both the `wipeAllData()` and `factoryReset()` table arrays.

---

### Bug #8 — HIGH: `useVisibleWidgets()` Has Runtime gridSpan Override That Should Be Removed

**Severity**: High
**Component**: `src/contexts/WidgetContext.tsx` (lines 427-433)

**Steps to Reproduce**:
1. On a Premium tier, note that expense-summary defaults to half-width (gridSpan=1 from DB)
2. Observe if widget appears full-width due to runtime override

**Expected Result** (per feature spec): gridSpan is controlled entirely by tier-specific DB initialization. No runtime override needed.

**Actual Result** (on `dev`): Lines 427-433 contain a runtime map that forcibly overrides `gridSpan` for `expense-summary` when `gridSpan === undefined`:
```tsx
return filtered.map((w) => {
  if (w.widgetType === 'expense-summary' && w.gridSpan === undefined) {
    const tierDefault = userTier === 'free' ? 2 : 1;
    return { ...w, gridSpan: tierDefault as GridSpan };
  }
  return w;
});
```
This creates a conflict: once `widget_config` stores the tier-specific `gridSpan` from initialization, this override never fires (correct). But for users who installed before tier-specific initialization was added, `gridSpan` in DB might be `null`, and this override kicks in — potentially incorrectly overriding what the user set. The feature branch removes this code entirely (commit `d167356`) since tier-specific initialization now handles it correctly at the DB level.

---

## Summary: Files Changed in PR Not Present on `dev`

| File | Change Description | Status on dev |
|------|-------------------|----|
| `src/components/charts/CategoryDonutChart.tsx` | Resize 280→240px | OLD dimensions |
| `src/widgets/cards/ExpenseSummaryWidget.tsx` | Add "N expenses" subtext | Missing |
| `src/widgets/cards/IncomeSummaryWidget.tsx` | Add "net +/-$X.XX" subtext | Missing |
| `src/contexts/WidgetContext.tsx` | isInitializingRef guard, WIDGET_TIER_KEY persistence, remove gridSpan override | Substantially incomplete |
| `src/repositories/WidgetConfigRepository.ts` | INSERT OR IGNORE for idempotency | Applied |
| `App.tsx` | Move WidgetProvider to app level | NOT applied |
| `src/components/WidgetDashboard.tsx` | Remove local WidgetProvider wrapper | NOT applied |
| `src/services/DatabaseService.ts` | Add widget_config to reset tables | NOT applied |
| `src/screens/SettingsScreen.tsx` | useWidgets, WIDGET_TIER_KEY clear on reset, GUIDE_SHOWN_KEY clear on wipe | NOT applied |
| `src/__tests__/screens/SettingsScreen.test.tsx` | Version bump 1.2.0→1.3.0, add WidgetContext mock | NOT applied |

Note: `WidgetConfigRepository.ts` `INSERT OR IGNORE` changes DO appear to be on dev (verified at lines 267, 338 of the current file). This was from commit `183874e` which was partially cherry-picked or independently committed.

---

## Action Required

The `feature/dashboard-compactness-optimization` branch must be merged into `dev` before QA testing can proceed. Once merged:

1. Re-run automated tests — the `SettingsScreen.test.tsx` version mismatch failure should be resolved
2. Verify no new test failures were introduced
3. Re-run this QA checklist against the updated `dev` branch

**QA cannot approve promotion to `main` until all feature branch commits are on `dev`.**

---

## Next Steps

1. **Merge PR #143 to dev**: `git merge feature/dashboard-compactness-optimization` (or reopen/re-merge the PR)
2. **Re-run tests**: Confirm the 1 existing failure is resolved and no new failures appear
3. **Re-trigger QA**: Notify qa-tester to re-run this validation

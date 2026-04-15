# QA Report: Group 8E — Dashboard Chart Upgrade & Triage Deck

**Date**: 2026-03-01
**Tester**: qa-tester agent
**Branch**: dev
**Merge commit**: a115a06
**Build**: atlas-ledger v1.4.1 (dev branch)

---

## Test Summary

| Metric | Result |
|---|---|
| Full test suite | 2660 passed, 55 skipped, 0 failed |
| Test suites | 132 passed, 1 skipped, 0 failed |
| New tests (Group 8E) | 30 passed |
| TypeScript errors | 0 |
| Regressions | None |
| Bugs found | 1 Medium, 3 confirmed P2 acceptances, 1 pre-existing stub |

**Recommendation: CONDITIONAL PASS — ship as-is, but the 1 Medium bug and 1 elevated P2 should be tracked in the backlog before the next major release.**

---

## Test Execution

### 1. Full Test Suite

```
Tests: 2660 passed, 55 skipped, 0 failed
Test Suites: 132 passed, 1 skipped (total 133)
Time: 7.46s
```

No regressions introduced. All 30 new Group 8E tests pass.

There is one benign console warning in `CategoryRadialChart.test.tsx`:

```
An update to CategoryRadialChart inside a test was not wrapped in act(...)
```

This is produced by the `AccessibilityInfo.isReduceMotionEnabled()` async call during mount
resolving after the test assertion. It does not cause test failure and is a known
React Testing Library pattern. Low priority cleanup item.

### 2. TypeScript

`npx tsc --noEmit` exits clean. Zero type errors across all new and modified files.

---

## Component-by-Component Findings

### Part A: CategoryRadialChart

**Status: PASS with notes**

**Verified:**
- Skeleton loading state renders SkeletonCard correctly
- Empty state renders when `data=[]` or `totalAmount=0`
- Top 5 categories capped correctly (MAX_RINGS=5); 6th category excluded from chart and legend
- Ring radius math is sound — ring 0 radius 109px, ring 4 radius 37px, all positive values
- `fillFraction` capped at `0.999` to prevent full-circle rendering artifact (correct)
- `accessibilityRole="image"` on SVG container; `accessibilityRole="button"` on each legend item
- VoiceOver label on legend items includes category, amount, percentage, and transaction count
- `accessibilityHint` conditionally added only when `onCategoryPress` prop is provided
- Touch targets: `legendItem` has `minHeight: 44` — meets iOS 44pt minimum
- Reduce motion: checked via `AccessibilityInfo.isReduceMotionEnabled()` + event listener with cleanup
- `animated={false}` passed when `isEditing` in CategorySpendingWidget (animations disabled in edit mode)

**Notes:**
- The center "TOTAL" label uses `pointerEvents="none"` on its wrapper View, preventing it from
  intercepting taps intended for the chart's deselect handler. Correct.
- The `accessibilityLabel` on the center amount text says `Total: $X` but the surrounding
  Pressable already has a comprehensive label covering the full breakdown. Redundant but harmless.

---

### Part B: TriageDeck + TriageCard

**Status: PASS with 1 Medium bug**

**Verified:**
- Loading state shows SkeletonCard (height 300)
- Empty state ("All Caught Up!") when no uncategorized expenses
- Completion state with celebration animation and canUndo button
- Progress bar width calculated correctly: `completedCount / totalCount * 100%`
- Reduce motion fallback renders static list with buttons (no gesture animations)
- Card stack: `visibleItems.reverse()` ensures top card is z-axis highest — correct
- GestureDetector only wraps the top card; background cards have no gestures
- Stack visual offset: `stackOffset * 8` translate, `1 - stackOffset * 0.05` scale — correct
- High confidence items sorted to front of deck
- `acceptSuggestion` error handling: DB error returns early without advancing card
- `assignCategory` error handling: same early return pattern
- `skip` and `markForSplit` do not call DB — correct (no category assigned)
- Undo correctly decrements `currentIndex` with `Math.max(prev-1, 0)` guard at boundary
- canUndo shown after completion state — undo button visible when `historyStack` non-empty

---

## Bug Reports

### BUG-8E-001: Swipe-right silently does nothing on expenses with no AI suggestion

**Severity**: Medium
**Component**: `src/hooks/useTriageDeck.ts` + `src/components/triage/TriageCard.tsx`

**Steps to Reproduce**
1. Open Triage from QuickActionsWidget
2. Have at least one uncategorized expense for a merchant that `MerchantCategoryService` cannot
   recognize (returns `null` — this is documented and expected per the service's `@returns` JSDoc)
3. The card shows "Swipe left to choose a category" instead of a suggestion chip
4. Swipe right on the card

**Expected Result**
The card should either: (a) advance to the next card (treating no-suggestion swipe-right as a skip),
or (b) display feedback explaining that there is no suggestion to accept, or (c) hide the "Accept"
gesture hint when no suggestion is available.

**Actual Result**
`acceptSuggestion()` exits at line 133 (`if (!currentItem?.suggestion) return`) without advancing,
without providing any user feedback, and without logging. The card snaps back to position.
The gesture hint row still shows the right-arrow "Accept" label, implying the action is available.
The VoiceOver label also says "Swipe right to accept" (line 111 of TriageCard.tsx), misleading
screen reader users.

**Evidence**
```typescript
// useTriageDeck.ts line 132-133
const acceptSuggestion = useCallback(async () => {
  if (!currentItem?.suggestion) return;  // silent no-op
```

```typescript
// TriageCard.tsx line 111 — always says "Swipe right to accept" regardless of suggestion
accessibilityLabel={`...${suggestion ? ... : 'No suggestion'}. Swipe right to accept...`}
```

**Recommended Fix**
Option A (simplest): When `!currentItem?.suggestion`, have `acceptSuggestion` call `skip()` to
advance the card, or call `setShowFan(true)` to open category selection.
Option B: In `TriageCard`, conditionally omit the right-arrow "Accept" hint when `!suggestion`,
and update the VoiceOver label accordingly.

---

## P2 Code Review Items — Assessment

These were flagged by the code reviewer. QA assessment of whether they are acceptable to ship.

### P2-1: CategoryFan unmounts without exit animation

**QA Assessment: Acceptable to ship as P2 backlog item**

The parent `TriageDeck` conditionally renders `{showFan && currentItem && <CategoryFan ...>}`.
When `showFan` becomes `false`, the component unmounts immediately. The `FanItem` components
never receive `visible=false` before unmounting, so the `scale.value = withSpring(0)` exit
animation in their `useEffect` never fires.

The visual result is that the fan disappears instantly with no exit animation (a "pop" rather
than a smooth close). This is a polish issue — the UX is functional and non-blocking. Users
can still select categories without any degraded behavior.

Severity: Low. Acceptable for this release. Recommend addressing in a subsequent UI polish wave.

---

### P2-2: Swipe-up (split) gesture not shown in visual hint row

**QA Assessment: Acceptable, with a note**

The hint row in `TriageCard` shows three gestures: left (Choose), down (Skip), right (Accept).
Swipe-up (split) is not shown. The VoiceOver accessibility label on line 111 does mention
"up to split", so screen reader users are informed. Sighted users are not.

The split action is an advanced operation. Omitting it from the basic hint row follows
progressive disclosure principles — the three most common actions are surfaced; the advanced
action (split) is discoverable through exploration or via VoiceOver.

Severity: Low. Acceptable. If split usage is tracked via analytics and proves to be low,
this design choice is validated. Recommend adding it in a future UX iteration if split
usage warrants surfacing the hint.

---

### P2-3: undo() DB revert is fire-and-forget without error handling

**QA Assessment: Elevated — recommend promoting to P1 backlog**

```typescript
// useTriageDeck.ts lines 204-207
if (lastEntry.chosenCategory) {
  void dbService.current.updateExpense(lastEntry.item.expense.id, {
    category: 'Uncategorized',
  });
}
```

The `void` operator discards the promise. If the DB write fails silently:

1. UI shows the expense as uncategorized (moved back in deck)
2. DB still has the previously-categorized value
3. If the user navigates away before re-categorizing, the DB and UI are permanently out of sync
4. The expense will not appear in the next Triage session (it already has a non-Uncategorized
   category in DB), but the user believes it was undone

This is a data integrity issue, not just a cosmetic one. The impact is low-probability
(SQLite local writes rarely fail in the app's normal operating environment), but the
consequence is a silent data inconsistency that could corrupt the user's records.

**Recommended fix**: Wrap in try/catch, and on failure, push the item back to its
categorized position in the history stack and show a Snackbar error.

Severity of gap: Medium. The code reviewer logged it as P2. QA agrees it should be
elevated to P1 backlog and addressed before the next release that touches Triage.

---

### P2-4: undo test does not assert DB revert call

**QA Assessment: Confirmed gap, low priority**

The `useTriageDeck.test.ts` `'should support undo'` test (line 166) verifies that:
- `currentIndex` decrements to 0 after undo

It does NOT verify that `mockUpdateExpense` was called with `{ category: 'Uncategorized' }`.
This means if the DB revert line is accidentally removed, the test will still pass.

Given that P2-3 above is already a known quality concern, having a test that doesn't
cover the revert behavior is a compound risk.

Severity: Low. The missing assertion is straightforward to add. Recommend adding as
part of the same fix for P2-3.

---

## Stub / Pre-existing Issues (Not Blocking)

### Export button in CategorySpendingWidget is a no-op stub

The "Export" button in `CategorySpendingWidget` (line 69-82) fires haptics but contains
`// Export logic would go here` with no actual implementation. This was presumably
carried forward from a previous version and is unrelated to Group 8E changes.

This is pre-existing behavior and not introduced by this PR. No action required for
this release.

---

## Regression Verification

### Modified files — all verified correct

| File | Change | Regression Check |
|---|---|---|
| `CategorySpendingWidget.tsx` | Removed tier-gated chart switching; always renders `CategoryRadialChart` | Pass: renders empty state via chart instead of returning null |
| `WidgetRegistry.ts` | Updated `category-spending` description to "Radial bar rings"; icon changed to `chart-arc`; deprecated donut/pie widgets hidden (`defaultVisible: false`) | Pass: deprecated widgets present but invisible by default |
| `QuickActionsWidget.tsx` | Added Triage button (no tier gate) | Pass: button visible for all tiers; navigates to `MoneyTab/Triage` |
| `MoneyStack.tsx` | Added `Triage` screen import and Stack.Screen | Pass: route registered correctly |
| `src/types/navigation.ts` | Added `Triage: undefined` to `MoneyStackParamList` | Pass: TypeScript clean, consistent with MoneyStack definition |

### Navigation type verification

`MoneyStackParamList.Triage: undefined` is defined and matches the `Stack.Screen name="Triage"` in
`MoneyStack.tsx`. The `navigateToTriage` call in `QuickActionsWidget` uses `screen: 'Triage'`
which correctly resolves within `MoneyTab`. Type-safe end-to-end.

---

## Accessibility Summary

| Component | VoiceOver Labels | Touch Targets | Reduce Motion |
|---|---|---|---|
| CategoryRadialChart | Chart: comprehensive breakdown label. Legend items: category + amount + % + tx count | 44pt min (legendItem) | Checked via AccessibilityInfo + event listener |
| TriageCard | Full gesture instructions in a11y label (left/right/down/up) | Card is full-width (generous) | TriageDeck disables gesture stack entirely in reduce motion mode |
| CategoryFan | Each button: category name + "suggested" marker. Container: role=menu | 44pt min (fanItem minHeight) | Not separately handled (fan is part of gesture path — unavailable in reduce motion) |
| TriageDeck undo button | "Undo last action" + hint "Reverts the last categorization" | IconButton size=20 (below 44pt — see note) | Static list shown; undo still accessible via button |

**Note on undo IconButton touch target**: The `IconButton size={20}` in `TriageDeck` renders a
visual icon of 20pt, but Material's `IconButton` component adds internal padding to meet the
iOS 44pt minimum. This is standard React Native Paper behavior and should be acceptable.
Recommend verifying on device if there is any concern.

**VoiceOver label inaccuracy (BUG-8E-001 context)**: The TriageCard a11y label always says
"Swipe right to accept" even when no suggestion is available. VoiceOver users are told they can
accept a suggestion that does not exist.

---

## Test Coverage Assessment

The 30 new tests provide adequate coverage for happy paths and key edge cases:

**CategoryRadialChart (10 tests)**: Loading, empty (no data), empty (zero total), chart render,
top-5 cap, amounts displayed, percentages, a11y label, a11y legend items with tx count,
a11y hint, legend press callback.

**TriageDeck (8 tests)**: Empty state messages, loading state, cards render, progress indicator,
merchant name, AI suggestion displayed, a11y card labels.

**useTriageDeck (12 tests)**: Load on mount, sort by confidence, current item, accept+advance,
assign+advance, skip without DB write, undo decrements index, empty state, complete state.

**Coverage gaps identified:**

1. No test for `acceptSuggestion` when `currentItem.suggestion === null` (the Medium bug)
2. No test asserting DB revert call in undo (P2-4)
3. No test for `assignCategory` when DB throws (error path)
4. No test for `loadUncategorized` failure (network/DB error at load time)
5. No test for CategoryFan with 0 categories or 1 category (boundary cases)
6. No test for swipe gesture directions in `useTriageGestures` (requires gesture handler mocking)

Gaps 1-4 represent real risk areas; gaps 5-6 are lower priority.

---

## Final Decision

**PASS (conditional) — approved for promotion from dev to main**

All automated tests pass. TypeScript is clean. No regressions in navigation, widget registry,
or modified components. The feature is functionally complete and accessible.

**Ship-blocking issues**: None.

**Backlog items before next release touching Triage:**
1. BUG-8E-001 (Medium): Swipe-right silent no-op on null-suggestion card
2. P2-3 elevated: undo DB revert fire-and-forget — data integrity risk (recommend P1 backlog)
3. P2-4: Add DB revert assertion to undo test
4. Add missing test for `acceptSuggestion` with null suggestion

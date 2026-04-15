# UX Review: Multi-Account Cash Flow Tracking

**Reviewer**: UX Designer
**Date**: 2026-01-18
**Proposal**: `openspec/changes/add-multi-account-cashflow/`

---

## Executive Summary

This proposal adds four major features: Multi-Account Management, Income Tracking, Bill Tracking, and a Cash Flow Dashboard. While these features address real user needs (separating personal/business finances, understanding cash flow), the **cumulative cognitive load is substantial**. The UX review identifies critical concerns around information architecture, navigation complexity, and user onboarding.

**Key Recommendations:**
1. **Phase the rollout** - Don't launch all features at once
2. **Redesign tab navigation** - 4 tabs → 5+ tabs exceeds iOS HIG recommendations
3. **Simplify account switching** - Current dropdown pattern may be too passive
4. **Improve bill payment UX** - "Mark paid" interaction needs clearer affordances
5. **Add comprehensive onboarding** - Users will need guidance to understand new mental model

---

## 1. User Journey Mapping

### Journey 1: Creating and Switching Accounts

**User Goal**: Separate personal and business expenses into distinct accounts

```
┌─────────────────────────────────────────────────────────────────────┐
│ Stage      │ Discover     │ Create       │ Use          │ Switch    │
├─────────────────────────────────────────────────────────────────────┤
│ Actions    │ Opens app    │ Taps header  │ Adds expense │ Taps      │
│            │ (first time) │ switcher     │ to business  │ switcher  │
│            │              │ → Create New │ account      │ again     │
├─────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "What's this │ "What should │ "Did this go │ "Which    │
│            │ header thing"│ I name it?"  │ to the right │ account   │
│            │              │              │ account?"    │ am I in?" │
├─────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 🤔 Curious   │ 😐 Neutral   │ 😰 Uncertain │ 😐 OK     │
├─────────────────────────────────────────────────────────────────────┤
│ Pain       │ No context   │ Type vs name │ No visual    │ Header is │
│ Points     │ for why this │ distinction  │ confirmation │ small hit │
│            │ exists       │ unclear      │ of scope     │ target    │
├─────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Onboarding   │ Suggested    │ Toast/badge  │ Active    │
│ unities    │ flow for     │ names + icon │ showing      │ account   │
│            │ new feature  │ selection    │ active acct  │ indicator │
└─────────────────────────────────────────────────────────────────────┘
```

**Critical Issues:**
- ❌ **Discoverability**: Header dropdown is a passive UI element - users may never notice it
- ❌ **Confirmation**: No visual feedback confirming which account is active after switch
- ❌ **Context loss**: Switching accounts changes ALL data - users may feel disoriented

**Recommendations:**
1. **First-run experience**: When feature launches, show modal explaining accounts with "Set Up Accounts" CTA
2. **Visual prominence**: Use color coding in header (not just dropdown) - e.g., colored accent bar under header
3. **Switch confirmation**: Show toast after switching: "Switched to [Account Name]"
4. **Account badge on FAB**: Show small account color indicator on the expense FAB to reinforce context

---

### Journey 2: Adding Income (One-Time)

**User Goal**: Record a freelance payment received today

```
┌─────────────────────────────────────────────────────────────────────┐
│ Stage      │ Navigate     │ Fill Form    │ Save         │ Verify    │
├─────────────────────────────────────────────────────────────────────┤
│ Actions    │ Tap Income   │ Enter amount │ Tap Save     │ Check     │
│            │ tab          │ Select cat.  │              │ dashboard │
├─────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "New tab     │ "Similar to  │ "Will this   │ "Did it   │
│            │ appeared?"   │ expense form"│ show up?"    │ count?"   │
├─────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 🤔 Confused  │ 😊 Familiar  │ 😐 OK        │ 😊 Good   │
├─────────────────────────────────────────────────────────────────────┤
│ Pain       │ Tab bar now  │ N/A          │ N/A          │ N/A       │
│ Points     │ has 5+ items │              │              │           │
├─────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Consolidate  │ Smart        │ Success msg  │ Link to   │
│ unities    │ tabs or use  │ defaults     │ with preview │ dashboard │
│            │ submenu      │ (today)      │ of totals    │ from form │
└─────────────────────────────────────────────────────────────────────┘
```

**Critical Issues:**
- ⚠️ **Navigation clutter**: Adding 3 new tabs (Dashboard, Income, Bills) brings total to **7 tabs** (Expenses, Mileage, Reports, Settings + 3 new) - **exceeds iOS HIG max of 5**
- ✅ **Form familiarity**: Income form mirrors expense form (good consistency)

**Recommendations:**
1. **Consolidate navigation** (see Navigation section below)
2. **Success state**: After saving income, show snackbar: "Income added • [View Dashboard →]" with link
3. **Smart defaults**: Date defaults to today (already in spec), category defaults to last-used

---

### Journey 3: Managing Bills

**User Goal**: Mark monthly rent as paid

```
┌─────────────────────────────────────────────────────────────────────┐
│ Stage      │ Navigate     │ Find Bill    │ Mark Paid    │ Confirm   │
├─────────────────────────────────────────────────────────────────────┤
│ Actions    │ Tap Bills    │ Scroll/      │ Tap bill →   │ Check     │
│            │ tab          │ search       │ ...what now? │ expenses  │
├─────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "Another new │ "Is this     │ "Where's the │ "Did it   │
│            │ tab"         │ overdue?"    │ mark paid    │ create an │
│            │              │              │ button?"     │ expense?" │
├─────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 😐 OK        │ 😰 Anxious   │ 😕 Unsure    │ 🤔 Unsure │
├─────────────────────────────────────────────────────────────────────┤
│ Pain       │ N/A          │ Overdue = red│ Unclear      │ No conf-  │
│ Points     │              │ but where's  │ interaction  │ irmation  │
│            │              │ action?      │ pattern      │ message   │
├─────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Dashboard    │ Sort over-   │ Swipe-right  │ Toast msg │
│ unities    │ quick action │ due to top + │ or button in │ linking   │
│            │ for bills    │ action btn   │ detail view  │ to expense│
└─────────────────────────────────────────────────────────────────────┘
```

**Critical Issues:**
- ❌ **Interaction ambiguity**: Spec doesn't define HOW user marks bill as paid (tap bill to detail → button? Swipe action? Checkbox on list?)
- ⚠️ **Overdue indicator**: Red badge is good, but color-only = accessibility issue
- ⚠️ **Silent expense creation**: "Silently creates expense" - users may not realize this happened

**Recommendations:**
1. **Define interaction pattern**:
   - **Option A (Recommended)**: Swipe right on bill → "Mark Paid" action (green) + haptic + toast: "Bill paid • Expense created"
   - **Option B**: Checkbox on list item (less iOS-native, but clearer affordance)
   - **Option C**: Tap bill → detail view → "Mark as Paid" button (slower, but explicit)
2. **Overdue handling**:
   - Add text label "Overdue" (not just red color) for accessibility
   - Sort overdue bills to top with section header: "Overdue (2)" in red
3. **Confirmation feedback**:
   - After marking paid, show snackbar: "Rent marked paid • Expense created ($1500)"
   - Include UNDO option (what if misclick?)

---

### Journey 4: Using Cash Flow Dashboard

**User Goal**: See if I'm cash-positive this month

```
┌─────────────────────────────────────────────────────────────────────┐
│ Stage      │ Navigate     │ Interpret    │ Drill Down   │ Take      │
│            │              │              │              │ Action    │
├─────────────────────────────────────────────────────────────────────┤
│ Actions    │ Tap Dash-    │ Read summary │ Tap category │ Adjust    │
│            │ board tab    │ cards        │ to see list  │ spending  │
├─────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "New main    │ "Am I doing  │ "What's      │ "Need to  │
│            │ screen?"     │ well?"       │ driving X?"  │ cut back" │
├─────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 😊 Curious   │ 😊/😰 Varies │ 🤔 Curious   │ 💪 Motiv. │
├─────────────────────────────────────────────────────────────────────┤
│ Pain       │ Too many     │ Info density │ Drill-down   │ N/A       │
│ Points     │ things to    │ may be high  │ unclear      │           │
│            │ track now    │              │              │           │
├─────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Make dash-   │ Progressive  │ Tappable     │ Quick     │
│ unities    │ board default│ disclosure   │ cards link   │ actions   │
│            │ home screen  │ (summary →   │ to lists     │ present   │
│            │              │ detail)      │              │           │
└─────────────────────────────────────────────────────────────────────┘
```

**Critical Issues:**
- ⚠️ **Information density**: Dashboard shows 9+ data points (income, expenses, net, YTD, bills summary, category breakdown, trend) - risk of overwhelming users
- ⚠️ **Date range complexity**: 6 preset options + custom range = decision paralysis

**Recommendations:**
1. **Progressive disclosure**:
   - **Default view**: Show only 3 hero metrics: Income, Expenses, Net (large cards)
   - **Expandable sections**: Bills, Category Breakdown, Trends = collapsed by default with "Show More" affordance
2. **Simplify date range**:
   - Default to "This Month" (most common use case)
   - Show only 3 presets in primary view: This Month | Last Month | This Year
   - "Custom" buried in "More" menu
3. **Visual hierarchy**:
   - Net amount = hero (largest, center, color-coded)
   - Income/Expenses = secondary (smaller cards flanking net)
   - Everything else = tertiary (list items below)

---

## 2. Usability Concerns

### A. Cognitive Overload Risks

**Issue**: Adding 4 major features simultaneously overwhelms users with new concepts.

**Impact**: High

**Evidence**:
- App currently has 2 primary entities (Expenses, Mileage)
- Proposal adds 3 new entities (Accounts, Income, Bills) + new dashboard
- Users must learn:
  - What accounts are and when to use them
  - Difference between income vs expenses vs bills
  - How to interpret cash flow metrics
  - When bills become expenses

**Mental Model Shift**:
```
BEFORE:                          AFTER:
┌──────────────┐                ┌──────────────────────────────┐
│ Expenses     │                │ Accounts (context layer)     │
│ Mileage      │                │  ├─ Expenses                 │
│ Reports      │     →          │  ├─ Income (NEW)             │
└──────────────┘                │  ├─ Bills (NEW)              │
                                │  ├─ Mileage                  │
                                │  └─ Dashboard (NEW)          │
                                └──────────────────────────────┘
```

**Recommendation**:
1. **Phased rollout**:
   - **Phase 1**: Accounts only (let users adapt to scoping)
   - **Phase 2**: Income tracking (3-4 weeks later)
   - **Phase 3**: Bills + Dashboard (another 3-4 weeks)
2. **Feature flags**: Allow early adopters to opt-in to test new features
3. **Onboarding**: Multi-step tutorial explaining the new model

---

### B. Navigation Complexity

**Issue**: Proposal implicitly adds 3+ new tabs, exceeding iOS HIG limits.

**Impact**: Critical

**Current State** (4 tabs):
```
┌─────────────────────────────────────────────────────┐
│  [Expenses]  [Mileage]  [Reports]  [Settings]      │
└─────────────────────────────────────────────────────┘
```

**Proposed State** (7+ tabs):
```
┌──────────────────────────────────────────────────────────────────┐
│  [Dashboard] [Expenses] [Income] [Bills] [Mileage] [Reports]... │
│                        ⬆ TOO MANY - text truncated              │
└──────────────────────────────────────────────────────────────────┘
```

**iOS Human Interface Guidelines**: Maximum 5 tabs for usability

**Recommendation - Option A: Consolidate into 5 tabs (Preferred)**:
```
┌─────────────────────────────────────────────────────┐
│  [Dashboard]  [Finances]  [Reports]  [Settings]    │
│                   ⬇                                 │
│           Finances tab shows:                       │
│           • Expenses (list)                         │
│           • Income (list)                           │
│           • Bills (list)                            │
│           Via segmented control or sections         │
└─────────────────────────────────────────────────────┘
```

**Recommendation - Option B: Hub & Spoke Model**:
```
┌─────────────────────────────────────────────────────┐
│  [Dashboard]  [Expenses]  [Mileage]  [Settings]    │
│       ⬇                                             │
│   Dashboard = hub with cards:                       │
│   • Income Summary (tap → income list)              │
│   • Expense Summary (tap → expense list)            │
│   • Bills Summary (tap → bills list)                │
└─────────────────────────────────────────────────────┘
```

**Rationale**: Dashboard becomes home screen, other features accessible via tappable cards

---

### C. Discoverability Issues

**Issue 1: Account Switcher in Header**

**Problem**: Dropdown in header is passive - users may never discover it

**Existing Pattern**: App uses persistent visible elements (tabs, FABs) - header has only been for navigation titles

**User Mental Model**: "Headers are for titles, not interactions"

**Recommendation**:
1. **First-launch modal**: When feature ships, show "New: Multiple Accounts" modal with setup wizard
2. **Persistent indicator**: Always show current account name in header with color stripe/badge
3. **Visual cue**: Add chevron or icon next to account name to signal tappability
4. **Alternative placement**: Consider account switcher as first item in Settings tab (more discoverable)

---

**Issue 2: Bill "Mark Paid" Interaction**

**Problem**: Spec doesn't define the interaction pattern - UX risk

**Options Analysis**:

| Pattern | Pros | Cons | Recommendation |
|---------|------|------|----------------|
| **Swipe Right** | Fast, iOS-native, muscle memory | Requires discovery, no undo if not careful | ✅ Best for power users |
| **Checkbox on List** | Clear affordance, obvious | Not iOS-native, less elegant | ⚠️ OK for accessibility |
| **Tap → Detail → Button** | Explicit, hard to misclick | Slow (3 taps), friction | ❌ Too slow for frequent task |
| **Long-press → Menu** | iOS-native, works with swipe | Slower than swipe, less obvious | ⚠️ Good alternative |

**Recommendation**: **Hybrid approach**
- Primary: Swipe right for "Mark Paid" (green) - fast for frequent users
- Alternative: Tap bill → detail view has "Mark as Paid" button (for new users)
- Accessibility: Long-press menu with same options (for VoiceOver users)

---

**Issue 3: Recurring Bills Reset**

**Problem**: Spec says bills reset "when user opens the app after month change"

**UX Concern**: Passive, delayed action - users may not notice bills reset until later

**Scenario**:
- User pays rent on Jan 31
- Opens app on Feb 3
- Rent bill is suddenly unpaid again
- User thinks: "Did I forget to mark it? Is this a bug?"

**Recommendation**:
1. **Proactive notification**: On first app open of new month, show modal:
   ```
   "New Month, New Bills"
   3 recurring bills reset for February:
   • Rent - $1,500
   • Netflix - $15
   • Insurance - $200
   [View Bills] [Dismiss]
   ```
2. **Visual indicator**: Show "Reset on [date]" label on recurring bills
3. **History preservation**: Let users view last month's bill payment history (currently spec deletes expense link)

---

### D. Potential Confusion Points

#### 1. Bills vs Expenses: Mental Model Confusion

**Problem**: Users must understand that bills are "pre-expenses" - many will wonder "why track twice?"

**Confusing Flow**:
```
User creates bill → marks paid → expense auto-created → now appears in both Bills AND Expenses
```

**Questions users will ask**:
- "Why is my rent showing up in both places?"
- "Do I add bills OR expenses for recurring things?"
- "If I delete the expense, does it unmark the bill?"

**Recommendation**:
1. **Clear explanation**: Bill detail view shows: "When marked paid, this will create an expense entry"
2. **Visual link**: Bill list item shows "→ Expense" icon when paid (tappable to navigate)
3. **Onboarding**: Tutorial explicitly explains: "Bills help you track what's DUE, expenses track what's PAID"
4. **Don't show bill-linked expenses twice**: In expense list, mark bill-created expenses with "📋 Bill" badge
5. **Spec clarification needed**: What happens if user edits the expense after paying bill? Does bill amount stay out of sync?

---

#### 2. Account Scope: Context Loss

**Problem**: Switching accounts changes ALL visible data - can be disorienting

**Scenario**:
- User in "Personal" account, viewing expenses
- Taps account switcher, switches to "Business"
- Entire expense list changes instantly
- User thinks: "Wait, where did my data go?"

**Recommendation**:
1. **Confirmation**: After switch, show toast: "Switched to [Business] • All data now scoped to this account"
2. **Persistent indicator**: Show active account name + color in EVERY screen header (not just home)
3. **Breadcrumb**: Settings screen shows "Account: [Name]" with edit button
4. **Safety**: Warn before deleting account: "This account has 47 expenses. Export first?" (already in spec ✅)

---

#### 3. Color Picker UX

**Problem**: Spec mentions color coding but doesn't define color picker UX

**Concerns**:
- How many colors? (too few = conflicts, too many = decision paralysis)
- Can users pick custom colors? (nice but adds complexity)
- What if two accounts have same color? (confusing)

**Recommendation**:
1. **Preset palette**: 8-10 distinct colors (ensure WCAG contrast with backgrounds)
2. **Smart defaults**:
   - First account (Personal): Blue
   - Second account (Business): Green
   - Third+ account: Rotate through palette
3. **Color uniqueness**: If color already used, show subtle indicator: "(Used by Personal)"
4. **Accessibility**: Always pair color with icon or pattern for color-blind users

---

## 3. Interaction Pattern Recommendations

### Account Switcher

**Current Spec**: "Account switcher dropdown in the app header"

**UX Pattern**: Bottom sheet selector (recommended over dropdown)

**Rationale**:
- Larger touch targets
- More space for account details (name, type badge, color)
- Native iOS pattern for selection
- Can include "Create New Account" as list item

**Design**:
```
┌──────────────────────────────────────┐
│  Select Account                 [X]  │
├──────────────────────────────────────┤
│  🔵 Personal (Personal)         ✓   │  ← Active account
│  🟢 Consulting LLC (Business)       │
│  🟠 Side Hustle (Business)          │
├──────────────────────────────────────┤
│  + Create New Account               │
└──────────────────────────────────────┘
```

**Interaction**:
- Tap header → bottom sheet slides up
- Tap account → sheet dismisses + switch with haptic + toast
- Swipe down or tap X to dismiss without switching

---

### Bill "Mark Paid" Interaction

**Recommended Pattern**: Swipe + Detail Button (hybrid)

**Primary (List View)**:
```
┌──────────────────────────────────────┐
│  Rent                       $1,500   │  ← Swipe right
│  Due: 1st • Overdue           🔴    │     to mark paid
│                                      │
│  ← Swipe Left: Delete               │
│  → Swipe Right: Mark Paid           │
└──────────────────────────────────────┘
```

**Secondary (Detail View)**:
```
┌──────────────────────────────────────┐
│  Rent                                │
│  Amount: $1,500                      │
│  Due: 1st of every month             │
│  Category: Rent                      │
│  Status: Unpaid • Overdue            │
│                                      │
│  [Mark as Paid] ← Large button      │
└──────────────────────────────────────┘
```

**Feedback**:
- After marking paid: Green checkmark animation + haptic + snackbar:
  ```
  "Rent marked paid • Expense created ($1,500) [UNDO]"
  ```

---

### Dashboard Date Range Picker

**Current Spec**: "Preset date range selection" with 6 options

**UX Pattern**: Segmented control + "More" button

**Design**:
```
┌──────────────────────────────────────┐
│  Cash Flow                           │
│                                      │
│  [This Month] [Last Month] [More▾]  │  ← Segmented control
│                                      │
│  Net: +$3,245  ↑ 12%                │
└──────────────────────────────────────┘
```

**"More" menu** (bottom sheet):
```
┌──────────────────────────────────────┐
│  Select Date Range                   │
├──────────────────────────────────────┤
│  This Week                           │
│  This Quarter                        │
│  This Year                           │
│  Custom Range...                     │
└──────────────────────────────────────┘
```

**Rationale**: Reduces decision paralysis, prioritizes common use cases

---

### Color Picker (Account Creation/Edit)

**UX Pattern**: Horizontal color swatch picker

**Design**:
```
┌──────────────────────────────────────┐
│  Account Color                       │
│  ┌────────────────────────────────┐  │
│  │ 🔵 🟢 🟠 🟡 🔴 🟣 🟤 ⚫       │  │  ← Swipe horizontally
│  │    ⬆                           │  │     Selected = checkmark
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

**Colors** (WCAG AA compliant):
- Blue (#2196F3), Green (#4CAF50), Orange (#FF9800), Yellow (#FFC107)
- Red (#F44336), Purple (#9C27B0), Brown (#795548), Gray (#607D8B)

**Interaction**:
- Tap color → updates preview + light haptic
- Selected color shows checkmark
- If color in use, show label below: "(Used by Personal)" in gray

---

## 4. iOS Pattern Alignment

### ✅ Patterns Aligned with iOS HIG

1. **Bottom Tab Navigation**: Using tab bar for top-level navigation (✅)
2. **Swipe Actions**: Left/right swipe for delete/actions (✅)
3. **Modal Sheets**: For focused tasks (account creation, bill detail) (✅)
4. **Pull-to-Refresh**: For lists (already implemented) (✅)
5. **Search + Filters**: Standard iOS pattern (already in expense list) (✅)

### ⚠️ Patterns Needing Adjustment

#### 1. Tab Bar Overflow (CRITICAL)

**Issue**: 7+ tabs exceeds iOS HIG maximum

**iOS HIG**: "Use 3 to 5 tabs. Too many tabs make it difficult to tap the one you want."

**Current Risk**: Tabs will truncate text or show "More" tab (poor UX)

**Fix**: See Navigation Complexity recommendations above

---

#### 2. Header Dropdown vs iOS Native Patterns

**Issue**: Dropdown in header is uncommon in iOS apps

**Common iOS Patterns for Account Switching**:
1. **Settings-based**: Switch account in Settings tab (most common)
2. **Sheet selector**: Tap header → bottom sheet appears (medium common)
3. **Sidebar**: Swipe from left to reveal account selector (iPadOS pattern)

**Recommendation**: Use bottom sheet (Option 2) - balances accessibility with discoverability

---

#### 3. "Silently" Creating Expenses

**Issue**: Spec says bills "silently create expense entries"

**iOS Pattern**: Users expect feedback for every action

**Recommendation**: NOT silent - show toast: "Bill paid • Expense created" (see above)

---

### ✅ Patterns Spec Got Right

1. **Swipe-to-delete with undo**: Already implemented for expenses - extend to bills (✅)
2. **FAB for primary action**: Add expense FAB (✅) - replicate for income/bills
3. **Form consistency**: Income form mirrors expense form structure (✅)
4. **Color coding**: Visual categorization (✅) - just needs picker UX defined
5. **Date selection**: iOS native date picker (✅)

---

## 5. Accessibility Considerations

### Critical Issues

#### Issue 1: Color-Only Indicators

**Problem**: Overdue bill badge = red only

**WCAG Violation**: Color alone cannot convey information (1.4.1 Use of Color)

**Fix**:
```
BAD:  [Bill Name] 🔴              ← Only color
GOOD: [Bill Name] 🔴 Overdue      ← Color + text
BEST: [Bill Name] ⚠️ Overdue      ← Icon + color + text
```

**Recommendation**:
- Overdue: Red badge with "⚠️ Overdue" text
- Paid: Green checkmark with "✓ Paid" text
- Unpaid: No badge (neutral state)

---

#### Issue 2: Account Color Differentiation

**Problem**: Color coding accounts - colorblind users can't distinguish

**Solution**: Pair color with unique icon or pattern

**Example**:
```
Personal:  🔵 Icon: 👤 (person)
Business:  🟢 Icon: 💼 (briefcase)
Freelance: 🟠 Icon: 💻 (laptop)
```

**Implementation**:
- Account creation: Let user pick both color AND icon
- Header: Show "👤 Personal" (icon + name), not just color dot
- Bottom sheet: Show both icon and color in account list

---

#### Issue 3: Touch Target Sizes

**Concern**: Spec doesn't define UI sizes - ensure compliance

**Requirements** (per iOS HIG + existing app patterns):
- Minimum touch target: 44×44 pt (✅ already in IOS_UX_PATTERNS.md)
- Bills list item: 64 pt tall minimum (✅ same as expense list)
- Account switcher header: Needs 44pt minimum height for tappable area
- Color picker swatches: 40×40 pt each with 8pt spacing

**Recommendation**: Add explicit size requirements to implementation spec

---

#### Issue 4: Screen Reader Announcements

**Required VoiceOver Labels**:

**Account Switcher**:
```
accessibilityLabel: "Account: Personal. Button. Double tap to switch accounts."
```

**Bill List Item**:
```
accessibilityLabel: "Rent, $1,500, due on 1st, overdue by 3 days, unpaid. Button."
accessibilityHint: "Double tap to view details or mark as paid."
```

**Dashboard Net Amount**:
```
accessibilityLabel: "Net cash flow: positive $3,245, up 12% from last month"
```

**Mark Paid Action**:
```
accessibilityLabel: "Mark Rent as paid, $1,500"
accessibilityHint: "Creates an expense entry for this payment"
```

---

### Gesture Alternatives (WCAG 2.5.1)

**Required**: Every swipe/gesture must have button alternative

| Gesture | Alternative |
|---------|-------------|
| Swipe right to mark paid | Tap bill → "Mark Paid" button in detail view |
| Swipe left to delete bill | Tap bill → "Delete" button in detail view |
| Pull-to-refresh | Refresh button in header (already exists ✅) |

**Recommendation**: Add long-press menu to bill list items:
```
Long press bill → Menu:
  • Mark as Paid
  • Edit Bill
  • View History
  • Delete
```

---

## 6. Actionable Recommendations

### Priority 0 (Must-Have Before Launch)

1. **Resolve navigation structure** - 7 tabs is a blocker
   - **Action**: Choose Option A (Finances consolidated tab) or Option B (Dashboard hub model)
   - **Owner**: Architect + Frontend
   - **Timeline**: Before any implementation starts

2. **Define bill "mark paid" interaction**
   - **Action**: Implement swipe-right pattern with button alternative
   - **Owner**: UX Designer (create detailed interaction spec)
   - **Deliverable**: Figma prototype showing interaction

3. **Fix accessibility issues**
   - **Action**: Add text labels to all color indicators, ensure 44pt touch targets
   - **Owner**: Frontend Dev
   - **Validation**: VoiceOver testing required

4. **Add onboarding flow**
   - **Action**: Create multi-step tutorial for first launch
   - **Steps**:
     1. "Welcome to Accounts" (explain concept)
     2. "Create Your First Business Account" (wizard)
     3. "Track Income & Bills" (feature tour)
   - **Owner**: Frontend Dev + UX Designer
   - **Estimated screens**: 3-4 modals

---

### Priority 1 (Strongly Recommended)

5. **Phase the rollout**
   - **Action**: Split into 3 releases (Accounts → Income → Bills+Dashboard)
   - **Rationale**: Reduce cognitive load, allow user adaptation
   - **Owner**: Product/PM + Architect
   - **Timeline**: 3-4 weeks between phases

6. **Improve account switcher discoverability**
   - **Action**: Show colored accent bar under header + chevron icon
   - **Owner**: Frontend Dev
   - **Visual**: UX Designer to provide mockup

7. **Add confirmation feedback**
   - **Action**: Toasts for: account switch, bill marked paid, recurring bills reset
   - **Owner**: Frontend Dev
   - **Pattern**: Follow existing snackbar pattern (already in expense list ✅)

8. **Dashboard progressive disclosure**
   - **Action**: Collapse secondary sections by default (bills detail, category breakdown)
   - **Owner**: Frontend Dev
   - **Rationale**: Reduce initial information density

---

### Priority 2 (Nice-to-Have)

9. **Smart defaults and suggestions**
   - Account creation: Suggest names ("Personal", "Business", "Freelance")
   - Color picker: Pre-select next unused color
   - Bill creation: Suggest due date based on current date (1st, 15th, etc.)

10. **Enhanced visual feedback**
    - Checkmark animation when marking bill paid
    - Account color pulse animation on switch
    - Trend indicator animation on dashboard

11. **Historical view for bills**
    - "View Payment History" button on recurring bills
    - Shows last 6 months of payment dates
    - Links to created expenses

---

### Spec Additions Needed

The following details are **missing from current specs** and need definition:

| Item | Current State | Needed |
|------|---------------|--------|
| **Account switcher UI** | "Dropdown in header" | Full interaction spec (bottom sheet design, touch targets) |
| **Bill mark paid interaction** | Not specified | Pattern definition (swipe vs button vs both) |
| **Color picker UX** | Mentioned but not defined | Number of colors, picker UI, collision handling |
| **Dashboard info hierarchy** | Flat list of features | Prioritization, progressive disclosure rules |
| **Onboarding flow** | Not mentioned | Multi-step tutorial design |
| **Error states** | Not specified | What happens if bill payment fails to create expense? |
| **Undo behavior** | Not specified | Can user undo marking bill as paid? For how long? |

**Recommendation**: UX Designer to create supplementary spec document addressing these gaps before handoff to frontend.

---

## 7. Summary & Next Steps

### Summary of Key Concerns

| Category | Issue | Severity | Fix |
|----------|-------|----------|-----|
| **Navigation** | 7+ tabs exceeds iOS HIG | 🔴 Critical | Consolidate to 5 tabs |
| **Cognitive Load** | 4 major features at once | 🟠 High | Phase rollout over 8-12 weeks |
| **Interaction** | Bill "mark paid" undefined | 🟠 High | Define swipe pattern |
| **Accessibility** | Color-only indicators | 🟠 High | Add text labels + icons |
| **Discoverability** | Account switcher passive | 🟡 Medium | Add visual cues + onboarding |
| **Confirmation** | Silent expense creation | 🟡 Medium | Add toast feedback |
| **Complexity** | Dashboard info density | 🟡 Medium | Progressive disclosure |

---

### Recommended Next Steps

**Immediate (Before Implementation Starts)**:
1. **Architecture decision**: Choose navigation model (consolidated tabs vs hub-and-spoke)
2. **UX spec supplement**: Create detailed interaction specs for:
   - Account switcher bottom sheet
   - Bill mark paid interaction
   - Color picker UI
   - Dashboard layout hierarchy
3. **Onboarding design**: Create first-run tutorial flow

**During Implementation**:
4. **Prototype testing**: Test account switcher and bill payment with 3-5 users
5. **Accessibility audit**: VoiceOver walkthrough of all new screens
6. **Visual design**: Finalize color palette, icon set, and animation specs

**Before Launch**:
7. **Usability testing**: Full flow testing with 5-7 target users (freelancers, small business owners)
8. **Accessibility validation**: WCAG 2.1 AA compliance check
9. **Onboarding refinement**: Adjust tutorial based on user feedback

---

### Open Questions for Team

1. **Navigation**: Which model do we prefer - consolidated "Finances" tab or dashboard hub?
2. **Rollout**: Are we committed to phased rollout or full launch?
3. **Onboarding**: Mandatory tutorial or skippable?
4. **Bills**: Should we allow non-recurring bills, or push users to just create expenses directly?
5. **Accounts**: Maximum number of accounts? (suggest 10 limit to prevent abuse)
6. **Color conflicts**: Enforce unique colors per account or allow duplicates?

---

## Appendix: Example User Flows (Detailed)

### Flow: First-Time User Creates Business Account

**Precondition**: User updated app, opens for first time with new features

```
Step 1: Feature Introduction Modal
┌─────────────────────────────────────────┐
│  ✨ New: Multiple Accounts              │
│                                         │
│  Separate your personal and business   │
│  expenses into different accounts.     │
│                                         │
│  [Set Up Now]  [Maybe Later]           │
└─────────────────────────────────────────┘

If [Set Up Now]:

Step 2: Migration Notice
┌─────────────────────────────────────────┐
│  Your existing data has been moved to   │
│  a "Personal" account.                  │
│                                         │
│  Ready to create a business account?   │
│                                         │
│  [Create Business Account]             │
└─────────────────────────────────────────┘

Step 3: Account Creation Form (Bottom Sheet)
┌─────────────────────────────────────────┐
│  New Account                       [X]  │
├─────────────────────────────────────────┤
│  Name *                                 │
│  [Consulting LLC          ]             │
│                                         │
│  Type *                                 │
│  ( ) Personal  (•) Business             │
│                                         │
│  Color                                  │
│  🔵 🟢 🟠 🟡 🔴 🟣 🟤 ⚫              │
│     ⬆                                   │
│  Icon                                   │
│  👤 💼 💻 🏠 🚗 ✈️ 🎓 📱              │
│        ⬆                                │
│                                         │
│  [Create Account]                       │
└─────────────────────────────────────────┘

Step 4: Success + Context Switch
┌─────────────────────────────────────────┐
│  ✓ Consulting LLC created               │
│                                         │
│  Switched to: 💼 Consulting LLC         │  ← Toast
│  All new expenses will go here.         │
│                                         │
└─────────────────────────────────────────┘
   (Auto-dismisses after 3 seconds)

Step 5: Header Updates
┌─────────────────────────────────────────┐
│  💼 Consulting LLC ▾     [Filter] [Sort]│  ← Green accent bar
├─────────────────────────────────────────┤
│  Expenses                               │
│                                         │
│  (Empty state - new account)            │
└─────────────────────────────────────────┘
```

**Total steps**: 5 interactions (modal → modal → form → toast → arrival)
**Estimated time**: 45-60 seconds

---

### Flow: User Marks Recurring Bill as Paid

**Precondition**: User has recurring "Rent" bill set up, due on 1st, today is the 3rd (overdue)

```
Step 1: Navigate to Bills Tab
┌─────────────────────────────────────────┐
│  Bills                                  │
├─────────────────────────────────────────┤
│  OVERDUE (1)                            │  ← Red section header
│  ┌─────────────────────────────────────┐│
│  │⚠️ Rent                    $1,500    ││  ← Red badge
│  │ Due: 1st • 2 days overdue           ││
│  └─────────────────────────────────────┘│
│                                         │
│  UPCOMING (2)                           │
│  ┌─────────────────────────────────────┐│
│  │ Netflix                     $15     ││
│  │ Due: 15th                           ││
│  └─────────────────────────────────────┘│
│  ┌─────────────────────────────────────┐│
│  │ Car Insurance              $200     ││
│  │ Due: 20th                           ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘

Step 2: User Swipes Right on Rent
┌─────────────────────────────────────────┐
│  ⚠️ Rent          <---- [Mark Paid]    │  ← Green action
│                          $1,500         │     revealed
│  Due: 1st • 2 days overdue              │
└─────────────────────────────────────────┘
   (Medium haptic feedback on swipe)

Step 3: User Completes Swipe
   (Heavy haptic feedback on release)
   (Checkmark animation plays)

┌─────────────────────────────────────────┐
│  ✓ Rent                        $1,500  │  ← Green checkmark
│  Paid on Jan 3                          │     fades in
│  ✓ Paid                                 │
└─────────────────────────────────────────┘

Step 4: Snackbar Appears
┌─────────────────────────────────────────┐
│                                         │
│  Rent marked paid                       │
│  Expense created ($1,500)               │  ← Bottom snackbar
│                                         │     5 sec duration
│                            [UNDO]       │
└─────────────────────────────────────────┘

Step 5: Bill Moves to "PAID" Section (Auto-sorts)
┌─────────────────────────────────────────┐
│  Bills                                  │
├─────────────────────────────────────────┤
│  UPCOMING (2)                           │  ← Overdue section
│  ┌─────────────────────────────────────┐│     gone!
│  │ Netflix                     $15     ││
│  │ Due: 15th                           ││
│  └─────────────────────────────────────┘│
│  ┌─────────────────────────────────────┐│
│  │ Car Insurance              $200     ││
│  │ Due: 20th                           ││
│  └─────────────────────────────────────┘│
│                                         │
│  PAID THIS MONTH (1)                    │
│  ┌─────────────────────────────────────┐│
│  │✓ Rent                      $1,500   ││
│  │  Paid on Jan 3                      ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

**Total steps**: 1 gesture (swipe right)
**Feedback points**: 4 (haptic on swipe, haptic on release, checkmark animation, snackbar)
**Estimated time**: 3-5 seconds

---

## Conclusion

The multi-account cash flow proposal addresses real user needs but introduces significant UX complexity. The **primary concern is navigation overload** - adding 3 new tabs to an existing 4-tab structure exceeds iOS guidelines and risks overwhelming users.

**Key Recommendations**:
1. **Consolidate navigation** to 5 tabs maximum (critical)
2. **Phase the rollout** over 8-12 weeks to reduce cognitive load
3. **Define missing interactions** (bill mark paid, account switcher, color picker)
4. **Add comprehensive onboarding** for first-time feature users
5. **Ensure accessibility** (text labels, touch targets, VoiceOver support)

With these adjustments, the feature set can deliver substantial value to users tracking business finances while maintaining the app's core usability and iOS-native feel.

---

**Next Action**: UX Designer to create supplementary interaction specs and visual mockups for:
- Account switcher bottom sheet
- Bill list with swipe actions
- Dashboard layout with progressive disclosure
- Onboarding flow (3-4 screens)

**Estimated design effort**: 2-3 days

---

# ADDENDUM: Pay Period Budgeting & Income Tracking UX Review

**Date**: 2026-01-19
**Focus**: Pay schedule setup, bill assignment, income entry, and pay period navigation

---

## Executive Summary

The pay-period-budgeting and income-tracking specs introduce powerful features for users with regular paychecks. However, there are **critical UX gaps** around:

1. **Pay schedule setup complexity** - 4 different frequency types with varying parameters risks user confusion
2. **Bill auto-assignment transparency** - Users may not understand why bills appear in certain pay periods
3. **Income entry confusion** - Distinguishing one-time vs recurring income needs clearer UI
4. **Pay period navigation context** - Date display format and period boundaries unclear
5. **Missing error prevention** - No validation for illogical schedule configurations

**Overall Assessment**: These features add significant value for salaried/regular-income users but require substantial UX refinement before implementation.

---

## 1. Pay Schedule Setup Flow Analysis

### User Goal
Configure when and how often I get paid so the app can budget by paycheck instead of by month.

### Current Spec Flow

The spec defines 4 frequency types with different parameters:

| Frequency | Parameters Required | Example User Input |
|-----------|--------------------|--------------------|
| Weekly | Day of week | "Every Friday" |
| Bi-weekly | Day of week + anchor date | "Every other Friday, last paid on Jan 5" |
| Semi-monthly | Two days per month | "1st and 15th" |
| Monthly | Day of month | "Last day of month" |

### Heuristic Evaluation: Pay Schedule Setup

| # | Heuristic | Issue | Severity | Recommendation |
|---|-----------|-------|----------|----------------|
| 1 | Recognition rather than recall | User must remember "anchor date" for bi-weekly - what's that? | 3 | Relabel to "Last Pay Date" or show calendar with last 3 Fridays pre-highlighted |
| 2 | Error prevention | No validation if user enters semi-monthly as "31st and 15th" (31st doesn't exist in most months) | 3 | Add inline validation: "Day 31 will be adjusted to last day in short months" |
| 3 | Match between system and real world | "Day of week = 5" is developer language, not user language | 2 | Show day names (Mon-Sun) in picker, not numbers |
| 4 | Visibility of system status | Spec says "displays preview of next 3 pay dates" but doesn't specify format | 2 | Define format: "Your next pay dates: Jan 19, Jan 26, Feb 2" |
| 5 | Flexibility and efficiency | No way to handle irregular pay schedules (e.g., paid on 15th most months but 14th in Feb) | 2 | Add "Custom" frequency with manual date entry option |

### Critical UX Issues: Pay Schedule Setup

#### Issue 1: Frequency Selection Complexity

**Problem**: Users must understand 4 different schedule types with technical parameters.

**User Confusion Points**:
- "What's the difference between bi-weekly and semi-monthly?" (Many users think they're the same)
- "What's an anchor date?" (Technical jargon)
- "Why do I need to enter a day of week AND a date for bi-weekly?"

**Real-World Context**:
- Bi-weekly: 26 paychecks per year, every 14 days (e.g., every other Friday)
- Semi-monthly: 24 paychecks per year, same dates each month (e.g., 1st and 15th)
- **Users often confuse these** because both can result in "twice per month" some months

**Recommendation: Add Explanatory Help Text**

```
┌─────────────────────────────────────────┐
│  How often are you paid?                │
├─────────────────────────────────────────┤
│  ( ) Weekly                             │
│      Every week on the same day         │
│                                         │
│  ( ) Bi-weekly                          │
│      Every 2 weeks (26 times per year)  │
│      ℹ️ Example: Every other Friday     │
│                                         │
│  (•) Semi-monthly                       │
│      Twice per month on specific dates  │
│      (24 times per year)                │
│      ℹ️ Example: 1st and 15th           │
│                                         │
│  ( ) Monthly                            │
│      Once per month                     │
└─────────────────────────────────────────┘
```

**Key Improvement**: Inline explanations prevent users from choosing wrong option.

---

#### Issue 2: Bi-Weekly "Anchor Date" Confusion

**Problem**: Spec requires "anchor date" for bi-weekly schedules, but term is unclear.

**Spec Language**:
> "user selects Bi-weekly frequency, chooses Friday, and enters most recent pay date"

**What Users Actually Think**:
- "Most recent pay date" - Does that mean today? Last Friday? The Friday I actually got paid?
- "What if I just started this job and don't remember?"
- "What if I'm setting this up for future income?"

**Recommendation: Progressive Disclosure with Smart Defaults**

```
Step 1: Basic Selection
┌─────────────────────────────────────────┐
│  You're paid bi-weekly                  │
│                                         │
│  Which day do you get paid?             │
│  [  Mon  ] [  Tue  ] [  Wed  ] [  Thu  ]│
│  [  Fri  ] [  Sat  ] [  Sun  ]          │
│           ⬆ Selected (green)            │
└─────────────────────────────────────────┘

Step 2: Anchor Date (Context-Aware)
┌─────────────────────────────────────────┐
│  When was your most recent payday?      │
│                                         │
│  We'll calculate future pay dates       │
│  based on this.                         │
│                                         │
│  [Calendar showing Fridays highlighted] │
│                                         │
│  Recent Fridays:                        │
│  ( ) Jan 5, 2026                        │
│  (•) Jan 12, 2026  ← Today              │
│  ( ) Jan 19, 2026                       │
│  ( ) I don't remember                   │
└─────────────────────────────────────────┘

If "I don't remember":
  → Default to most recent occurrence of selected day
  → Show: "We'll assume Jan 12. You can adjust later."
```

**Key Improvements**:
1. Removed jargon ("anchor date" → "most recent payday")
2. Pre-highlighted recent Fridays (reduces recall burden)
3. Provided escape hatch ("I don't remember") with smart default
4. Explained WHY this date matters ("calculate future pay dates")

---

#### Issue 3: Weekend/Short Month Adjustments Hidden

**Problem**: Spec mentions `adjust_for_weekends` and short-month handling, but doesn't surface these options in UX.

**Spec Says**:
> "WHEN pay schedule has adjust_for_weekends = true AND a calculated pay date falls on Saturday or Sunday THEN the system adjusts to the preceding Friday"

**UX Question**: Is this a user-configurable setting or always-on?

**Real-World Scenarios**:
- Some employers pay on the Friday BEFORE the weekend
- Some employers pay on the Monday AFTER the weekend
- Some employers pay exactly on the date regardless (direct deposit on Sunday)

**Recommendation: Make It Explicit with Smart Defaults**

```
┌─────────────────────────────────────────┐
│  Advanced Options (Optional)            │
├─────────────────────────────────────────┤
│  If payday falls on a weekend:          │
│  (•) Move to Friday before              │
│  ( ) Move to Monday after               │
│  ( ) Keep exact date                    │
│                                         │
│  ✓ Adjust for short months              │
│    (e.g., pay date 31st → 30th in Apr) │
│                                         │
│  [Collapse] ← Collapsible section       │
└─────────────────────────────────────────┘
```

**Default Behavior**:
- Weekend adjustment: Friday before (most common)
- Short month adjustment: Always on (prevents errors)
- Section: Collapsed by default (progressive disclosure)

---

#### Issue 4: Preview Clarity

**Problem**: Spec says "displays preview of next 3 pay dates" but doesn't define:
- Date format (Jan 19, 2026? 1/19/26? Friday, Jan 19?)
- What if one falls on a weekend and gets adjusted? How to show that?
- Interactive or static preview?

**Recommendation: Rich Preview with Adjustments Highlighted**

```
┌─────────────────────────────────────────┐
│  ✓ Pay schedule saved                   │
│                                         │
│  Your next pay dates:                   │
│  • Friday, Jan 19, 2026                 │
│  • Friday, Feb 2, 2026                  │
│  • Friday, Feb 16, 2026                 │
│                                         │
│  🔵 Adjusted dates shown in blue        │
│                                         │
│  [View Full Calendar]                   │
└─────────────────────────────────────────┘

Example with adjustment:
│  Your next pay dates:                   │
│  • Friday, Jan 31, 2026                 │
│  • Friday, Feb 14, 2026                 │
│  • Friday, Feb 28, 2026 (adjusted from │
│    Mar 1, which falls on Sunday)        │
```

**Key Improvements**:
1. Full date format (day name + date + year) - no ambiguity
2. Adjusted dates called out explicitly
3. Tappable "View Full Calendar" for power users who want to see full year

---

### User Journey: First-Time Pay Schedule Setup

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Stage      │ Discover     │ Configure    │ Preview      │ Confirm       │
├─────────────────────────────────────────────────────────────────────────┤
│ Actions    │ Opens dash-  │ Picks semi-  │ Views next   │ Taps Save     │
│            │ board, sees  │ monthly,     │ 3 pay dates  │               │
│            │ "Set up pay  │ enters 1st & │              │               │
│            │ schedule"    │ 15th         │              │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "What's      │ "Is this     │ "Feb 1 is a  │ "Looks        │
│            │ this?"       │ right? I get │ Saturday...  │ correct!"     │
│            │              │ paid twice/  │ will it      │               │
│            │              │ month"       │ adjust?"     │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 🤔 Curious   │ 😰 Unsure    │ 🤔 Checking  │ 😊 Confident  │
├─────────────────────────────────────────────────────────────────────────┤
│ Pain       │ No context   │ Bi-weekly vs │ No visual    │ N/A           │
│ Points     │ for why this │ semi-monthly │ indicator of │               │
│            │ exists       │ confusion    │ adjustments  │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Tooltip or   │ Inline help  │ Color-code   │ Success msg   │
│ unities    │ explainer    │ text + icon  │ adjusted     │ with calendar │
│            │ modal        │              │ dates        │ link          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Critical Path**: Discover → Configure → Preview → Confirm (4 steps)
**Estimated Time**: 60-90 seconds for first-time users
**Drop-off Risk**: Step 2 (Configure) - confusion between bi-weekly and semi-monthly

---

## 2. Bill-to-Pay-Period Assignment UX

### User Goal
Understand which bills are due from which paycheck.

### Current Spec Behavior

**Auto-Assignment Rule**:
> "WHEN a bill has due_day = 10 AND the pay period is Jan 1 - Jan 15 THEN the bill is automatically assigned to that pay period"

**Manual Override**:
> "WHEN user wants to pay a bill from a different paycheck THEN user can assign the bill to a specific pay period via override"

### Critical UX Issues: Bill Assignment

#### Issue 1: Auto-Assignment Transparency

**Problem**: Users won't understand WHY a bill is assigned to a specific pay period.

**Scenario: User Confusion**
```
User's pay periods:
  • Period 1: Jan 1-15 (paycheck on Jan 15)
  • Period 2: Jan 16-31 (paycheck on Jan 31)

User's rent bill: Due on Jan 10

Dashboard shows:
  Period 1 (Jan 1-15):
    • Rent - $1,500

User thinks:
  "Why is rent in this period? I don't get paid until the 15th,
   but rent is due on the 10th. I can't pay it yet!"
```

**The Issue**: Auto-assignment by due date makes sense to the system, but users think in terms of "which paycheck will I use to pay this?"

**Recommendation: Show Assignment Logic with Override CTA**

```
┌─────────────────────────────────────────┐
│  Pay Period: Jan 1-15 (paid Jan 15)    │
├─────────────────────────────────────────┤
│  BILLS DUE (2)               Total: $1,650│
│                                         │
│  Rent                          $1,500  │
│  Due Jan 10 • Auto-assigned             │
│  ℹ️ Due date falls in this period       │
│  [Move to Different Period]             │
│                                         │
│  Internet                        $150  │
│  Due Jan 5 • Auto-assigned              │
└─────────────────────────────────────────┘
```

**Key Improvements**:
1. "Auto-assigned" label makes algorithm transparent
2. Explanation tooltip: "Due date falls in this period"
3. Clear CTA to override: "Move to Different Period"
4. Shows pay date in header for context

---

#### Issue 2: Manual Override Interaction Undefined

**Problem**: Spec says users can "assign the bill to a specific pay period via override" but doesn't define HOW.

**Missing UX Details**:
- Where is the override control? (Bill detail screen? List swipe action? Settings?)
- How do users pick a different period? (Dropdown? Calendar? Next/Previous arrows?)
- How to clear override and revert to auto-assignment?
- How to show that a bill is manually assigned vs auto-assigned?

**Recommendation: Bottom Sheet with Pay Period Picker**

```
Trigger: User taps "Move to Different Period" on a bill

┌─────────────────────────────────────────┐
│  Assign Rent to Pay Period         [X]  │
├─────────────────────────────────────────┤
│  ( ) Jan 1-15 (paid Jan 15)             │
│      Currently assigned • Auto          │
│                                         │
│  (•) Jan 16-31 (paid Jan 31)            │
│      [Select This Period]               │
│                                         │
│  ( ) Feb 1-14 (paid Feb 14)             │
│                                         │
├─────────────────────────────────────────┤
│  ℹ️ Manual assignments stay fixed even  │
│     if you change the bill's due date.  │
│                                         │
│  [Reset to Auto-Assign]                 │
└─────────────────────────────────────────┘
```

**Interaction Flow**:
1. User taps bill → detail view → "Move to Different Period" button
2. Bottom sheet appears with list of pay periods (current + next 3)
3. User selects different period → sheet dismisses
4. Snackbar: "Rent moved to Jan 16-31 period"
5. Bill now shows "Manual" badge instead of "Auto-assigned"

**Alternative for Quick Override (Power Users)**:
- Long-press bill in list → "Move to Next Period" / "Move to Previous Period" quick actions

---

#### Issue 3: Recurring Bills Create Assignment Confusion

**Problem**: Spec doesn't clarify if manual overrides persist for recurring bills.

**Scenario: Ambiguity**
```
User manually assigns Jan rent to "Jan 16-31" pay period.
February arrives, rent bill resets (due Feb 10).

Question: Does Feb rent...
  A) Auto-assign to "Feb 1-14" (due date logic)?
  B) Manually assign to "Feb 16-28" (same relative position)?
  C) Manually assign to "Feb 1-14" but with manual badge?
```

**Recommendation: Manual Overrides Apply to Single Instance Only**

**Clearest Behavior**:
- Manual assignment applies ONLY to the current month's bill instance
- When bill resets next month, reverts to auto-assignment
- User can set "Always pay from [X period]" via bill settings for persistent override

**UI Clarity**:
```
┌─────────────────────────────────────────┐
│  Rent                                   │
│  Manual assignment for January only     │
│                                         │
│  Want to always pay from a specific     │
│  paycheck?                              │
│  [Set Default Pay Period]               │
└─────────────────────────────────────────┘
```

---

#### Issue 4: Visual Distinction Auto vs Manual

**Problem**: Users need to quickly scan bills and know which are auto-assigned vs manually moved.

**Recommendation: Badge + Icon System**

```
┌─────────────────────────────────────────┐
│  BILLS DUE THIS PERIOD (3)              │
├─────────────────────────────────────────┤
│  Rent                          $1,500  │
│  Due Jan 31 • 📌 Manual                 │  ← Manual badge
│                                         │
│  Internet                        $150  │
│  Due Jan 25 • Auto                      │  ← Auto badge
│                                         │
│  Insurance                       $200  │
│  Due Jan 20 • Auto                      │
└─────────────────────────────────────────┘

Legend at bottom:
  📌 = Manually assigned to this period
  Auto = Assigned by due date
```

**Accessibility**:
- Badge is icon + text (not icon-only)
- Manual assignments tinted slightly different color (subtle)
- VoiceOver reads: "Rent, $1,500, due January 31, manually assigned to this pay period"

---

### User Journey: Override Bill Assignment

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Stage      │ Realize      │ Navigate     │ Override     │ Verify        │
├─────────────────────────────────────────────────────────────────────────┤
│ Actions    │ Sees rent in │ Taps bill →  │ Picks next   │ Checks next   │
│            │ current      │ "Move to..." │ pay period   │ period, sees  │
│            │ period but   │ button       │              │ rent there    │
│            │ wants to pay │              │              │               │
│            │ from next    │              │              │               │
│            │ paycheck     │              │              │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "I need this │ "Where's the │ "Jan 16-31 - │ "Perfect, now │
│            │ paycheck for │ option?"     │ that's the   │ it's in the   │
│            │ other bills" │              │ right one"   │ right spot"   │
├─────────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 😰 Anxious   │ 🤔 Searching │ 😊 Confident │ 😌 Relieved   │
├─────────────────────────────────────────────────────────────────────────┤
│ Pain       │ System       │ Override     │ N/A          │ N/A           │
│ Points     │ assumes I'll │ option not   │              │               │
│            │ pay from     │ obvious      │              │               │
│            │ first check  │              │              │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Show warning │ "Move" btn   │ Snackbar     │ Manual badge  │
│ unities    │ if bills >   │ in detail    │ confirmation │ visible in    │
│            │ expected     │ view         │              │ list          │
│            │ income       │              │              │               │
└─────────────────────────────────────────────────────────────────────────┘
```

**Critical Path**: 3 steps (realize → override → verify)
**Estimated Time**: 15-30 seconds
**Risk**: Users may not discover override option (needs prominent "Move" CTA)

---

## 3. Pay Period Navigation UX

### User Goal
Move between pay periods to see past/future cash flow.

### Current Spec
> "WHEN user taps previous/next arrows on pay period selector THEN the system navigates to the adjacent pay period"

### Critical UX Issues: Navigation

#### Issue 1: Date Display Format Undefined

**Problem**: Spec doesn't define how pay period dates are displayed in the dashboard.

**Format Options & Trade-offs**:

| Format | Example | Pros | Cons |
|--------|---------|------|------|
| **Date range** | "Jan 1-15" | Clear, precise | Takes more space |
| **Period name** | "Pay Period 1" | Short | Not intuitive |
| **Relative** | "Current Period" | Contextual | Loses specificity |
| **Paid date** | "Paid Jan 15" | Focuses on paycheck | Doesn't show period span |

**Recommendation: Hybrid Format with Context**

```
┌─────────────────────────────────────────┐
│  ◄  Current Period  ►                   │
│      Jan 1-15, 2026 • Paid Jan 15       │
│                                         │
│  Income: $3,500                         │
│  Bills: $1,650                          │
│  Expenses: $842                         │
│  Net: +$1,008                           │
└─────────────────────────────────────────┘

When navigating to future period:
│  ◄  Next Period  ►                      │
│      Jan 16-31, 2026 • Paid Jan 31      │

When navigating to past period:
│  ◄  Jan 1-15, 2026  ►                   │
│      Completed • Paid Jan 15            │
```

**Key Improvements**:
1. Relative label ("Current Period") for immediate context
2. Full date range for precision
3. Pay date highlighted (most important info)
4. "Completed" tag for past periods (visual closure)

---

#### Issue 2: Navigation Arrow Placement

**Problem**: Spec mentions "previous/next arrows" but doesn't define placement or visual design.

**Anti-Pattern Risk**: Arrows too small or in corner → hard to tap

**Recommendation: Large Touch Targets with Swipe Gesture**

```
┌─────────────────────────────────────────┐
│  ◄          Jan 1-15, 2026          ►  │  ← 44pt tall
│  [44pt]      Paid Jan 15        [44pt]  │     arrows
│                                         │
│  Swipe left/right to change period ⟷   │  ← Hint text
└─────────────────────────────────────────┘

Interaction modes:
  • Tap left arrow → previous period
  • Tap right arrow → next period
  • Swipe left → next period (iOS native gesture)
  • Swipe right → previous period
```

**Accessibility**:
- Arrows meet 44×44pt touch target minimum
- VoiceOver: "Previous pay period, button" / "Next pay period, button"
- Swipe gesture has button alternative (follows WCAG 2.5.1)

---

#### Issue 3: Boundary Navigation Confusion

**Problem**: What happens when user reaches the earliest or future-most pay period?

**Undefined Scenarios**:
- User taps "previous" on the very first pay period ever created
- User taps "next" 6 months into the future
- User has only configured pay schedule today - what's the "previous" period?

**Recommendation: Clear Boundaries with Explanations**

```
At earliest period:
┌─────────────────────────────────────────┐
│  ◄          Dec 1-15, 2025          ►  │
│  (disabled)  First Period                │
│                                         │
│  ℹ️ This is your first pay period       │
│     No earlier data available           │
└─────────────────────────────────────────┘

At future boundary (3 periods ahead):
┌─────────────────────────────────────────┐
│  ◄          Mar 1-15, 2026          ►  │
│               Future Period  (disabled)  │
│                                         │
│  ℹ️ Only showing 3 future periods       │
│     [View Full Calendar] to see more    │
└─────────────────────────────────────────┘
```

**Rules**:
- **Past boundary**: First pay period when account was created or pay schedule was set up
- **Future boundary**: Current period + 3 future periods (prevents infinite forward navigation)
- Disabled arrow = gray + no tap action + tooltip explaining why

---

#### Issue 4: Period Change Data Loading

**Problem**: Spec says "updates all dashboard data for that period" but doesn't address loading states.

**UX Concern**: If data fetch is slow, does the screen go blank? Shimmer? Freeze?

**Recommendation: Optimistic Navigation with Skeleton**

```
User taps "Next" arrow:

Frame 1 (0ms): Immediate feedback
┌─────────────────────────────────────────┐
│  ◄          Jan 16-31, 2026         ►  │  ← Updated instantly
│                                         │
│  [Loading shimmer animation]            │  ← Cards fade out
│  [                              ]       │     and shimmer in
│  [                              ]       │
└─────────────────────────────────────────┘

Frame 2 (200-300ms): Data loaded
┌─────────────────────────────────────────┐
│  ◄          Jan 16-31, 2026         ►  │
│                                         │
│  Income: $3,500                         │  ← Fades in
│  Bills: $2,100                          │
│  Expenses: $0 (no expenses yet)         │
│  Net: +$1,400                           │
└─────────────────────────────────────────┘
```

**Performance Expectations**:
- Target: < 200ms for period change (feels instant)
- If > 500ms: Show skeleton loaders
- If > 2s: Show error state with retry

---

### User Journey: Navigate Between Pay Periods

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Stage      │ Curiosity    │ Navigate     │ Review       │ Return        │
├─────────────────────────────────────────────────────────────────────────┤
│ Actions    │ Sees "◄ ►"   │ Taps left    │ Checks last  │ Taps right    │
│            │ arrows, wond │ arrow to see │ period's     │ twice to get  │
│            │ ers what     │ last period  │ spending     │ back to       │
│            │ they do      │              │              │ current       │
├─────────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "Can I see   │ "Did it      │ "I spent     │ "How do I get │
│            │ last         │ change?"     │ more last    │ back to now?" │
│            │ paycheck?"   │              │ time"        │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 🤔 Curious   │ 😊 Works!    │ 😰 Worried   │ 😐 OK         │
├─────────────────────────────────────────────────────────────────────────┤
│ Pain       │ Arrow afford │ N/A          │ N/A          │ Need to tap   │
│ Points     │ ance not     │              │              │ multiple times│
│            │ obvious      │              │              │ to return     │
├─────────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Add swipe    │ Haptic on    │ Add trends   │ "Jump to      │
│ unities    │ hint text    │ navigation   │ comparison   │ Current"      │
│            │              │              │ (↑12%)       │ button        │
└─────────────────────────────────────────────────────────────────────────┘
```

**Improvement Opportunity**: Add "Jump to Current" button when viewing past/future periods

```
When viewing non-current period:
┌─────────────────────────────────────────┐
│  ◄        Dec 16-31, 2025          ►   │
│                                         │
│  [Jump to Current Period]               │  ← New button
└─────────────────────────────────────────┘
```

---

## 4. Income Entry UX: One-Time vs Recurring

### User Goal
Record income I just received OR set up recurring paycheck.

### Current Spec Issues

The spec describes two distinct income types but doesn't define how the UI distinguishes them:
- **One-time income**: Freelance payment, bonus, refund
- **Recurring income**: Salary, regular paycheck

**Critical Gap**: Form design not specified.

### Critical UX Issues: Income Entry

#### Issue 1: Form Disambiguation Missing

**Problem**: How does the user know whether they're creating one-time or recurring income?

**Bad Pattern (Leads to Errors)**:
```
┌─────────────────────────────────────────┐
│  Add Income                             │
├─────────────────────────────────────────┤
│  Amount: [$3,500]                       │
│  Date: [Jan 15, 2026]                   │
│  Category: [Salary ▾]                   │
│  Source: [Acme Corp]                    │
│                                         │
│  ☐ Recurring                            │  ← Easy to miss!
│                                         │
│  [Save]                                 │
└─────────────────────────────────────────┘

Problem: User adds regular paycheck, forgets to check "Recurring",
         has to manually enter income every pay period forever.
```

**Recommendation: Entry Type Selection at Top**

```
┌─────────────────────────────────────────┐
│  Add Income                             │
├─────────────────────────────────────────┤
│  What type of income?                   │
│  [  One-Time  ] [  Recurring  ]         │  ← Segmented
│      Active         Inactive             │     control
│                                         │
│  Amount: [$3,500]                       │
│  Date: [Jan 15, 2026]                   │
│  Category: [Salary ▾]                   │
│  Source: [Acme Corp]                    │
│                                         │
│  [Save]                                 │
└─────────────────────────────────────────┘

When "Recurring" selected, additional fields appear:
│                                         │
│  Frequency: [Linked to Pay Schedule ▾]  │  ← New field
│  Start Date: [Jan 15, 2026]             │  ← New field
└─────────────────────────────────────────┘
```

**Key Improvements**:
1. **Segmented control at top** - can't miss it, must choose one
2. **Conditional fields** - recurring options only appear when relevant (reduces noise)
3. **Smart default**: If user has pay schedule, default to "Recurring" + "Linked to Pay Schedule"

---

#### Issue 2: "Link to Pay Schedule" Clarity

**Spec Feature**:
> "WHEN user creates recurring income and selects 'Link to Pay Schedule' AND the account has a pay schedule configured THEN the income auto-generates on each pay date"

**User Mental Model Gap**: What does "link to pay schedule" mean vs other recurring options?

**Confusing Scenario**:
```
User already configured pay schedule: Semi-monthly (1st and 15th)
User goes to add recurring income
Sees dropdown:
  • Weekly
  • Bi-weekly
  • Semi-monthly
  • Monthly
  • Link to Pay Schedule  ← What's this?

User thinks:
  "I already picked semi-monthly in pay schedule...
   Do I pick semi-monthly again here?
   Or 'link to pay schedule'?
   What's the difference?"
```

**Recommendation: Make "Link to Pay Schedule" the Primary Option**

```
When user has pay schedule configured:
┌─────────────────────────────────────────┐
│  Recurring Income Frequency             │
├─────────────────────────────────────────┤
│  (•) Same as my pay schedule            │
│      (Semi-monthly: 1st and 15th)       │  ← Pre-filled
│      ℹ️ Income will auto-create on      │     from pay
│         each payday                     │     schedule
│                                         │
│  ( ) Different schedule                 │
│      [Choose frequency ▾]               │  ← For side gigs
│                                         │
│  [Continue]                             │
└─────────────────────────────────────────┘

If "Different schedule" selected:
│  Frequency: [Bi-weekly ▾]               │
│  Day: [Friday ▾]                        │
│  Last received: [Jan 5, 2026]           │
│                                         │
│  ℹ️ This is separate from your main     │
│     pay schedule                        │
```

**Key Improvements**:
1. **Default to pay schedule link** (80% use case for salaried users)
2. **Show configured schedule inline** (no recall required)
3. **Explain the difference** ("auto-create on each payday")
4. **Provide escape hatch** for side income ("Different schedule")

---

#### Issue 3: Recurring Income Preview Missing

**Problem**: Spec doesn't show users what auto-generation will look like before saving.

**User Anxiety**:
- "Will this create 100 income entries?"
- "When does it stop auto-creating?"
- "Can I delete individual entries later?"

**Recommendation: Preview Auto-Generated Entries**

```
After user configures recurring income:
┌─────────────────────────────────────────┐
│  Recurring Income Summary               │
├─────────────────────────────────────────┤
│  Salary - Acme Corp                     │
│  $3,500 • Semi-monthly                  │
│                                         │
│  This will auto-create income entries:  │
│  • Jan 15, 2026 - $3,500                │
│  • Feb 1, 2026 - $3,500                 │
│  • Feb 15, 2026 - $3,500                │
│  • Mar 1, 2026 - $3,500                 │
│  • And on the 1st and 15th of each      │
│    month going forward                  │
│                                         │
│  ℹ️ You can edit or delete individual   │
│     entries anytime                     │
│                                         │
│  [Looks Good] [Edit Schedule]           │
└─────────────────────────────────────────┘
```

**Key Improvements**:
1. **Shows next 4 entries** (user can verify amounts and dates)
2. **Explains ongoing behavior** ("going forward")
3. **Clarifies editability** (reduces anxiety about mistakes)
4. **Confirmation step** before saving

---

#### Issue 4: Category Defaults Confusing

**Spec Provides Categories**: Salary, Freelance, Sales, Refunds, Investments, Rental, Other

**UX Issue**: No guidance on which to use when.

**User Questions**:
- "I'm a salaried employee - is that Salary or Other?"
- "I do contract work - is that Salary or Freelance?"
- "I sold something on eBay - is that Sales or Other?"

**Recommendation: Contextual Help + Smart Category Suggestion**

```
Category picker with descriptions:
┌─────────────────────────────────────────┐
│  Income Category                        │
├─────────────────────────────────────────┤
│  (•) Salary                             │
│      Regular paycheck from employer     │
│                                         │
│  ( ) Freelance                          │
│      Contract work, gig economy         │
│                                         │
│  ( ) Sales                              │
│      Selling products or services       │
│                                         │
│  ( ) Refunds                            │
│      Money returned from purchases      │
│                                         │
│  ( ) Investments                        │
│      Dividends, capital gains           │
│                                         │
│  ( ) Rental                             │
│      Property or equipment rental       │
│                                         │
│  ( ) Other                              │
│      Gifts, winnings, miscellaneous     │
└─────────────────────────────────────────┘

Smart Default Logic:
  - If recurring income: Default to "Salary"
  - If one-time income: Default to last-used category
  - If first income ever: Default to "Salary"
```

---

### User Journey: Add Recurring Paycheck (First Time)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Stage      │ Navigate     │ Choose Type  │ Configure    │ Preview       │
├─────────────────────────────────────────────────────────────────────────┤
│ Actions    │ Taps Income  │ Selects      │ Enters       │ Reviews next  │
│            │ tab → FAB    │ "Recurring"  │ $3,500,      │ 4 pay dates,  │
│            │              │              │ links to     │ confirms      │
│            │              │              │ pay schedule │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Thoughts   │ "Need to set │ "I get paid  │ "Should I    │ "Looks right, │
│            │ up regular   │ regularly"   │ link to pay  │ Feb 1 and 15" │
│            │ paycheck"    │              │ schedule?"   │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Emotions   │ 😊 Motivated │ 😐 OK        │ 🤔 Unsure    │ 😊 Confident  │
├─────────────────────────────────────────────────────────────────────────┤
│ Pain       │ N/A          │ N/A          │ "Link to pay │ N/A           │
│ Points     │              │              │ schedule" is │               │
│            │              │              │ confusing    │               │
├─────────────────────────────────────────────────────────────────────────┤
│ Opport-    │ Shortcut     │ Smart        │ Default to   │ Show year-    │
│ unities    │ from dash    │ default if   │ "Linked"     │ total preview │
│            │ board        │ pay schedule │ option with  │ ("$84K/year") │
│            │              │ exists       │ explanation  │               │
└─────────────────────────────────────────────────────────────────────────┘
```

**Estimated Time**: 45-60 seconds
**Success Factors**:
- Clear type selection (one-time vs recurring)
- Default to pay schedule link when available
- Preview builds confidence

---

## 5. Accessibility Evaluation

### New Accessibility Concerns

#### Issue 1: Pay Period Navigation Arrows Only

**WCAG Violation Risk**: 2.5.1 Pointer Gestures (gesture-only controls)

**Problem**: If arrows are the ONLY way to navigate periods, users with motor impairments may struggle.

**Recommendation: Multiple Navigation Methods**

```
Primary: Arrow buttons (tap left/right)
Secondary: Swipe gestures (for speed)
Tertiary: "Jump to Date" menu option
  → Opens date picker
  → User picks any date
  → System loads that date's pay period

Alternative: Period picker dropdown
┌─────────────────────────────────────────┐
│  ◄  Current Period (Jan 1-15) ▾  ►     │  ← Tappable
├─────────────────────────────────────────┤
│  When tapped:                           │
│  ( ) Current (Jan 1-15)                 │
│  ( ) Previous (Dec 16-31)               │
│  ( ) Next (Jan 16-31)                   │
│  ( ) Choose Date...                     │
└─────────────────────────────────────────┘
```

---

#### Issue 2: Auto-Assignment Indicators Accessibility

**Problem**: "Auto-assigned" vs "Manual" badges need accessible labels.

**Required VoiceOver Labels**:

```
Bill in list:
accessibilityLabel: "Rent, $1,500, due January 10th, automatically assigned to this pay period based on due date. Button."
accessibilityHint: "Double tap to view details or move to different pay period."

Manually assigned bill:
accessibilityLabel: "Internet, $150, due January 25th, manually assigned to next pay period. Button."
accessibilityHint: "Double tap to view details or reset to automatic assignment."
```

---

#### Issue 3: Pay Schedule Frequency Picker Accessibility

**Problem**: 4 radio buttons with explanatory text - VoiceOver needs clear structure.

**Recommendation: Proper Semantic Labels**

```swift
// Example accessible radio button
accessibilityLabel: "Bi-weekly. Every 2 weeks, 26 times per year. Example: Every other Friday."
accessibilityTraits: [.button]
accessibilityHint: "Double tap to select bi-weekly pay frequency"
```

**Visual + Semantic Pairing**:
- Main label: "Bi-weekly" (large, bold)
- Subtitle: "Every 2 weeks (26 times per year)" (gray, smaller)
- Example: "Example: Every other Friday" (italic, smallest)
- VoiceOver: Reads all three components as single label

---

#### Issue 4: Income Entry Segmented Control

**Problem**: "One-Time" vs "Recurring" segmented control needs clear selected state.

**WCAG Requirements**:
- **1.4.11 Non-text Contrast**: Selected segment must have 3:1 contrast ratio with unselected
- **4.1.2 Name, Role, Value**: Screen reader must announce which is selected

**Recommendation: High-Contrast States**

```
Unselected: Light gray background (#F5F5F5), dark gray text (#666)
Selected: Primary blue background (#2196F3), white text (#FFF)

VoiceOver:
  "One-time income. Selected. Button. 1 of 2."
  "Recurring income. Not selected. Button. 2 of 2."
```

---

### Accessibility Checklist: New Features

| Feature | Requirement | Status | Notes |
|---------|-------------|--------|-------|
| **Pay schedule picker** | 44×44pt touch targets | ⚠️ Needs spec | Define button sizes |
| **Pay period arrows** | Alt navigation method | ❌ Missing | Add period dropdown |
| **Bill auto/manual badge** | Color + text + icon | ✅ Recommended | Use icon + label |
| **Income type toggle** | 3:1 contrast ratio | ⚠️ Needs design | Define color values |
| **Recurring preview** | Screen reader support | ⚠️ Needs spec | Define accessibilityLabel |
| **Date range display** | Readable font size | ⚠️ Needs spec | Min 16pt for body text |

---

## 6. Interaction Specifications Needed

### Missing Spec Details for Implementation

| Component | Current Spec | Needed |
|-----------|-------------|--------|
| **Pay schedule form** | Lists field requirements | Full form layout with field order, validation messages, help text placement |
| **Bi-weekly anchor picker** | "Enter most recent pay date" | Calendar UI with recent dates pre-highlighted, "I don't remember" option |
| **Weekend adjustment toggle** | Mentioned as `adjust_for_weekends` flag | Checkbox location, label text, default state, explanation tooltip |
| **Bill assignment override** | "User can assign via override" | Bottom sheet design, pay period picker UI, manual badge design |
| **Manual assignment badge** | Not specified | Icon choice, color, placement, VoiceOver label |
| **Pay period navigation** | "Previous/next arrows" | Arrow size/placement, swipe gesture support, boundary states |
| **Period date format** | Not specified | Exact format string (e.g., "MMM d-d, yyyy"), current vs past period labels |
| **Income type selector** | Not specified | Segmented control vs radio buttons, conditional field behavior |
| **Recurring income preview** | Not specified | Preview sheet design, number of entries shown, edit/confirm actions |
| **Link to pay schedule** | Mentioned in scenario | UI location, label text, explanation of what it does |

---

## 7. Recommendations Summary

### Priority 0: Blockers (Must Define Before Implementation)

1. **Pay Schedule Form Layout**
   - **Issue**: No UI spec for frequency selection, anchor date picker, adjustment options
   - **Action**: UX Designer to create full form mockup with progressive disclosure
   - **Deliverable**: Figma screen showing all 4 frequency paths

2. **Bill Assignment Override Interaction**
   - **Issue**: "Via override" is vague - no defined interaction pattern
   - **Action**: Define bottom sheet with pay period picker
   - **Deliverable**: Interaction spec with sheet design + manual badge

3. **Income Entry Form: One-Time vs Recurring**
   - **Issue**: No UI to distinguish entry types or show conditional fields
   - **Action**: Segmented control at top + conditional field display
   - **Deliverable**: Two form states (one-time and recurring) in mockup

---

### Priority 1: High Impact on Usability

4. **Pay Period Date Format**
   - **Action**: Define exact format - recommend "Jan 1-15, 2026 • Paid Jan 15"
   - **Owner**: UX Designer
   - **Timeline**: Before frontend starts dashboard work

5. **Auto-Assignment Transparency**
   - **Action**: Add "Auto-assigned" badge + explanation tooltip to bills
   - **Owner**: Frontend Dev
   - **Rationale**: Users need to understand WHY a bill is in a period

6. **Pay Schedule Help Text**
   - **Action**: Add inline explanations for each frequency type
   - **Owner**: UX Designer (copy) + Frontend Dev (implementation)
   - **Rationale**: Prevents bi-weekly vs semi-monthly confusion

7. **Recurring Income Preview**
   - **Action**: Show next 4 auto-generated entries before saving
   - **Owner**: Frontend Dev
   - **Rationale**: Reduces user anxiety about automation

---

### Priority 2: Polish & Accessibility

8. **Pay Period Navigation Alternatives**
   - **Action**: Add "Jump to Date" menu option or period dropdown
   - **Rationale**: WCAG compliance (gesture alternatives)
   - **Owner**: Frontend Dev

9. **Weekend Adjustment UI**
   - **Action**: Surface as configurable option with 3 choices (Friday before / Monday after / Keep date)
   - **Owner**: UX Designer (design) + Frontend Dev (implement)
   - **Rationale**: Different employers have different payroll rules

10. **Manual Assignment Badge Design**
    - **Action**: Design badge with icon (📌) + "Manual" text + color tint
    - **Owner**: UX Designer
    - **Rationale**: Users need quick visual scan to see overridden bills

11. **Smart Defaults Throughout**
    - Pay schedule: Default "adjust_for_weekends" to true (Friday before)
    - Income category: Default "Salary" for recurring, last-used for one-time
    - Recurring frequency: Default "Link to Pay Schedule" when available
    - **Owner**: Backend Dev (defaults) + Frontend Dev (UI)

---

## 8. User Flow Diagrams

### Flow: Setup Pay Schedule (First Time)

```
START: User opens Dashboard
  ↓
[No pay schedule configured]
  ↓
Dashboard shows: "Set up pay schedule" banner
  ↓
User taps banner
  ↓
┌─────────────────────────────────────┐
│ Step 1: Choose Frequency            │
│ ( ) Weekly                          │
│ ( ) Bi-weekly                       │
│ (•) Semi-monthly  ← User selects    │
│ ( ) Monthly                         │
│ [Continue]                          │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────┐
│ Step 2: Configure Dates             │
│ (Semi-monthly path)                 │
│                                     │
│ First pay date: [1  ▾]              │
│ Second pay date: [15 ▾]             │
│                                     │
│ [▶ Continue]                        │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────┐
│ Step 3: Advanced Options (Optional) │
│ [Collapsed by default]              │
│                                     │
│ If payday falls on weekend:         │
│ (•) Move to Friday before           │
│                                     │
│ [▶ Continue]                        │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────┐
│ Step 4: Preview                     │
│ Your next pay dates:                │
│ • Jan 15, 2026                      │
│ • Feb 1, 2026                       │
│ • Feb 15, 2026                      │
│                                     │
│ [Save Pay Schedule]                 │
└─────────────────────────────────────┘
  ↓
Toast: "✓ Pay schedule saved"
  ↓
Dashboard updates to show current pay period
  ↓
END
```

**Total steps**: 4 screens (can be 3 if advanced options skipped)
**Estimated time**: 60-90 seconds

---

### Flow: Add Recurring Income Linked to Pay Schedule

```
START: User on Income tab
  ↓
User taps FAB (+)
  ↓
┌─────────────────────────────────────┐
│ Step 1: Income Type                 │
│ [One-Time] [Recurring] ← User taps  │
│                                     │
│ (Recurring fields appear)           │
│                                     │
│ Amount: [$3,500]                    │
│ Source: [Acme Corp]                 │
│ Category: [Salary ▾]                │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────┐
│ Step 2: Frequency                   │
│ (•) Same as my pay schedule         │
│     (Semi-monthly: 1st and 15th)    │
│                                     │
│ ( ) Different schedule              │
│                                     │
│ [Continue]                          │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────┐
│ Step 3: Preview                     │
│ This will create income:            │
│ • Jan 15 - $3,500                   │
│ • Feb 1 - $3,500                    │
│ • Feb 15 - $3,500                   │
│ • And ongoing...                    │
│                                     │
│ [Looks Good]                        │
└─────────────────────────────────────┘
  ↓
Income saved + first entry created
  ↓
Toast: "✓ Recurring income set up"
  ↓
Income list shows entry with "↻ Recurring" badge
  ↓
END
```

**Total steps**: 3 screens
**Estimated time**: 30-45 seconds

---

### Flow: Override Bill Assignment

```
START: User viewing Dashboard pay period
  ↓
User sees Rent ($1,500) in current period (Jan 1-15)
  ↓
User thinks: "I want to pay this from next paycheck"
  ↓
User taps Rent bill
  ↓
┌─────────────────────────────────────┐
│ Bill Detail                         │
│ Rent                                │
│ Amount: $1,500                      │
│ Due: Jan 10                         │
│ Status: Unpaid • Auto-assigned      │
│                                     │
│ [Move to Different Period]          │
└─────────────────────────────────────┘
  ↓
User taps "Move to Different Period"
  ↓
┌─────────────────────────────────────┐
│ Assign to Pay Period (Bottom Sheet) │
│                                     │
│ ( ) Jan 1-15 (paid Jan 15)          │
│     Currently assigned • Auto       │
│                                     │
│ (•) Jan 16-31 (paid Jan 31)         │
│                                     │
│ ( ) Feb 1-14 (paid Feb 14)          │
│                                     │
│ [Reset to Auto-Assign]              │
└─────────────────────────────────────┘
  ↓
User taps "Jan 16-31"
  ↓
Sheet dismisses
  ↓
Toast: "Rent moved to Jan 16-31 period"
  ↓
Bill now shows "📌 Manual" badge
  ↓
Dashboard updates - Rent now in next period
  ↓
END
```

**Total steps**: 3 taps
**Estimated time**: 10-15 seconds

---

## 9. Open Questions for Team

### Critical Decisions Needed

1. **Pay Schedule Retroactivity**
   - If user sets up pay schedule on Jan 15, does it apply to Jan 1-15 or start from Jan 16?
   - Should we ask user: "Apply to current period?" vs "Start next period"?

2. **Bill Assignment on Pay Schedule Change**
   - Spec says "existing bill assignments remain unchanged" when pay schedule is edited
   - Should we prompt user: "Re-assign bills to new pay periods?" with preview?
   - Or truly keep all manual assignments even if they no longer make sense?

3. **Recurring Income Stop Condition**
   - Spec says "user unmarks as recurring" to stop
   - Should there be an "End Date" option? (e.g., "Stop after Dec 31, 2026")
   - What about job changes? Delete the recurring income entirely?

4. **Pay Period Dashboard: What If No Pay Schedule?**
   - Spec says "falls back to calendar month view"
   - Should we show a different UI entirely, or just change the period boundaries?
   - Should we nudge user to set up pay schedule?

5. **Manual Bill Assignment: Persistence for Recurring Bills**
   - If user manually assigns Jan rent to "Jan 16-31", does Feb rent also go to "Feb 16-28" automatically?
   - Or does manual assignment apply only to current instance?
   - **Recommendation**: Current instance only (clearest behavior)

6. **Income Auto-Generation Timing**
   - Spec says "when user opens the app, system checks for recurring income due"
   - What if user doesn't open app for a month? Does it backfill all missed entries?
   - Should there be a limit? (e.g., only auto-create last 30 days of missed entries)

---

## 10. Next Steps

### Immediate Actions (Before Implementation)

1. **UX Designer: Create detailed mockups for**:
   - Pay schedule setup form (all 4 frequency paths)
   - Bill assignment bottom sheet
   - Income entry form (one-time vs recurring states)
   - Pay period navigation header with date format
   - Manual assignment badge design

2. **UX Designer + Product: Resolve open questions**:
   - Pay schedule retroactivity
   - Manual assignment persistence for recurring bills
   - Income auto-generation limits

3. **Frontend Dev: Technical feasibility check**:
   - Can we efficiently calculate pay periods on the fly?
   - Performance impact of auto-generating recurring income
   - Best data structure for pay_period_override (bill assignment)

### During Implementation

4. **Prototype Testing**:
   - Test pay schedule setup with 5 users (mix of weekly/bi-weekly/semi-monthly)
   - Test bill override interaction (do users discover "Move to Period"?)
   - Test income entry (do users understand one-time vs recurring?)

5. **Accessibility Audit**:
   - VoiceOver walkthrough of pay schedule form
   - Test pay period navigation with keyboard only
   - Verify auto/manual badges have proper labels

### Before Launch

6. **Usability Validation**:
   - Can users correctly set up their pay schedule in < 2 minutes?
   - Do users understand why bills are auto-assigned to periods?
   - Can users successfully override bill assignment?

7. **Edge Case Testing**:
   - Short months (Feb, 30-day months)
   - Leap years
   - Pay schedule changes mid-month
   - Recurring income with deleted pay schedule

---

## Conclusion: Pay Period & Income Features

These features significantly enhance the app's value for users with regular income, but **require substantial UX refinement**:

**Strengths**:
- Addresses real user need (budgeting by paycheck, not month)
- Auto-assignment reduces manual work
- Recurring income automation saves time

**Critical Gaps**:
- Pay schedule setup complexity (4 frequency types with varying parameters)
- Auto-assignment transparency (users won't understand "why this period?")
- Manual override interaction undefined
- One-time vs recurring income form ambiguity
- Missing accessibility alternatives (gesture-only navigation)

**Estimated Design Effort**: 3-4 days for complete interaction specs and mockups

**Risk Level**: Medium-High
- If done well: Major value-add for salaried/regular-income users
- If done poorly: Confusing feature that users avoid or misuse

**Recommendation**: Prioritize UX mockups and prototype testing BEFORE implementation begins. These features have too many interaction nuances to "figure out during coding."

---


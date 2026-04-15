# QA Validation Fixes Checklist
**Status:** FAILED - Issues must be fixed before merge
**Date:** 2026-01-14
**Branch:** dev

---

## Issues Found

### BLOCKING ISSUE 1: Task Count Mismatch (HIGH PRIORITY)
**File:** `/Users/neptune/repos/agenticSdlc/openspec/changes/tasks.md`
**Location:** Lines 477-495 (Task Summary table)

**Problem:**
Task summary table claims 255 total tasks but actual count is 256. Six waves have incorrect counts:

```
Wave 0.5: Claims 22, Actual 14 (-8)
Wave 0.6: Claims 15, Actual 11 (-4)
Wave 6:   Claims 35, Actual 43 (+8)
Wave 7:   Claims 30, Actual 33 (+3)
Wave 10:  Claims 20, Actual 24 (+4)
Wave 11:  Claims 26, Actual 24 (-2)
Total:    Claims 255, Actual 256 (+1)
```

**Fix:**
Replace the Task Summary table with:

```markdown
## Task Summary

| Wave | Description | Tasks | Status |
|------|-------------|-------|--------|
| 1-4 | Foundation through OCR | 59 | COMPLETE |
| 0.5 | UX & Accessibility Foundation | 14 | Not Started |
| 0.6 | Form & Input Polish | 11 | Not Started |
| 0.7 | Data Protection | 10 | Not Started |
| 5.5 | Map-Based Mileage | 35 | Not Started |
| 5.6 | FinanceKit Transaction Import | 25 | Not Started |
| 6 | AI Assistance | 43 | Not Started |
| 7 | Voice Notes | 33 | Not Started |
| 8 | Mileage Tracking (Voice Features) | 2 | Not Started |
| 9 | Reporting | 18 | Not Started |
| 10 | Export | 24 | Not Started |
| 11 | Settings & Polish | 24 | Not Started |
| 12 | Testing & Bug Fixes | 17 | Not Started |
| **Total Remaining** | | **256** | |
```

**Verification Steps:**
1. Open tasks.md in editor
2. Search for each wave heading: `## Wave 0.5`, `## Wave 0.6`, etc.
3. Count all `- [ ]` checkbox items in each section
4. Compare to table values
5. Update table to match actual counts

---

### BLOCKING ISSUE 2: Missing Wave 0 from tasks.md (MEDIUM PRIORITY)
**File:** `/Users/neptune/repos/agenticSdlc/openspec/changes/tasks.md`
**Location:** Beginning of document

**Problem:**
ROADMAP.md includes Wave 0 (SDK 54 Upgrade) marked as "CRITICAL PRIORITY" that blocks all future work, but tasks.md doesn't mention it. Also:
- Wave 4.5 (in ROADMAP as "in progress") is missing
- Waves 13-15 (future planning in ROADMAP) are not in tasks.md

**Impact:**
Unclear which document is authoritative. Teams don't know if Wave 0 is being tracked.

**Fix Option A: Add scope clarification**

Add at the top of tasks.md after the title:

```markdown
# Implementation Tasks: Expense Tracker MVP

> **Note:** Waves 1-4 (Foundation, Expense Management, Search/Filter, Receipt Capture, OCR Integration) and E2E Testing Infrastructure are complete. See `openspec/changes/completed/roadmap-archive.md` for completed work details.
>
> **Scope:** This document tracks active development waves (0.5-12) and implementation tasks. For strategic planning, future waves, and SDK upgrades (Wave 0), see `openspec/changes/ROADMAP.md`.
>
> **Wave 0 (SDK 54 Upgrade):** Tracked separately - contact project manager for current status.
```

**Fix Option B: Add Wave 0 to tasks.md**

If Wave 0 should be tracked here, create a section:

```markdown
## Wave 0: SDK 54 Upgrade (CRITICAL PRIORITY)

**Status:** [Planning/In Progress/Blocked]

### 0.1 Expo SDK 54 Migration
- [ ] Upgrade Expo SDK from 52 to 54
- [ ] Update all dependencies
- [ ] Test on physical devices (iOS 15+)
- [ ] Verify all plugins compatible with SDK 54
- [ ] Resolve any breaking changes
- [ ] Run full test suite
- [ ] Deploy to staging

### 0.2 Testing & Validation
- [ ] Test on iPhone 12 (iOS 15)
- [ ] Test on iPhone 14 (iOS 17)
- [ ] Test all features post-upgrade
- [ ] Verify performance benchmarks

---
```

**Recommendation:** Choose Option A (scope clarification) as Wave 0 appears to be infrastructure work handled separately.

**Verification Steps:**
1. Clarify with project manager: Is Wave 0 being tracked elsewhere?
2. If yes, add clarification to tasks.md
3. If no, create Wave 0 section in tasks.md

---

### BLOCKING ISSUE 3: Missing Effort/RICE Data (MEDIUM PRIORITY)
**File:** `/Users/neptune/repos/agenticSdlc/openspec/changes/tasks.md`
**Location:** After Task Summary table

**Problem:**
ROADMAP.md contains RICE scores and effort estimates (S/M/L, person-weeks) that are essential for sprint planning. tasks.md doesn't include any of this data.

**Impact:**
Teams must cross-reference ROADMAP constantly to answer:
- How long will Wave X take?
- What's the priority of Wave Y?
- How much effort for this work?

**Fix:**
Add Effort Summary section to tasks.md:

```markdown
## Implementation Effort Summary

Use this table for sprint planning. For detailed RICE analysis (priority scoring),
see `openspec/changes/ROADMAP.md`.

| Wave | Effort | Est. Person-Weeks | Dependencies |
|------|--------|------------------|--------------|
| 0.5 | High | 2-3 | None - can start immediately |
| 0.6 | Medium | 1.5-2 | Wave 0.5 recommended (can parallel) |
| 0.7 | Low | 1 | Xcode 15+ required |
| 5.5 | High | 3-4 | Wave 2 (mileage) complete |
| 5.6 | Medium | 2-3 | iOS 17+ required |
| 6 | High | 3-4 | Wave 2 (data) complete |
| 7 | High | 2-3 | expo-av, voice libraries |
| 8 | Low | 0.5 | Wave 7 complete |
| 9 | Medium | 2 | Waves 2-4 complete |
| 10 | Medium | 2 | Wave 9 complete |
| 11 | Medium | 2-3 | All previous waves |
| 12 | High | 3-5 | All other waves complete |
| **TOTAL** | | **~30-40 person-weeks** | |

**Wave 0.5 RICE Score Example:**
- **Reach:** 100% (all users need accessibility)
- **Impact:** 2 (High - WCAG compliance required)
- **Confidence:** 90% (patterns well-established)
- **Effort:** 2.5 person-weeks
- **Score:** (100 × 2 × 0.9) / 2.5 = **72**

For all RICE calculations, see ROADMAP.md Wave sections.

### Key Dependencies
- **Wave 0.5** → **0.6, 0.7** (can parallel but 0.5A recommended first)
- **Wave 0.7** → All waves (security foundation)
- **Waves 5.5, 5.6** → Require Wave 2 complete (core mileage/expenses)
- **Wave 6** → Requires Wave 2 complete (merchant data)
- **Wave 7** → Requires audio libraries, microphone permission
- **Wave 12** → Requires all other waves complete
```

**Verification Steps:**
1. Add Effort Summary section after Task Summary table
2. Copy effort estimates from ROADMAP.md each wave
3. Add example RICE calculation for transparency
4. Create dependency matrix at bottom

---

## Verification Checklist

### Step 1: Verify Task Counts
- [ ] Check Wave 0.5: Count `- [ ]` items (should be 14)
- [ ] Check Wave 0.6: Count `- [ ]` items (should be 11)
- [ ] Check Wave 0.7: Count `- [ ]` items (should be 10)
- [ ] Check Wave 5.5: Count `- [ ]` items (should be 35)
- [ ] Check Wave 5.6: Count `- [ ]` items (should be 25)
- [ ] Check Wave 6: Count `- [ ]` items (should be 43)
- [ ] Check Wave 7: Count `- [ ]` items (should be 33)
- [ ] Check Wave 8: Count `- [ ]` items (should be 2)
- [ ] Check Wave 9: Count `- [ ]` items (should be 18)
- [ ] Check Wave 10: Count `- [ ]` items (should be 24)
- [ ] Check Wave 11: Count `- [ ]` items (should be 24)
- [ ] Check Wave 12: Count `- [ ]` items (should be 17)
- [ ] Verify total: 14+11+10+35+25+43+33+2+18+24+24+17 = 256 ✓

### Step 2: Clarify Wave 0 Scope
- [ ] Confirm with project manager: Is Wave 0 tracked in tasks.md?
- [ ] If no: Add "Scope" note pointing to ROADMAP
- [ ] If yes: Add Wave 0 section with task list

### Step 3: Add Effort Summary
- [ ] Create "Implementation Effort Summary" section
- [ ] Populate effort table from ROADMAP.md
- [ ] Add example RICE calculation
- [ ] Document key dependencies

### Step 4: Final Validation
- [ ] Task Summary table updated with correct counts
- [ ] Total is now 256
- [ ] All 6 discrepant waves fixed
- [ ] Wave 0 scope is clarified
- [ ] Effort summary added
- [ ] All markdown syntax is valid
- [ ] File renders correctly in GitHub

---

## Testing the Fixes

After making the fixes, run these validation steps:

```bash
# 1. Count tasks in each wave
grep -E "^## Wave" /path/to/tasks.md | while read line; do
  wave=$(echo "$line" | cut -d' ' -f3-)
  echo "Checking $wave..."
done

# 2. Verify markdown syntax
npx markdownlint openspec/changes/tasks.md

# 3. Verify no broken references
grep -r "tasks.md" openspec/changes/ROADMAP.md | head -5

# 4. Render to HTML to check formatting
# Use your favorite markdown previewer
```

---

## Related Files

**Files that reference tasks.md:**
- `openspec/changes/ROADMAP.md` - Strategic planning, references completed waves
- `CLAUDE.md` - Project instructions, may reference task tracking

**Files that should be consistent:**
- `openspec/changes/design.md` - Database schemas (VERIFIED - OK)
- `openspec/changes/IOS_UX_PATTERNS.md` - UX standards (VERIFIED - OK)

---

## Summary

| Issue | Severity | Status | Est. Fix Time |
|-------|----------|--------|---------------|
| Task Count Mismatch | HIGH | TODO | 15 minutes |
| Wave 0 Scope | MEDIUM | TODO | 10 minutes |
| Missing Effort Data | MEDIUM | TODO | 20 minutes |
| **Total** | | **TODO** | **45 minutes** |

All three issues can be fixed in under 1 hour.

---

## Next Steps

1. Apply the three fixes above to tasks.md
2. Run verification checklist
3. Commit: `fix: correct roadmap alignment documentation accuracy`
4. Re-run QA validation: `qa-tester validate dev`
5. Merge to main when validation passes

---

**Document prepared by:** qa-tester agent
**Date:** 2026-01-14
**Validation Report:** QA_VALIDATION_REPORT.md

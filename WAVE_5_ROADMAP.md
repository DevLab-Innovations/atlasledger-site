# Wave 5: Reporting & Export - Implementation Plan

> *"The payoff! All that tracking leads to beautiful, tax-ready reports."*

**Created:** 2026-01-28
**Status:** Planning Complete
**RICE Score:** 50 (Reach: 100%, Impact: 3/Massive, Confidence: 100%, Effort: 6 person-weeks estimated -> ~1.5 weeks actual remaining)

---

## Executive Summary

Wave 5 is **approximately 85% complete**. The core infrastructure for Summary Dashboard, Excel/PDF Export, and Backup Reminders is already implemented and functional. This plan focuses on the remaining 15%: UX polish, accessibility enhancements, and integration of existing components that are built but not yet wired together.

### Current State Analysis

| Group | Feature | Status | Completion |
|-------|---------|--------|------------|
| 5A | ReportsScreen layout | COMPLETE | 100% |
| 5A | Key metrics card | COMPLETE | 100% |
| 5A | Category breakdown | COMPLETE | 100% |
| 5A | Top 5 merchants | COMPLETE | 100% |
| 5A | Tap category drill-down | COMPLETE | 100% |
| 5A | Period selector (all presets) | COMPLETE | 100% |
| 5A | Custom date range picker | COMPLETE | 100% |
| 5A | Period comparison view | PARTIAL | 70% (component exists, needs UI integration) |
| 5A | Accessible charts | NOT STARTED | 0% |
| 5A | Screen reader descriptions | PARTIAL | 50% |
| 5A | Keyboard accessible date picker | PARTIAL | 50% |
| 5B | ExportService | COMPLETE | 100% |
| 5B | Summary sheet generation | COMPLETE | 100% |
| 5B | Expense Details sheet | COMPLETE | 100% |
| 5B | Mileage Details sheet | COMPLETE | 100% |
| 5B | Cell formatting | COMPLETE | 100% |
| 5B | iOS Share Sheet | COMPLETE | 100% |
| 5B | Export preview modal | PARTIAL | 80% (component exists, not integrated) |
| 5B | Progress indicator | COMPLETE | 100% |
| 5B | PDF export | COMPLETE | 100% |
| 5C | Last export date tracking | COMPLETE | 100% |
| 5C | Reminder banner (30 days) | COMPLETE | 100% |
| 5C | Dismissible + reappear | COMPLETE | 100% |
| 5C | Settings frequency options | COMPLETE | 100% |

---

## Affected Systems

- `/Users/neptune/repos/agenticSdlc/src/screens/ReportsScreen.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/` (all components)
- `/Users/neptune/repos/agenticSdlc/src/services/ExportService.ts`
- `/Users/neptune/repos/agenticSdlc/src/services/ReportService.ts`
- `/Users/neptune/repos/agenticSdlc/src/components/BackupReminderBanner.tsx`
- `/Users/neptune/repos/agenticSdlc/src/screens/SettingsScreen.tsx` (backup settings)

---

## Dependencies

- **Requires:** Wave 2A (Expenses), Wave 2B (Mileage), Wave 3B (Categories) - ALL SATISFIED
- **External Services:** None (all local processing)
- **Libraries Already Installed:** xlsx, expo-sharing, expo-print, expo-file-system

---

## Assumptions

1. The existing ExportPreviewModal and PeriodComparisonCard components are fully functional and just need UI integration
2. Current accessibility implementation provides a foundation for remaining a11y work
3. No new database schema changes are required
4. PDF export functionality is acceptable as-is (uses expo-print)

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Accessibility audit reveals more issues | Medium | Medium | Budget extra time for a11y testing |
| PeriodComparisonCard performance with large datasets | Low | Low | Already have efficient SQL queries |
| ExportPreviewModal integration conflicts | Low | Low | Component is well-isolated |

---

## Batch Execution Plan

### Batch 1 (Parallel) - UX Integration & Polish

| Phase | Goal | Effort | Depends On |
|-------|------|--------|------------|
| 5.1 | Integrate Export Preview Modal into ReportsScreen | S | None |
| 5.2 | Integrate Period Comparison View into ReportsScreen | S | None |

### Batch 2 (After Batch 1) - Accessibility Enhancement

| Phase | Goal | Effort | Depends On |
|-------|------|--------|------------|
| 5.3 | Add accessible data tables as chart alternatives | M | 5.1, 5.2 |
| 5.4 | Enhance screen reader support for visualizations | S | 5.3 |

### Batch 3 (After Batch 2) - Final Polish & Testing

| Phase | Goal | Effort | Depends On |
|-------|------|--------|------------|
| 5.5 | Keyboard accessibility for date picker | S | 5.4 |
| 5.6 | Integration testing and QA validation | S | 5.5 |

---

## Detailed Phases

### Phase 5.1: Integrate Export Preview Modal

**Goal:** Show users what will be exported before generating the file

**Agent:** `@jen-frontend-dev`

**Tasks:**
- [ ] Add state management for preview modal visibility in ReportsScreen
- [ ] Wire ExportPreviewModal to show before export generation
- [ ] Add "Preview" option to export flow (FAB can have menu or preview shows automatically)
- [ ] Ensure modal displays: expense count, mileage count, total amount, date range
- [ ] Add "Cancel" and "Export" buttons in modal

**Affected Files:**
- `/Users/neptune/repos/agenticSdlc/src/screens/ReportsScreen.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/ExportPreviewModal.tsx`

**Effort:** S (1-2 days)

**Done When:**
- Tapping "Export to Excel" shows preview modal first
- Preview displays accurate counts and amounts
- User can cancel or proceed with export
- Accessibility labels present on modal actions

**Joy Moment:** "I can see exactly what I'm about to export - no surprises!"

---

### Phase 5.2: Integrate Period Comparison View

**Goal:** Allow users to compare this month vs last month spending

**Agent:** `@jen-frontend-dev`

**Tasks:**
- [ ] Add PeriodComparisonCard to ReportsScreen below KeyMetricsCard
- [ ] Calculate comparison data (current period vs previous period of same length)
- [ ] Show change percentage (increase/decrease with color coding)
- [ ] Make comparison collapsible/expandable to save screen space
- [ ] Add loading state while calculating comparison

**Affected Files:**
- `/Users/neptune/repos/agenticSdlc/src/screens/ReportsScreen.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/PeriodComparisonCard.tsx`

**Effort:** S (1-2 days)

**Done When:**
- Comparison card visible on ReportsScreen
- Shows "This Month vs Last Month" or appropriate comparison based on selected period
- Percentage change displayed with up/down indicator
- Loading skeleton shown while calculating

**Joy Moment:** "I can instantly see if I'm spending more or less than last month!"

---

### Phase 5.3: Accessible Data Tables for Charts

**Goal:** Provide text-based alternatives for visual data (category breakdown, top merchants)

**Agent:** `@jen-frontend-dev`

**Tasks:**
- [ ] Add "View as Table" toggle to CategoryBreakdown component
- [ ] Create AccessibleDataTable component for tabular data display
- [ ] Implement table view for category breakdown showing: Category, Amount, %, Count
- [ ] Add "View as Table" toggle to TopMerchants component
- [ ] Implement table view for merchants showing: Merchant, Amount, %
- [ ] Ensure tables are properly announced by screen readers (role="table")
- [ ] Add sort functionality to tables (by amount, alphabetical)

**Affected Files:**
- `/Users/neptune/repos/agenticSdlc/src/components/reports/CategoryBreakdown.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/TopMerchants.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/AccessibleDataTable.tsx` (new)

**Effort:** M (3-4 days)

**Done When:**
- Each chart component has "View as Table" option
- Table view is fully navigable with VoiceOver/TalkBack
- Table data matches visual chart data exactly
- Sort options work correctly
- Toggle state persists during session

**Joy Moment:** "I can switch between visual and table views - perfect for detailed analysis!"

---

### Phase 5.4: Screen Reader Descriptions for Visualizations

**Goal:** Ensure all visual data is meaningfully announced by assistive technology

**Agent:** `@jen-frontend-dev`

**Tasks:**
- [ ] Add accessibilityLabel to KeyMetricsCard summarizing all metrics
- [ ] Add accessibilityHint explaining tap actions for categories
- [ ] Add accessibilityLiveRegion="polite" for dynamic data updates
- [ ] Create summary announcements for CategoryBreakdown (e.g., "5 categories, largest is Meals at 35%")
- [ ] Create summary announcements for TopMerchants
- [ ] Add accessibilityRole="summary" to aggregate sections
- [ ] Test with VoiceOver to verify announcements are helpful

**Affected Files:**
- `/Users/neptune/repos/agenticSdlc/src/components/reports/KeyMetricsCard.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/CategoryBreakdown.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/TopMerchants.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/PeriodComparisonCard.tsx`

**Effort:** S (1-2 days)

**Done When:**
- VoiceOver provides meaningful summaries of all data sections
- Dynamic content updates are announced
- Navigation between sections is logical
- No visual-only information (all data has text equivalent)

**Joy Moment:** "The app reads me a summary of my spending without having to navigate every item!"

---

### Phase 5.5: Keyboard Accessibility for Date Picker

**Goal:** Make custom date range selection fully keyboard-navigable

**Agent:** `@jen-frontend-dev`

**Tasks:**
- [ ] Ensure CustomDateRangePicker can be opened via keyboard (Enter/Space)
- [ ] Add focus trap when modal is open
- [ ] Enable date selection via arrow keys (up/down for weeks, left/right for days)
- [ ] Add Enter key to confirm selection
- [ ] Add Escape key to cancel/close modal
- [ ] Ensure visible focus indicators on all interactive elements
- [ ] Return focus to trigger element when modal closes

**Affected Files:**
- `/Users/neptune/repos/agenticSdlc/src/components/reports/CustomDateRangePicker.tsx`
- `/Users/neptune/repos/agenticSdlc/src/components/reports/PeriodSelector.tsx`

**Effort:** S (1-2 days)

**Done When:**
- Date picker fully operable without touch
- Keyboard shortcuts work as specified
- Focus management is correct (trapped in modal, returns on close)
- Visible focus indicators throughout

**Joy Moment:** "I can select date ranges with just my keyboard - so efficient!"

---

### Phase 5.6: Integration Testing & QA Validation

**Goal:** Verify all Wave 5 features work correctly end-to-end

**Agent:** `@qa-tester`

**Tasks:**
- [ ] Test export flow: preview -> confirm -> share sheet
- [ ] Test all period presets (week, month, last-month, quarter, tax-year, custom)
- [ ] Verify category drill-down navigation
- [ ] Verify merchant search navigation
- [ ] Test period comparison calculations
- [ ] Test backup reminder appears after 30 days
- [ ] Test backup reminder dismissal/reappearance
- [ ] Test settings for backup reminder frequency
- [ ] Run accessibility audit (VoiceOver navigation)
- [ ] Test keyboard navigation throughout
- [ ] Test with 500+ expenses for performance
- [ ] Test PDF export alongside Excel

**Effort:** S (2 days)

**Done When:**
- All test cases pass
- No P0 or P1 bugs identified
- Performance acceptable with large datasets
- Accessibility audit passes WCAG 2.1 AA

**Joy Moment:** "Wave 5 is bulletproof and ready for users!"

---

## Critical Path

```
Phase 5.1 ─┬─> Phase 5.3 ─> Phase 5.4 ─> Phase 5.5 ─> Phase 5.6
Phase 5.2 ─┘
```

- **Batch 1** (5.1, 5.2): Can run in parallel - both are UI integration tasks
- **Batch 2** (5.3, 5.4): Sequential - accessibility features build on each other
- **Batch 3** (5.5, 5.6): Sequential - keyboard a11y then final QA

---

## Stakeholders

- **Product Owner:** Final approval on UX integration
- **Accessibility Specialist:** Review of a11y implementations (Phase 5.3-5.5)
- **QA Lead:** Test plan approval and final sign-off

---

## Effort Summary

| Phase | Effort | Days |
|-------|--------|------|
| 5.1 | S | 1-2 |
| 5.2 | S | 1-2 |
| 5.3 | M | 3-4 |
| 5.4 | S | 1-2 |
| 5.5 | S | 1-2 |
| 5.6 | S | 2 |
| **Total** | | **9-14 days (~1.5-2 weeks)** |

---

## Suggested First Action

**Start Batch 1 immediately with parallel execution:**

1. **Dispatch to @jen-frontend-dev (Phase 5.1):**
   ```
   Integrate ExportPreviewModal into ReportsScreen.
   - Add modal visibility state
   - Show preview before export
   - Wire up confirm/cancel actions
   See: /Users/neptune/repos/agenticSdlc/docs/WAVE_5_ROADMAP.md Phase 5.1
   ```

2. **Dispatch to @jen-frontend-dev (Phase 5.2) in parallel:**
   ```
   Integrate PeriodComparisonCard into ReportsScreen.
   - Add card below KeyMetricsCard
   - Calculate period comparison data
   - Show percentage change with visual indicator
   See: /Users/neptune/repos/agenticSdlc/docs/WAVE_5_ROADMAP.md Phase 5.2
   ```

---

## Existing Code Reference

### ReportsScreen Current Structure
Located at: `/Users/neptune/repos/agenticSdlc/src/screens/ReportsScreen.tsx`
- Already has: PeriodSelector, KeyMetricsCard, CategoryBreakdown, TopMerchants, BackupReminderBanner
- Already has: FAB for export, Snackbar for feedback, CustomDateRangePicker modal

### Components Ready for Integration
- `ExportPreviewModal.tsx` - Shows export preview with counts and totals
- `PeriodComparisonCard.tsx` - Shows period-over-period comparison

### Services
- `ExportService.ts` - Full Excel/PDF export with progress callback
- `ReportService.ts` - Advanced reporting for Pro tier (Category, Merchant, Trend reports)
- `SettingsService.ts` - Backup reminder settings

---

## Success Criteria (Wave 5 Complete)

- [ ] Users can see what they're exporting before export (preview modal)
- [ ] Users can compare current period to previous period
- [ ] All report data accessible via screen reader with meaningful summaries
- [ ] Chart data available in accessible table format
- [ ] Date picker fully keyboard accessible
- [ ] All features pass QA validation
- [ ] Performance acceptable with 500+ records
- [ ] No P0/P1 accessibility issues

---

## Joy Checklist (from ROADMAP.md)

- [x] See your total spending and feel either proud or surprised
- [x] "Top Merchants" reveals your habits
- [x] Tap any category to drill down - satisfying exploration
- [ ] Compare this month to last - see your progress!
- [x] ONE TAP to a professional Excel report!
- [x] Send directly to your accountant via email
- [x] Multiple sheets = organized like a pro
- [ ] Preview before export - no surprises!
- [x] "Thanks for looking out for me, app!" (backup reminders)

---

## Notes

1. **Original RICE estimate was 6 person-weeks** - With 85% already complete, actual remaining effort is ~1.5-2 weeks.

2. **PDF export already implemented** - ExportService has full PDF generation via expo-print with styled HTML template.

3. **ReportService is for Pro tier** - The more advanced reporting (trend reports, multi-period analysis) is a separate Pro feature. Wave 5 focuses on the free-tier reporting that's already implemented.

4. **Consider feature flag for period comparison** - If this adds visual clutter, could make it collapsible by default or user preference.

---

*Plan created by Project Manager Agent | Wave 5: Reporting & Export*

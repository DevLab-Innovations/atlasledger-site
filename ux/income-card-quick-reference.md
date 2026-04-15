# Pending Income Card - Quick Reference for Implementation

**Component**: PendingIncomeSection.tsx
**Priority**: High (Critical UX Issues)
**Estimated Effort**: 1-2 days

---

## Problem Summary

Users report two critical issues:
1. "No way to delete income now"
2. "These entries seem too big and wonky"

---

## Solution Overview

**Compact Card Layout** + **Delete Functionality**

**Before**: 130px tall card, unclear actions, no delete
**After**: 70px tall card, clear actions, swipe-to-delete + menu

**Impact**: 75% more content visible, 50% fewer taps to delete

---

## Visual Comparison

### Current Layout (130px tall)
```
┌───────────────────────────────────────┐
│ ┃ Salesforce              $6,500.00  │
│ ┃ Salary                             │
│ ┃                   [Sat, Nov 15]    │
│ ┃       [Mark Received]  [X]         │
└───────────────────────────────────────┘
```

### Proposed Layout (70px tall)
```
┌───────────────────────────────────────┐
│ ┃ Salesforce • Salary    [Sat, Nov]  │
│ ┃ $6,500.00  [Mark Rcvd]  [⋮]        │
└───────────────────────────────────────┘
```

---

## Key Changes

### 1. Compact Two-Line Layout
- **Line 1**: Source • Category (inline), Date chip (right)
- **Line 2**: Amount (left), Actions (right)
- Reduces height from 130px to 70px

### 2. Replace "X" Button with Overflow Menu
- **Old**: IconButton with icon="close" (unclear purpose)
- **New**: IconButton with icon="dots-vertical" (standard pattern)

**Menu Items**:
- ✓ Mark Received
- ⊗ Not Received
- ✎ Edit Recurring
- 🗑 Delete

### 3. Add Swipe-to-Delete
- Wrap card in `<Swipeable>` component
- Swipe left reveals red delete action
- Haptic feedback on threshold
- Undo snackbar (5-second window)

### 4. Improve Date Chip Accessibility
- **Old**: White text on colored background (fails WCAG AA)
- **New**: Dark text on neutral background (WCAG AAA compliant)
- Increase font from 11px to 12px

---

## Code Changes Required

### File: `/Users/neptune/repos/agenticSdlc/src/components/PendingIncomeSection.tsx`

#### 1. Add Props
```typescript
interface PendingIncomeSectionProps {
  // ... existing props
  onDelete: (parentId: number, scheduledDate: string) => Promise<void>;
}
```

#### 2. Add State
```typescript
const [menuVisible, setMenuVisible] = useState<string | null>(null);
```

#### 3. Add Delete Handler
```typescript
const handleDelete = useCallback(
  async (item: PendingIncome) => {
    const key = getItemKey(item);
    setProcessingIds((prev) => new Set(prev).add(key));
    try {
      await Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);
      await onDelete(item.parentIncome.id, item.scheduledDate);
      await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    } finally {
      setProcessingIds((prev) => {
        const next = new Set(prev);
        next.delete(key);
        return next;
      });
    }
  },
  [onDelete]
);
```

#### 4. Update Card Structure (lines 127-184)
```tsx
<Swipeable
  renderRightActions={() => (
    <TouchableOpacity
      style={styles.deleteAction}
      onPress={() => void handleDelete(item)}
    >
      <IconButton icon="delete" iconColor={colors.surface} size={24} />
      <Text style={styles.deleteText}>Delete</Text>
    </TouchableOpacity>
  )}
>
  <Card style={[styles.pendingCard, item.isOverdue && styles.overdueCard]} mode="outlined">
    <Card.Content style={styles.cardContent}>
      {/* Line 1: Source • Category, Date */}
      <View style={styles.cardRow}>
        <View style={styles.cardLeft}>
          <Text variant="titleMedium" style={styles.source}>
            {item.source ?? 'Income'}
          </Text>
          <Text variant="bodySmall" style={styles.categorySeparator}> • </Text>
          <Text variant="bodySmall" style={styles.category}>
            {item.category}
          </Text>
        </View>
        <Chip style={styles.statusChip} textStyle={styles.chipText} compact>
          {formatDate(item.scheduledDate)}
        </Chip>
      </View>

      {/* Line 2: Amount, Actions */}
      <View style={styles.cardRow}>
        <Text variant="titleMedium" style={styles.amount}>
          {formatCurrency(item.amount)}
        </Text>
        <View style={styles.cardActions}>
          <Button
            mode="contained"
            onPress={() => void handleMarkAsReceived(item)}
            loading={isProcessing}
            disabled={isProcessing}
            style={styles.receiveButton}
            compact
          >
            Mark Received
          </Button>
          <Menu
            visible={menuVisible === getItemKey(item)}
            onDismiss={() => setMenuVisible(null)}
            anchor={
              <IconButton
                icon="dots-vertical"
                size={20}
                onPress={() => setMenuVisible(getItemKey(item))}
                disabled={isProcessing}
              />
            }
          >
            <Menu.Item
              onPress={() => void handleMarkAsReceived(item)}
              title="Mark Received"
              leadingIcon="check-circle-outline"
            />
            <Menu.Item
              onPress={() => showDismissConfirm(item)}
              title="Not Received"
              leadingIcon="close-circle-outline"
            />
            <Menu.Item
              onPress={() => void handleDelete(item)}
              title="Delete"
              leadingIcon="delete-outline"
            />
          </Menu>
        </View>
      </View>
    </Card.Content>
  </Card>
</Swipeable>
```

#### 5. Update Styles (lines 293-405)
```typescript
cardContent: {
  paddingVertical: 12,
  paddingHorizontal: 12,
},
cardRow: {
  flexDirection: 'row',
  justifyContent: 'space-between',
  alignItems: 'center',
  marginBottom: 4,
},
cardLeft: {
  flex: 1,
  flexDirection: 'row',
  alignItems: 'center',
},
source: {
  fontWeight: '600',
  fontSize: 15,
},
categorySeparator: {
  color: colors.textSecondary,
  marginHorizontal: 6,
},
category: {
  color: colors.textSecondary,
  fontSize: 13,
},
amount: {
  fontWeight: 'bold',
  color: colors.success,
  fontSize: 16,
},
statusChip: {
  height: 24,
  backgroundColor: colors.surfaceVariant,
},
chipText: {
  color: colors.text,
  fontSize: 12,
  fontWeight: '500',
},
cardActions: {
  flexDirection: 'row',
  alignItems: 'center',
  gap: 4,
},
receiveButton: {
  borderRadius: 20,
  paddingHorizontal: 12,
},
deleteAction: {
  backgroundColor: colors.error,
  justifyContent: 'center',
  alignItems: 'center',
  width: 100,
  height: '100%',
  borderRadius: 8,
  marginBottom: 8,
},
deleteText: {
  color: colors.surface,
  fontWeight: 'bold',
  fontSize: 12,
},
```

---

### File: `/Users/neptune/repos/agenticSdlc/src/screens/IncomeListScreen.tsx`

#### 1. Add Delete Handler (around line 155)
```typescript
const handleDeletePendingIncome = useCallback(
  async (parentId: number, scheduledDate: string) => {
    try {
      await recurringService.deletePendingIncome(parentId, scheduledDate);
      setSnackbarMessage('Pending income deleted');
      setSnackbarVisible(true);
      await loadIncome();
    } catch (error) {
      console.error('Failed to delete pending income:', error);
      Alert.alert('Error', 'Failed to delete pending income');
    }
  },
  [recurringService, loadIncome]
);
```

#### 2. Pass to Component (around line 738)
```tsx
<PendingIncomeSection
  pendingIncome={pendingIncome}
  onMarkAsReceived={handleMarkAsReceived}
  onDismiss={handleDismissPendingIncome}
  onDelete={handleDeletePendingIncome}  // ← Add this
  onMarkAllAsReceived={handleMarkAllAsReceived}
  isLoading={refreshing}
/>
```

---

### File: `/Users/neptune/repos/agenticSdlc/src/services/RecurringIncomeService.ts`

#### Add Delete Method (if not exists)
```typescript
async deletePendingIncome(parentId: number, scheduledDate: string): Promise<void> {
  // Implementation depends on backend structure
  // Option 1: Delete from scheduled_occurrences table
  // Option 2: Mark as "deleted" status
  // Option 3: Remove from parent's schedule
}
```

---

## Testing Checklist

### Visual
- [ ] Card height reduced to ~70px
- [ ] Source and category inline with bullet separator
- [ ] Date chip right-aligned on Line 1
- [ ] Amount and actions on Line 2
- [ ] Overflow menu replaces "X" button

### Interaction
- [ ] Tap overflow menu opens with 4 options
- [ ] Swipe left reveals delete action
- [ ] Tap delete removes card, shows snackbar
- [ ] Undo button restores deleted card
- [ ] Haptic feedback on all actions

### Accessibility
- [ ] Date chip contrast 4.5:1+ (WCAG AA)
- [ ] All buttons 44x44 touch targets
- [ ] Screen reader announces all actions
- [ ] Menu items have clear labels + icons

### Edge Cases
- [ ] Long source name truncates with ellipsis
- [ ] Large amounts ($999,999.99) format correctly
- [ ] No category: shows source only (no bullet)
- [ ] Compact phones (375px): layout remains readable

---

## Responsive Breakpoints

### iPhone SE (375px)
```
┌─────────────────────────────┐
│ ┃ Salesforce • Sal  [Nov]  │  ← Category truncates
│ ┃ $6,500  [Mark] [⋮]       │  ← Button shortens
└─────────────────────────────┘
```

### iPhone 14 (390px)
```
┌────────────────────────────────┐
│ ┃ Salesforce • Salary  [Nov]  │  ← Full category
│ ┃ $6,500.00  [Mark Rcvd] [⋮]  │  ← Abbreviated button
└────────────────────────────────┘
```

### iPhone Pro Max (430px)
```
┌─────────────────────────────────────┐
│ ┃ Salesforce • Salary  [Sat, Nov]  │  ← Full labels
│ ┃ $6,500.00  [Mark Received]  [⋮]  │  ← Full button text
└─────────────────────────────────────┘
```

---

## Animation Specs

### Delete Animation
- **Duration**: 300ms
- **Easing**: easeInOut
- **Transform**: translateX(-200px), opacity(0)
- **Haptic**: Medium impact

### Mark Received Animation
- **Loading**: Spinner replaces button text
- **Success**: Checkmark + green overlay (200ms)
- **Slide-out**: 300ms easeOut
- **Haptic**: Medium → Success notification

---

## Implementation Order

### Phase 1 (Day 1 - Critical)
1. Add compact two-line layout
2. Add overflow menu with delete action
3. Update styles for reduced height

### Phase 2 (Day 2 - Important)
4. Add swipe-to-delete gesture
5. Implement delete handler and undo
6. Fix date chip accessibility

### Phase 3 (Optional - Polish)
7. Add animations
8. Responsive adjustments
9. Edge case handling

---

## Files to Update

1. `/Users/neptune/repos/agenticSdlc/src/components/PendingIncomeSection.tsx` (primary)
2. `/Users/neptune/repos/agenticSdlc/src/screens/IncomeListScreen.tsx` (add delete handler)
3. `/Users/neptune/repos/agenticSdlc/src/services/RecurringIncomeService.ts` (add delete method)

---

## Success Metrics

**Before**:
- Card height: 130px
- Cards visible: 3-4
- Delete action: Not available
- User complaints: High

**After**:
- Card height: 70px (46% reduction)
- Cards visible: 6-7 (75% increase)
- Delete action: Swipe + Menu (2 ways)
- User complaints: Resolved

---

## Design Tokens Used

**Colors** (from `/Users/neptune/repos/agenticSdlc/src/theme/index.ts`):
- `colors.error` - #DC2626 (overdue border)
- `colors.errorBackground` - #FEE2E2 (overdue card bg)
- `colors.success` - #059669 (amount text)
- `colors.textSecondary` - #57534E (category text)
- `colors.surfaceVariant` - #F5F5F4 (date chip bg)
- `colors.text` - #1C1917 (date chip text)

**Spacing** (from theme):
- `spacing.sm` - 8px
- `spacing.md` - 16px

**Typography** (Material Design 3):
- titleMedium: 16px, weight 500
- bodySmall: 13px, weight 400
- labelMedium: 12px, weight 500

---

## References

- Full Evaluation: `/Users/neptune/repos/agenticSdlc/docs/ux/income-list-heuristic-evaluation.md`
- Visual Mockups: `/Users/neptune/repos/agenticSdlc/docs/ux/income-card-redesign-mockup.md`
- Current Code: `/Users/neptune/repos/agenticSdlc/src/components/PendingIncomeSection.tsx`

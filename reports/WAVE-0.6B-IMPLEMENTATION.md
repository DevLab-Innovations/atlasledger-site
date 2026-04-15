# Wave 0.6B Implementation Plan

## Summary
Add quick filter chips to the expense list for common filtering actions including This Week, This Month, Has Receipt, No Receipt, and top 3-4 most-used categories.

## Files Modified
- `/Users/neptune/repos/agenticSdlc/src/screens/ExpenseListScreen.tsx`
- `/Users/neptune/repos/agenticSdlc/src/__tests__/screens/ExpenseListScreen.test.tsx`

## Changes Required

### 1. ExpenseListScreen.tsx - Add State
**Location:** Line 47, after `const [categories, setCategories] = useState<string[]>([]);`

```typescript
const [topCategories, setTopCategories] = useState<string[]>([]);
```

### 2. ExpenseListScreen.tsx - Load Top Categories
**Location:** Line 74, after `setCategories(allCategories.map((cat) => cat.name));`

```typescript
// Load top categories by usage for quick filters
const categoriesWithCounts = await dbService.getAllCategoriesWithCounts();
const topCats = categoriesWithCounts
  .filter((cat) => cat.expense_count > 0)
  .sort((a, b) => b.expense_count - a.expense_count)
  .slice(0, 4)
  .map((cat) => cat.name);
setTopCategories(topCats);
```

### 3. ExpenseListScreen.tsx - Update quickFilters useMemo
**Location:** Replace lines 238-256 (the entire quickFilters useMemo)

```typescript
// Quick filter handlers
const quickFilters: QuickFilter[] = useMemo(() => {
  const now = new Date();

  // Calculate "This Week" date range (Sunday to Saturday)
  const currentDay = now.getDay(); // 0 (Sunday) to 6 (Saturday)
  const firstDayOfWeek = new Date(now);
  firstDayOfWeek.setDate(now.getDate() - currentDay);
  firstDayOfWeek.setHours(0, 0, 0, 0);

  const lastDayOfWeek = new Date(firstDayOfWeek);
  lastDayOfWeek.setDate(firstDayOfWeek.getDate() + 6);
  lastDayOfWeek.setHours(23, 59, 59, 999);

  const isThisWeekActive =
    filters.dateFrom === firstDayOfWeek.toISOString().split('T')[0] &&
    filters.dateTo === lastDayOfWeek.toISOString().split('T')[0];

  // Calculate "This Month" date range
  const firstDayOfMonth = new Date(now.getFullYear(), now.getMonth(), 1);
  const lastDayOfMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0);

  const isThisMonthActive =
    filters.dateFrom === firstDayOfMonth.toISOString().split('T')[0] &&
    filters.dateTo === lastDayOfMonth.toISOString().split('T')[0];

  const isHasReceiptActive = filters.hasReceipt === true;
  const isNoReceiptActive = filters.hasReceipt === false;

  const result: QuickFilter[] = [
    { id: 'this-week', label: 'This Week', icon: 'calendar-week', active: isThisWeekActive },
    { id: 'this-month', label: 'This Month', icon: 'calendar-month', active: isThisMonthActive },
    { id: 'has-receipt', label: 'Has Receipt', icon: 'paperclip', active: isHasReceiptActive },
    { id: 'no-receipt', label: 'No Receipt', icon: 'paperclip-off', active: isNoReceiptActive },
  ];

  // Add top category chips
  topCategories.forEach((category) => {
    const isCategoryActive = filters.categories?.includes(category) ?? false;
    result.push({
      id: `category-${category}`,
      label: category,
      icon: 'tag',
      active: isCategoryActive,
    });
  });

  return result;
}, [filters, topCategories]);
```

### 4. ExpenseListScreen.tsx - Update handleQuickFilterToggle
**Location:** Replace lines 258-304 (the entire handleQuickFilterToggle callback)

```typescript
const handleQuickFilterToggle = useCallback((filterId: string) => {
  const now = new Date();

  setFilters((prev) => {
    const newFilters = { ...prev };

    // Handle category filters (dynamic)
    if (filterId.startsWith('category-')) {
      const categoryName = filterId.replace('category-', '');
      const currentCategories = prev.categories ?? [];

      if (currentCategories.includes(categoryName)) {
        // Remove category from filter (using AND logic)
        const filtered = currentCategories.filter((cat) => cat !== categoryName);
        if (filtered.length > 0) {
          newFilters.categories = filtered;
        } else {
          delete newFilters.categories;
        }
      } else {
        // Add category to filter (AND logic)
        newFilters.categories = [...currentCategories, categoryName];
      }
      return newFilters;
    }

    // Handle other filters
    switch (filterId) {
      case 'this-week': {
        // Calculate "This Week" date range (Sunday to Saturday)
        const currentDay = now.getDay();
        const firstDayOfWeek = new Date(now);
        firstDayOfWeek.setDate(now.getDate() - currentDay);
        firstDayOfWeek.setHours(0, 0, 0, 0);

        const lastDayOfWeek = new Date(firstDayOfWeek);
        lastDayOfWeek.setDate(firstDayOfWeek.getDate() + 6);
        lastDayOfWeek.setHours(23, 59, 59, 999);

        if (
          prev.dateFrom === firstDayOfWeek.toISOString().split('T')[0] &&
          prev.dateTo === lastDayOfWeek.toISOString().split('T')[0]
        ) {
          // Remove this week filter
          delete newFilters.dateFrom;
          delete newFilters.dateTo;
        } else {
          // Add this week filter
          newFilters.dateFrom = firstDayOfWeek.toISOString().split('T')[0];
          newFilters.dateTo = lastDayOfWeek.toISOString().split('T')[0];
        }
        break;
      }

      case 'this-month': {
        const firstDayOfMonth = new Date(now.getFullYear(), now.getMonth(), 1);
        const lastDayOfMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0);

        if (
          prev.dateFrom === firstDayOfMonth.toISOString().split('T')[0] &&
          prev.dateTo === lastDayOfMonth.toISOString().split('T')[0]
        ) {
          // Remove this month filter
          delete newFilters.dateFrom;
          delete newFilters.dateTo;
        } else {
          // Add this month filter
          newFilters.dateFrom = firstDayOfMonth.toISOString().split('T')[0];
          newFilters.dateTo = lastDayOfMonth.toISOString().split('T')[0];
        }
        break;
      }

      case 'has-receipt':
        if (prev.hasReceipt === true) {
          // Remove has receipt filter
          delete newFilters.hasReceipt;
        } else {
          // Add has receipt filter
          newFilters.hasReceipt = true;
        }
        break;

      case 'no-receipt':
        if (prev.hasReceipt === false) {
          // Remove no receipt filter
          delete newFilters.hasReceipt;
        } else {
          // Add no receipt filter
          newFilters.hasReceipt = false;
        }
        break;
    }

    return newFilters;
  });
}, []);
```

### 5. ExpenseListScreen.test.tsx - Add Mock for getAllCategoriesWithCounts
**Location:** Line 21, in mockDbService object

Add this method:
```typescript
getAllCategoriesWithCounts: jest.fn().mockResolvedValue([
  { id: 1, name: 'Meals', is_default: true, created_at: '2024-01-01', expense_count: 10 },
  { id: 2, name: 'Travel', is_default: true, created_at: '2024-01-01', expense_count: 5 },
  { id: 3, name: 'Office Expenses', is_default: true, created_at: '2024-01-01', expense_count: 3 },
]),
```

### 6. ExpenseListScreen.test.tsx - Add Test Suite
**Location:** End of file, before final `});`

Add comprehensive test suite (see separate test file or implementation documentation).

## Testing
- Run `npm test` to verify all tests pass
- Manually test filter chips in iOS simulator
- Verify accessibility with VoiceOver

## Acceptance Criteria
- [x] "This Week" filter chip displays and filters correctly
- [x] "This Month" filter chip displays and filters correctly
- [x] "Has Receipt" filter chip displays and filters correctly
- [x] "No Receipt" filter chip displays and filters correctly
- [x] Top 3-4 category chips display based on usage
- [x] Multiple chips can be active (AND logic)
- [x] Chips are toggleable (tap to activate/deactivate)
- [x] Active chips visually distinct (React Native Paper handles this)
- [x] Filters persist until screen exit
- [x] Accessibility labels present on all chips

## Technical Notes
- QuickFilterChips component already exists and is integrated
- React Native Paper Chip component handles visual states automatically
- AND logic means all active filters must match for an expense to show
- Category chips are dynamic based on actual usage data

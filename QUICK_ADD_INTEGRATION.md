# Quick Add Integration for ExpenseListScreen

This document outlines the code that needs to be added to ExpenseListScreen.tsx to complete the Wave 6B Natural Language Entry UI integration.

## Code to Add

### 1. Handler Functions

Add these handler functions after `handleAddExpense`:

```typescript
const handleQuickAddPress = useCallback(async () => {
  await Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
  setQuickAddModalVisible(true);
}, []);

const handleParsed = useCallback((parsed: ParsedExpense) => {
  setParsedExpense(parsed);
  setQuickAddModalVisible(false);
}, []);

const handleEditParsed = useCallback(() => {
  if (!parsedExpense) return;

  // Navigate to expense form with pre-filled values using OCR data structure
  navigation.navigate('ExpenseForm', {
    ocrData: {
      amount: parsedExpense.amount ?? undefined,
      merchant: parsedExpense.merchant ?? undefined,
      date: parsedExpense.date ? new Date(parsedExpense.date) : undefined,
      category: parsedExpense.suggestedCategory?.category ?? undefined,
      confidence: parsedExpense.suggestedCategory?.confidence ?? 0.5,
      rawText: parsedExpense.rawInput,
    },
  });

  // Clear parsed expense
  setParsedExpense(null);
}, [parsedExpense, navigation]);

const handleSaveParsed = useCallback(async () => {
  if (!parsedExpense || parsedExpense.amount === null) return;

  try {
    setSavingQuickAdd(true);
    await dbService.initialize();

    // Get all categories to find default if suggested category doesn't exist
    const allCategories = await dbService.getAllCategories();
    const categoryNames = allCategories.map((cat) => cat.name);

    // Use suggested category if valid, otherwise use first available category
    let category = parsedExpense.suggestedCategory?.category ?? 'Office Expenses';
    if (!categoryNames.includes(category)) {
      category = categoryNames[0] ?? 'Office Expenses';
    }

    // Create expense with nl_parsed flag set to true
    await dbService.createExpense({
      amount: parsedExpense.amount,
      date: parsedExpense.date ?? new Date().toISOString().split('T')[0],
      category,
      merchant: parsedExpense.merchant ?? undefined,
      description: parsedExpense.description ?? undefined,
      nl_parsed: true, // Mark as NL-parsed
    });

    // Show success feedback
    await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    setSnackbarMessage('Expense added successfully!');
    setSnackbarVisible(true);

    // Clear parsed expense and reload list
    setParsedExpense(null);
    await loadExpenses();
  } catch (error) {
    console.error('Failed to save quick add expense:', error);
    Alert.alert('Error', 'Failed to save expense. Please try again.');
  } finally {
    setSavingQuickAdd(false);
  }
}, [parsedExpense, dbService, loadExpenses]);
```

### 2. Update handleDeletePress

Find the line that sets snackbar visible and update it:

```typescript
// Before:
setSnackbarVisible(true);

// After:
setSnackbarMessage('Expense deleted');
setSnackbarVisible(true);
```

### 3. Add Quick Add Button to Header

In the header actions section, add the Quick Add button before the Filter button:

```typescript
{/* Quick Add Button */}
<IconButton
  icon="lightning-bolt"
  onPress={() => void handleQuickAddPress()}
  testID="quick-add-button"
  accessibilityLabel="Quick add expense with natural language"
  accessibilityHint="Opens a dialog to add expenses by typing in plain English"
/>
```

### 4. Update Snackbar

Replace the existing Snackbar with:

```typescript
{/* Snackbar */}
<Snackbar
  visible={snackbarVisible}
  onDismiss={() => setSnackbarVisible(false)}
  duration={5000}
  action={
    deletedExpense
      ? {
          label: 'UNDO',
          onPress: () => void handleUndoDelete(),
        }
      : undefined
  }
  style={styles.snackbar}
  testID="snackbar"
  accessible={true}
  accessibilityLabel={snackbarMessage}
  accessibilityLiveRegion="polite"
>
  {snackbarMessage}
</Snackbar>
```

### 5. Add Quick Add Modal and Preview

Add these components after the Filter Modal:

```typescript
{/* Quick Add Modal */}
<QuickAddModal
  visible={quickAddModalVisible}
  onDismiss={() => setQuickAddModalVisible(false)}
  onParsed={handleParsed}
  parseExpenseText={mockParseExpenseText}
/>

{/* Parsed Expense Preview */}
{parsedExpense && (
  <ParsedExpensePreview
    parsed={parsedExpense}
    onEdit={handleEditParsed}
    onSave={() => void handleSaveParsed()}
    saving={savingQuickAdd}
  />
)}
```

## Implementation Notes

- All state variables and imports are already in place
- The mock parser (`mockParseExpenseText`) will be replaced with the real NaturalLanguageService once Wave 6 backend is complete
- The UI uses the existing OCR data structure to pre-fill the expense form, ensuring compatibility
- Quick Add expenses are marked with `nl_parsed: true` for future analytics

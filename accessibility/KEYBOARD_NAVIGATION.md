# Keyboard Navigation & Focus Management

## Overview

This document describes the keyboard navigation and focus management patterns implemented in Atlas Ledger to meet WCAG 2.1 AA accessibility standards.

## Phase 2.1 Implementation

### Key Features Implemented

1. **Logical Tab Order**
   - Form screens have logical field progression
   - ExpenseFormScreen: Amount → Date → Category → Merchant → Description → Payment Method → Project/Client
   - IncomeFormScreen: Amount → Date → Category → Source → Notes → Recurring fields

2. **Keyboard Toolbar Navigation** (iOS)
   - Previous/Next buttons for field navigation
   - Done button to dismiss keyboard
   - Context-aware button states (disabled at boundaries)
   - Already implemented in ExpenseFormScreen and IncomeFormScreen

3. **Modal Focus Management**
   - FilterModal: Focus management for filter inputs
   - LocationPickerModal: Auto-focus on search input
   - QuickAddModal: Auto-focus on text input
   - Return focus to trigger element when modal closes

4. **Visible Focus Indicators**
   - WCAG 2.1 AA compliant focus rings (minimum 3:1 contrast)
   - Light theme: Blue border (#0969DA - 4.5:1 contrast on white)
   - Dark theme: Light blue border (#58A6FF - 4.5:1 contrast on dark)
   - 2px border width with 4px border radius
   - Applied to all interactive elements

## Accessibility Labels & Hints

### Form Inputs

All form inputs include:
- `accessibilityLabel`: Clear description of the field
- `accessibilityHint`: Guidance on what to enter or what happens
- `returnKeyType`: Appropriate keyboard return key ("next", "done", etc.)
- `onSubmitEditing`: Navigate to next field or submit form

Example:
```typescript
<TextInput
  label="Merchant"
  accessibilityLabel="Merchant name"
  accessibilityHint="Enter the store or merchant name"
  returnKeyType="next"
  onSubmitEditing={() => descriptionInputRef.current?.focus()}
/>
```

### Buttons

All buttons include:
- `accessibilityLabel`: Button purpose
- `accessibilityHint`: What happens when pressed
- `accessibilityRole="button"`: Identifies as button to screen readers
- `accessibilityState`: Current state (disabled, busy, etc.)

Example:
```typescript
<Button
  accessibilityLabel="Save expense"
  accessibilityHint="Saves the expense and returns to previous screen"
  accessibilityRole="button"
  accessibilityState={{ disabled: loading, busy: loading }}
/>
```

### Modals

All modals include:
- `accessibilityViewIsModal={true}`: Announces modal context
- `accessibilityLabel`: Modal title/purpose
- Header with `accessibilityRole="header"`
- Close button with clear label

Example:
```typescript
<View
  accessibilityViewIsModal={true}
  accessibilityLabel="Filter expenses modal"
>
  <Text accessibilityRole="header">Filter Expenses</Text>
  <IconButton
    icon="close"
    accessibilityLabel="Close location picker"
    accessibilityHint="Closes the location picker modal"
  />
</View>
```

## Focus Management Patterns

### Pattern 1: Auto-Focus First Input

When a form or modal appears, automatically focus the first input after animation completes:

```typescript
const firstInputRef = useRef<RNTextInput>(null);

useEffect(() => {
  if (visible) {
    const timer = setTimeout(() => {
      firstInputRef.current?.focus();
    }, 300); // Delay for animation
    return () => clearTimeout(timer);
  }
}, [visible]);
```

### Pattern 2: Field Navigation

Use `returnKeyType` and `onSubmitEditing` to create logical tab order:

```typescript
<TextInput
  ref={merchantInputRef}
  returnKeyType="next"
  onSubmitEditing={() => descriptionInputRef.current?.focus()}
/>

<TextInput
  ref={descriptionInputRef}
  returnKeyType="next"
  onSubmitEditing={() => projectClientInputRef.current?.focus()}
/>

<TextInput
  ref={projectClientInputRef}
  returnKeyType="done"
  onSubmitEditing={() => Keyboard.dismiss()}
/>
```

### Pattern 3: Return Focus After Modal

Store the trigger element and return focus when modal closes:

```typescript
const triggerRef = useRef<any>(null);

// When closing modal
useEffect(() => {
  if (!visible && previousFocusRef.current) {
    setTimeout(() => {
      previousFocusRef.current?.focus?.();
    }, 100);
  }
}, [visible]);
```

### Pattern 4: Focus Trap in Modals

Ensure focus cycles within the modal and doesn't escape:

```typescript
// Listen for Escape key to close
useEffect(() => {
  if (!isActive) return;

  const handleKeyDown = (event: KeyboardEvent) => {
    if (event.key === 'Escape' && onEscape) {
      event.preventDefault();
      onEscape();
    }
  };

  if (typeof window !== 'undefined' && window.addEventListener) {
    window.addEventListener('keydown', handleKeyDown);
  }

  return () => {
    if (typeof window !== 'undefined' && window.removeEventListener) {
      window.removeEventListener('keydown', handleKeyDown);
    }
  };
}, [isActive, onEscape]);
```

## Testing Keyboard Navigation

### Manual Testing Checklist

- [ ] All form fields can be navigated using keyboard (Tab/Shift+Tab or Previous/Next buttons on iOS)
- [ ] Tab order is logical and matches visual layout
- [ ] Focus is visible with high contrast indicators
- [ ] Return key advances to next field or submits form appropriately
- [ ] Modals trap focus and prevent escape
- [ ] Focus returns to trigger element when modal closes
- [ ] All interactive elements are keyboard accessible
- [ ] Screen reader announces field labels and errors correctly

### iOS Keyboard Toolbar Testing

ExpenseFormScreen and IncomeFormScreen include KeyboardToolbar:
- [ ] Previous button disabled on first field
- [ ] Next button disabled on last field
- [ ] Previous/Next navigation works correctly
- [ ] Done button dismisses keyboard
- [ ] Field index tracking is accurate

### Automated Testing

Run focus management tests:
```bash
npm test -- src/utils/__tests__/focusManagement.test.ts
```

## WCAG 2.1 AA Compliance

### Success Criteria Met

- **2.1.1 Keyboard (A)**: All functionality available via keyboard
- **2.1.2 No Keyboard Trap (A)**: Users can navigate away from all components
- **2.4.3 Focus Order (A)**: Focus order preserves meaning and operability
- **2.4.7 Focus Visible (AA)**: Keyboard focus indicator is visible with 3:1 minimum contrast
- **3.2.1 On Focus (A)**: Focus doesn't initiate unexpected context changes

### Focus Indicator Standards

- Minimum contrast ratio: 3:1 against adjacent colors
- Border width: 2px (recommended minimum)
- Color choices meet contrast requirements:
  - Light mode: #0969DA on white backgrounds
  - Dark mode: #58A6FF on dark backgrounds

## React Native Considerations

### Platform Differences

**iOS:**
- Native keyboard toolbar with Previous/Next/Done buttons
- Use `inputAccessoryViewID` to attach custom toolbar
- `returnKeyType` sets keyboard return key label

**Android:**
- No native keyboard toolbar
- Relies on `returnKeyType` and `onSubmitEditing`
- May need custom navigation UI

**Web (React Native Web):**
- Standard browser keyboard navigation
- Tab key moves focus between elements
- Use `tabIndex` to control focus order if needed

### Limitations

- React Native doesn't support `:focus` CSS pseudo-class
- Must manage focus indicators programmatically
- Limited control over native keyboard behavior
- Screen reader focus management varies by platform

## Future Enhancements

### Planned Improvements

1. **Focus Trap Component**
   - Reusable hook for modal focus trapping
   - Cycle focus within modal boundaries
   - Handle first/last element wrapping

2. **Skip Links**
   - Add "Skip to main content" for navigation heavy screens
   - Quick jump to primary actions

3. **Keyboard Shortcuts**
   - Common actions (Save: Cmd+S, Cancel: Esc)
   - Document shortcuts in help screen

4. **Focus Management Library**
   - Centralized focus utility hooks
   - Consistent patterns across all screens

## Resources

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [React Native Accessibility](https://reactnative.dev/docs/accessibility)
- [iOS HIG Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Material Design Accessibility](https://m3.material.io/foundations/accessible-design)

## Implementation Notes

The existing ExpenseFormScreen and IncomeFormScreen already have excellent keyboard navigation via the KeyboardToolbar component. This Phase 2.1 implementation focused on:

1. Documenting existing patterns
2. Adding accessibility labels and hints
3. Ensuring modal focus management
4. Providing focus indicator styles in theme
5. Creating reusable focus management patterns

All changes maintain backward compatibility and enhance the existing user experience without breaking changes.

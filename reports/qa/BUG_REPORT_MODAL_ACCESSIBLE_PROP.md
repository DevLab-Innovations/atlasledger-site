# Bug Report: Invalid 'accessible' Prop on Modal Component

**Bug ID**: BUG-2026-01-23-001
**Severity**: Critical (Release Blocker)
**Component**: LocationPickerModal
**Discovered**: 2026-01-23
**Reporter**: qa-tester agent
**Status**: Open

## Summary

TypeScript compilation fails due to invalid `accessible` prop being used on react-native-paper Modal component. The Modal component does not support this prop in its type definition.

## Severity Justification

**Critical** because:
- Blocks TypeScript compilation completely (`npx tsc --noEmit` fails)
- Prevents production build
- Violates type safety requirements
- Release blocker for promotion from dev to main

## Component

**File**: `src/components/LocationPickerModal.tsx`
**Lines**: 245-246
**Feature**: Wave 5.5 Map-based Mileage Tracking (PR #50)

## Steps to Reproduce

1. Checkout dev branch
   ```bash
   git checkout dev
   git pull
   ```

2. Run TypeScript compilation check
   ```bash
   npx tsc --noEmit
   ```

3. Observe compilation error

## Expected Result

TypeScript compilation should succeed with zero errors.

```
$ npx tsc --noEmit
$ (no output - successful compilation)
```

## Actual Result

TypeScript compilation fails with type error:

```
src/components/LocationPickerModal.tsx(245,9): error TS2322: Type '{ children: Element[]; visible: boolean; onDismiss: () => undefined; contentContainerStyle: { shadowColor: string; shadowOffset: { width: number; height: number; }; shadowOpacity: number; ... 6 more ...; maxHeight: "80%"; }; accessible: true; accessibilityLabel: string; }' is not assignable to type 'IntrinsicAttributes & Props'.
  Property 'accessible' does not exist on type 'IntrinsicAttributes & Props'.
```

## Code Snippet

**Current (Broken) Code** - Lines 241-247:

```tsx
<Modal
  visible={visible}
  onDismiss={() => void handleDismiss()}
  contentContainerStyle={styles.modalContainer}
  accessible          // ❌ INVALID - Modal doesn't support this prop
  accessibilityLabel={title}  // ❌ INVALID - Modal doesn't support this prop
>
```

## Root Cause Analysis

The react-native-paper `Modal` component does not accept `accessible` or `accessibilityLabel` props. These props are valid for React Native core components like TextInput, Button, etc., but not for Modal.

From react-native-paper's Modal type definition, the supported props are:
- `visible: boolean`
- `onDismiss: () => void`
- `contentContainerStyle?: StyleProp<ViewStyle>`
- `dismissable?: boolean`
- `children: React.ReactNode`
- `theme?: Theme`

The `accessible` prop does not exist in this type definition.

## Recommended Fix

Remove the invalid props from the Modal component:

```tsx
<Modal
  visible={visible}
  onDismiss={() => void handleDismiss()}
  contentContainerStyle={styles.modalContainer}
>
```

**Rationale**: Accessibility is already handled by the Modal's content:
- The header contains a Text component with the title
- The close IconButton has `accessibilityLabel="Close location picker"`
- All interactive elements inside the modal have proper accessibility props

The Modal wrapper itself doesn't need accessibility props because it's a container. Screen readers will properly announce the modal's content.

## Verification Steps

After applying the fix:

1. Run TypeScript compilation:
   ```bash
   npx tsc --noEmit
   ```
   Expected: No errors

2. Run test suite:
   ```bash
   npm test
   ```
   Expected: All 865 tests pass

3. Manual testing (iOS Simulator):
   - Open app
   - Navigate to Mileage Form
   - Tap "Start Location" or "End Location"
   - Verify LocationPickerModal opens correctly
   - Enable VoiceOver
   - Verify screen reader announces modal content properly

## Impact Assessment

### Build Impact
- ❌ Cannot compile TypeScript
- ❌ Cannot create production build
- ❌ IDE shows type errors

### Runtime Impact
- ⚠️ May work in runtime (React Native ignores unknown props)
- ⚠️ Console warnings possible
- ❌ Violates type safety

### User Impact
- No direct user impact if runtime works
- But blocks release completely due to compilation failure

### Development Impact
- Blocks all developers from building
- IDE errors distract from development
- Cannot pass CI/CD checks

## Additional Context

### Other Occurrences

The same file has `accessible` prop used on TextInput components (lines 210, 226), which is **correct** because TextInput does support this prop. Only the Modal usage is invalid.

### How This Was Missed

This suggests the developer and code-reviewer didn't run `npx tsc --noEmit` before merging the PR. The automated tests passed (865/865) because Jest doesn't enforce TypeScript types at runtime.

### Prevention

To prevent similar issues:
1. Code review checklist must include running `npx tsc --noEmit`
2. CI/CD pipeline should fail if TypeScript compilation fails
3. Pre-commit hook could run type checking

## Related Files

Only one file needs modification:
- `src/components/LocationPickerModal.tsx`

## Testing Evidence

Test suite results:
- Total tests: 865
- Passing: 865
- Failing: 0

But TypeScript compilation: **FAILED**

This demonstrates that passing tests alone are insufficient - type checking is mandatory.

## Priority

**HIGHEST** - This must be fixed before any promotion to main.

## Assigned To

Requesting handoff to frontend developer (jen-frontend-dev) for immediate fix.

## Fix Branch

Suggested branch name: `fix/wave-5.5-modal-accessible-prop`

## Estimated Fix Time

5 minutes (trivial fix - just remove 2 lines)

## Estimated Test Time

10 minutes (re-run compilation + tests + smoke test)

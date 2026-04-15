# Tech Debt Report

**Generated:** 2026-01-09
**Codebase:** Expense Tracker MVP
**Lines Reviewed:** ~20,000 across 38 files

---

## Executive Summary

The codebase has a solid foundation but exhibits several patterns common to AI-generated code: **repeated patterns without abstraction**, **inconsistent error handling**, **missing dependency injection**, and **hardcoded values**. The tech debt is manageable now but will compound quickly as features are added.

**Priority Level Legend:**
- **P0 (Critical):** Security risks or data loss potential
- **P1 (High):** Architectural issues that block scalability
- **P2 (Medium):** Code quality issues affecting maintainability
- **P3 (Low):** Nice-to-have improvements

---

## P0 - Critical Issues

### 1. SQL Injection Vulnerability Risk
**Location:** `src/services/DatabaseService.ts:216-236`

```typescript
// Current: Dynamic field building without validation
Object.entries(updates).forEach(([key, value]) => {
  fields.push(`${key} = ?`);  // key is not validated
  values.push(value);
});
```

**Risk:** If `updates` object contains malicious keys, SQL injection is possible.

**Fix:** Whitelist allowed column names before building query.

### 2. No Input Sanitization
**Location:** All form screens

Form inputs flow directly to database without sanitization. XSS payloads could be stored and rendered.

**Fix:** Add input sanitization layer before database operations.

### 3. Unencrypted Sensitive Data
**Location:** SQLite database, AsyncStorage

Financial data stored in plaintext. Device compromise exposes all expense data.

**Fix:** Consider `expo-sqlite` encryption or `expo-secure-store` for sensitive fields.

---

## P1 - High Priority (Architecture)

### 4. Service Instantiation Anti-Pattern
**Location:** Every screen component

```typescript
// Every screen does this:
const dbService = new DatabaseService();
const fileService = new FileService();

// Then in useEffect:
await dbService.initialize();
```

**Problems:**
- Multiple service instances created
- Initialization called repeatedly (wasteful)
- No singleton guarantee
- Impossible to mock for testing without module-level mocks
- No cleanup on unmount

**Fix:** Create a service context provider:
```typescript
// ServiceProvider.tsx
const services = {
  db: new DatabaseService(),
  file: new FileService(),
  settings: new SettingsService(),
};

// Initialize once at app startup
// Provide via React Context
```

### 5. Duplicated First Launch Logic
**Location:**
- `src/navigation/RootNavigator.tsx:12-35` (own AsyncStorage key)
- `src/services/SettingsService.ts:56-62` (settings object)

Two different mechanisms track first launch. They can desync.

**Fix:** Use single source of truth via SettingsService.

### 6. Duplicated Date Grouping Logic
**Location:**
- `src/utils/dateGrouping.ts` (used by ExpenseListScreen)
- `src/screens/MileageListScreen.tsx:60-96` (inline implementation)

**Fix:** Generalize `dateGrouping.ts` to work with any date-bearing object.

### 7. No Global Error Boundary
**Location:** N/A (missing)

Unhandled errors crash the app. No error reporting or recovery.

**Fix:** Add React error boundary with crash reporting.

---

## P2 - Medium Priority (Code Quality)

### 8. Hardcoded Theme Values
**Location:** All StyleSheet definitions

```typescript
// Theme defines:
colors.background = '#F5F5F5';
spacing.md = 16;

// But screens use:
backgroundColor: '#f5f5f5',  // hardcoded
padding: 16,                  // hardcoded
```

**Files affected:** All 9 screens, 2 components

**Fix:** Import and use theme constants consistently.

### 9. TypeScript `any` Types
**Locations:**
- `src/services/DatabaseService.ts:220` - `values: any[]`
- `src/screens/ExpenseFormScreen.tsx:90` - `as any`
- `src/screens/MileageFormScreen.tsx:153` - `event: any`

**Fix:** Define proper types for all values.

### 10. Inconsistent Loading States
**Location:** List screens

```typescript
// MileageListScreen shows:
<Text>Loading...</Text>

// ExpenseListScreen shows:
// Nothing - just empty state while loading
```

**Fix:** Create unified loading component, use consistently.

### 11. Magic Strings and Numbers
**Locations:**
- Payment methods: `['Cash', 'Credit', 'Debit', 'Business']` (hardcoded in component)
- Mileage rate: `0.67` (default in multiple places)
- Database name: `'expense_tracker.db'` (repeated in DatabaseService and MigrationService)

**Fix:** Extract to constants file.

### 12. Inconsistent Import Paths
**Location:** Throughout codebase

```typescript
// Some files use:
import { DatabaseService } from '@/services/DatabaseService';

// Others use:
import { DatabaseService } from '../services/DatabaseService';
```

**Fix:** Standardize on `@/` alias everywhere.

---

## P3 - Low Priority (Performance & Polish)

### 13. Missing FlatList Optimizations
**Location:** `ExpenseListScreen.tsx`, `MileageListScreen.tsx`

Missing:
- `getItemLayout` (enables faster scrolling)
- `initialNumToRender`
- `maxToRenderPerBatch`
- `windowSize`

### 14. No Memoization
**Location:** List screens

```typescript
// Recalculated on every render:
const flattenedData = groupedExpenses.flatMap(...)
```

**Fix:** Wrap in `useMemo`.

### 15. Redundant Data Loading
**Location:** List screens load all data on every focus

```typescript
navigation.addListener('focus', () => {
  loadExpenses();  // Always reloads everything
});
```

**Fix:** Only reload if data changed (use state timestamp or event bus).

### 16. Missing Accessibility
**Location:** All screens

- No `accessibilityLabel` on interactive elements
- No `accessibilityHint` for complex actions
- Color contrast not verified

---

## Testing Debt

### Current State
- **Unit tests:** Services have basic coverage
- **Component tests:** Form screens have validation tests
- **Integration tests:** None
- **E2E tests:** None

### Missing Test Coverage
1. Database migration scenarios
2. Concurrent access to services
3. Error recovery paths
4. Network failure handling
5. Large dataset performance (500+ expenses)

---

## Recommendations by Sprint

### Sprint 1 (Critical)
1. Fix SQL injection vulnerability (P0)
2. Add input sanitization (P0)
3. Implement service context provider (P1)

### Sprint 2 (Architecture)
1. Consolidate first launch logic (P1)
2. Add global error boundary (P1)
3. Generalize date grouping utility (P1)

### Sprint 3 (Code Quality)
1. Replace hardcoded theme values (P2)
2. Fix TypeScript `any` types (P2)
3. Extract magic strings to constants (P2)

### Sprint 4 (Polish)
1. Add FlatList optimizations (P3)
2. Implement memoization (P3)
3. Add accessibility labels (P3)

---

## Metrics

| Metric | Current | Target |
|--------|---------|--------|
| TypeScript `any` usage | 8 instances | 0 |
| Hardcoded colors | 23 instances | 0 |
| Service instantiations per render | 2-3 | 0 |
| Test coverage (estimated) | ~40% | 80% |
| Accessibility score | Unknown | WCAG AA |

---

*Report generated by code review. Re-run after each sprint to track progress.*

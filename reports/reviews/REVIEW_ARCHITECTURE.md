# Architecture Review: Expense Tracker

**Review Date:** January 9, 2026
**Reviewer:** Senior Architect Agent
**Project Phase:** Wave 2 Complete (Core Functionality)

## Executive Summary

The Expense Tracker application demonstrates a **solid foundation** with well-structured services, clear separation of concerns, and appropriate technology choices for a local-first React Native application. The codebase is in good shape for an MVP, but several architectural patterns need refinement to ensure long-term maintainability and scalability.

**Overall Assessment:** 🟡 Good with Improvement Areas

### Key Strengths
- Clean service layer architecture
- Comprehensive TypeScript typing
- Local-first design with SQLite and FileSystem
- Well-organized navigation structure
- Good test coverage for services

### Critical Areas for Improvement
1. Service instantiation pattern (creating new instances everywhere)
2. Missing global error handling and logging strategy
3. Lack of centralized state management
4. No ESLint configuration for code quality
5. Inconsistent error handling patterns
6. Missing hooks layer for service interaction

---

## 1. Architecture Patterns

### Current State
The application follows a **layered architecture**:
```
UI Layer (Screens/Components)
    ↓
Services Layer (Direct instantiation)
    ↓
Storage Layer (SQLite/FileSystem/AsyncStorage)
```

### Issues Identified

#### 🔴 CRITICAL: Service Instantiation Anti-Pattern
**Severity:** Critical
**Scope:** All screens that use services
**Files Affected:**
- `/Users/neptune/repos/agenticSdlc/src/screens/ExpenseListScreen.tsx`
- `/Users/neptune/repos/agenticSdlc/src/screens/ExpenseFormScreen.tsx`
- `/Users/neptune/repos/agenticSdlc/src/screens/SettingsScreen.tsx`
- All other screens using services

**Problem:**
```typescript
// Current anti-pattern - new instance created on every render
export default function ExpenseListScreen({ navigation }: ExpenseListScreenProps) {
  const dbService = new DatabaseService();  // ❌ Created on every render
  const fileService = new FileService();    // ❌ Created on every render

  useEffect(() => {
    loadExpenses();
  }, []);

  const loadExpenses = async () => {
    await dbService.initialize();  // Re-initializing unnecessarily
    // ...
  };
}
```

**Impact:**
- Multiple service instances created unnecessarily
- Redundant database initialization calls
- Memory overhead from duplicate instances
- Potential state inconsistencies across service instances
- Testing becomes harder (can't mock singleton)

**Recommended Solution:**
Implement a **Service Provider pattern** with React Context:

```typescript
// src/services/ServiceContext.tsx
import React, { createContext, useContext, useMemo } from 'react';

interface Services {
  database: DatabaseService;
  file: FileService;
  settings: SettingsService;
  migration: MigrationService;
}

const ServiceContext = createContext<Services | null>(null);

export function ServiceProvider({ children }: { children: React.ReactNode }) {
  const services = useMemo(() => ({
    database: new DatabaseService(),
    file: new FileService(),
    settings: new SettingsService(),
    migration: new MigrationService(),
  }), []);

  useEffect(() => {
    // Initialize all services once
    async function init() {
      await services.migration.initialize();
      await services.database.initialize();
      await services.file.initialize();
    }
    init();
  }, []);

  return (
    <ServiceContext.Provider value={services}>
      {children}
    </ServiceContext.Provider>
  );
}

export function useServices() {
  const context = useContext(ServiceContext);
  if (!context) {
    throw new Error('useServices must be used within ServiceProvider');
  }
  return context;
}
```

Then update screens:
```typescript
// Updated pattern
export default function ExpenseListScreen({ navigation }: ExpenseListScreenProps) {
  const { database, file } = useServices();  // ✅ Singleton access

  const loadExpenses = async () => {
    const expenses = await database.getAllExpenses();
    // ...
  };
}
```

**Migration Path:**
1. Create `src/services/ServiceContext.tsx`
2. Wrap app in `ServiceProvider` in `App.tsx`
3. Create custom hooks for each service interaction pattern
4. Update screens one by one to use hooks
5. Remove direct service instantiation

---

#### 🟠 HIGH: Missing Hooks Layer
**Severity:** High
**Scope:** All data-fetching screens

**Problem:**
Business logic is embedded directly in screens, leading to:
- Code duplication across screens
- Difficult testing of business logic
- Poor separation of concerns
- No centralized data refetch/invalidation

**Recommended Solution:**
Create a **hooks layer** that abstracts service interactions:

```typescript
// src/hooks/useExpenses.ts
export function useExpenses() {
  const { database } = useServices();
  const [expenses, setExpenses] = useState<Expense[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const loadExpenses = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await database.getAllExpenses();
      setExpenses(data);
    } catch (err) {
      setError(err as Error);
      console.error('Failed to load expenses:', err);
    } finally {
      setLoading(false);
    }
  }, [database]);

  useEffect(() => {
    loadExpenses();
  }, [loadExpenses]);

  const createExpense = useCallback(async (data: CreateExpenseData) => {
    await database.createExpense(data);
    await loadExpenses(); // Auto-refresh
  }, [database, loadExpenses]);

  const deleteExpense = useCallback(async (id: number) => {
    await database.deleteExpense(id);
    await loadExpenses(); // Auto-refresh
  }, [database, loadExpenses]);

  return {
    expenses,
    loading,
    error,
    refresh: loadExpenses,
    createExpense,
    deleteExpense,
  };
}
```

**Benefits:**
- Centralized data loading logic
- Automatic refresh after mutations
- Easier testing (test hooks independently)
- Cleaner screen components
- Consistent loading/error states

---

#### 🟠 HIGH: No Global Error Handling Strategy
**Severity:** High
**Scope:** Entire application

**Problem:**
Error handling is inconsistent:
- Some places use `Alert.alert()`
- Console.error scattered everywhere
- No error tracking/reporting
- User experience varies across errors
- No graceful degradation

**Current Pattern:**
```typescript
// Inconsistent error handling
try {
  await dbService.initialize();
} catch (error) {
  console.error('Failed to initialize database:', error);
  throw error; // Sometimes throws, sometimes doesn't
}

try {
  await loadExpenses();
} catch (error) {
  console.error('Failed to load expenses:', error);
  Alert.alert('Error', 'Failed to load expenses'); // Different UX
}
```

**Recommended Solution:**
Implement a **centralized error handling system**:

```typescript
// src/services/ErrorService.ts
export enum ErrorSeverity {
  INFO = 'info',
  WARNING = 'warning',
  ERROR = 'error',
  CRITICAL = 'critical',
}

export interface AppError {
  code: string;
  message: string;
  originalError: Error;
  severity: ErrorSeverity;
  context?: Record<string, any>;
}

export class ErrorService {
  static handle(error: Error, context?: Record<string, any>) {
    const appError = this.classify(error, context);

    // Log to console (could add external logging in future)
    this.log(appError);

    // Show user-friendly message
    this.showUserMessage(appError);

    // Track for analytics (future)
    this.track(appError);
  }

  private static classify(error: Error, context?: Record<string, any>): AppError {
    // Classify error based on type, message, etc.
    if (error.message.includes('database')) {
      return {
        code: 'DB_ERROR',
        message: 'Database operation failed',
        originalError: error,
        severity: ErrorSeverity.ERROR,
        context,
      };
    }
    // ... more classifications
  }

  private static showUserMessage(error: AppError) {
    if (error.severity === ErrorSeverity.CRITICAL) {
      Alert.alert(
        'Critical Error',
        'Something went wrong. Please restart the app.',
        [{ text: 'OK' }]
      );
    } else if (error.severity === ErrorSeverity.ERROR) {
      Alert.alert('Error', error.message);
    }
    // WARNING and INFO might use toast notifications
  }
}

// Usage in screens/services
try {
  await database.createExpense(data);
} catch (error) {
  ErrorService.handle(error as Error, { screen: 'ExpenseForm', action: 'create' });
}
```

---

#### 🟡 MEDIUM: Inconsistent Data Flow
**Severity:** Medium
**Scope:** Screen components

**Problem:**
- Screens refetch data on focus listener
- Manual refresh logic in multiple places
- No data invalidation strategy
- Duplicate loading states

**Recommended Solution:**
Consider adding a lightweight state management solution like **Zustand** for shared state:

```typescript
// src/store/expenseStore.ts
import create from 'zustand';

interface ExpenseStore {
  expenses: Expense[];
  loading: boolean;
  error: Error | null;

  loadExpenses: () => Promise<void>;
  addExpense: (expense: CreateExpenseData) => Promise<void>;
  deleteExpense: (id: number) => Promise<void>;
  invalidate: () => void;
}

export const useExpenseStore = create<ExpenseStore>((set, get) => ({
  expenses: [],
  loading: false,
  error: null,

  loadExpenses: async () => {
    set({ loading: true, error: null });
    try {
      const database = getService('database'); // From service provider
      const expenses = await database.getAllExpenses();
      set({ expenses, loading: false });
    } catch (error) {
      set({ error: error as Error, loading: false });
    }
  },

  addExpense: async (data) => {
    const database = getService('database');
    await database.createExpense(data);
    await get().loadExpenses(); // Auto-refresh
  },

  deleteExpense: async (id) => {
    const database = getService('database');
    await database.deleteExpense(id);
    await get().loadExpenses(); // Auto-refresh
  },

  invalidate: () => {
    get().loadExpenses();
  },
}));
```

**Benefits:**
- Single source of truth for expense data
- Automatic UI updates when data changes
- Easier to test
- Better performance (avoid unnecessary refetches)

---

## 2. Dependency Management

### Current State
**Dependencies:** 17 production, 9 dev dependencies

### Issues Identified

#### 🔴 CRITICAL: Missing ESLint Configuration
**Severity:** Critical
**Scope:** Entire codebase

**Problem:**
- No ESLint configuration file in project root
- No linting enforcement for code quality
- Potential for inconsistent code styles
- TypeScript strict mode is enabled but no lint rules
- Package.json has `lint` script but no config

**Impact:**
- Code quality issues may slip through
- Inconsistent patterns across codebase
- Harder to maintain as team grows
- No auto-formatting/fixing

**Recommended Solution:**
Create `.eslintrc.js`:

```javascript
module.exports = {
  root: true,
  extends: [
    '@react-native',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint', 'react', 'react-native'],
  rules: {
    // TypeScript
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',

    // React
    'react/prop-types': 'off',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    // React Native
    'react-native/no-inline-styles': 'warn',
    'react-native/no-color-literals': 'warn',
    'react-native/no-raw-text': 'off',

    // Code quality
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-var': 'error',
  },
};
```

**Required Dev Dependencies:**
```bash
npm install --save-dev \
  eslint \
  @react-native/eslint-config \
  @typescript-eslint/eslint-plugin \
  @typescript-eslint/parser \
  eslint-plugin-react \
  eslint-plugin-react-native \
  eslint-plugin-react-hooks
```

---

#### 🟡 MEDIUM: Missing Prettier Configuration
**Severity:** Medium
**Scope:** Code formatting

**Problem:**
- No automated code formatting
- Potential style inconsistencies
- Manual formatting in code reviews

**Recommended Solution:**
Add Prettier configuration:

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

Install:
```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

---

#### 🟡 MEDIUM: React Hook Form Validation
**Severity:** Medium
**Scope:** Form screens

**Problem:**
React Hook Form is installed but **Yup/Zod validation** is not configured for schema-based validation. Currently using inline validation functions.

**Current:**
```typescript
const validateAmount = (value: string): string | true => {
  if (!value || value.trim() === '') {
    return 'Amount is required';
  }
  const amount = parseFloat(value);
  if (isNaN(amount) || amount <= 0) {
    return 'Amount must be greater than 0';
  }
  return true;
};
```

**Recommended:**
```typescript
// With Zod
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const expenseSchema = z.object({
  amount: z.string()
    .min(1, 'Amount is required')
    .refine((val) => !isNaN(parseFloat(val)) && parseFloat(val) > 0, {
      message: 'Amount must be greater than 0',
    }),
  category: z.string().min(1, 'Category is required'),
  date: z.date().max(new Date(), 'Date cannot be in the future'),
  // ... more fields
});

const { control, handleSubmit } = useForm<ExpenseFormData>({
  resolver: zodResolver(expenseSchema),
});
```

**Install:**
```bash
npm install zod @hookform/resolvers
```

---

#### 🟢 LOW: Bundle Size Analysis Missing
**Severity:** Low
**Scope:** Build optimization

**Problem:**
- No bundle size analysis tool configured
- Unknown which dependencies contribute most to bundle size
- Potential for optimization

**Recommended Solution:**
Add bundle analysis script:
```bash
npm install --save-dev @expo/webpack-config
```

---

## 3. Code Organization

### Current Structure
```
src/
├── components/        # 2 files (DatePicker, CurrencyInput)
├── navigation/        # 6 files (well organized)
├── screens/          # 9 files
├── services/         # 4 files
├── theme/            # 1 file
├── types/            # 2 files
├── utils/            # 1 file
└── __tests__/        # Test files mirror src structure
```

### Issues Identified

#### 🟡 MEDIUM: Missing Index Files
**Severity:** Medium
**Scope:** All directories

**Problem:**
No `index.ts` barrel exports for cleaner imports:

**Current:**
```typescript
import { DatabaseService } from '@/services/DatabaseService';
import { FileService } from '@/services/FileService';
import { SettingsService } from '@/services/SettingsService';
```

**Better:**
```typescript
import { DatabaseService, FileService, SettingsService } from '@/services';
```

**Recommended Solution:**
Add index files:

```typescript
// src/services/index.ts
export { DatabaseService } from './DatabaseService';
export { FileService } from './FileService';
export { SettingsService } from './SettingsService';
export { MigrationService } from './MigrationService';

// src/components/index.ts
export { DatePicker } from './DatePicker';
export { CurrencyInput } from './CurrencyInput';

// src/types/index.ts (already exists, expand it)
export * from './navigation';
export type { Expense, Mileage, Category, Settings } from './index';
```

---

#### 🟡 MEDIUM: Constants Need Extraction
**Severity:** Medium
**Scope:** Multiple files

**Problem:**
Magic strings and constants scattered throughout code:

**Examples:**
- IRS categories hardcoded in DatabaseService
- Payment methods array in ExpenseFormScreen
- Sort options in ExpenseListScreen
- Storage keys scattered

**Recommended Solution:**
```typescript
// src/constants/categories.ts
export const IRS_CATEGORIES = [
  'Advertising',
  'Car and Truck Expenses',
  // ...
] as const;

// src/constants/expense.ts
export const PAYMENT_METHODS = ['Cash', 'Credit', 'Debit', 'Business'] as const;
export type PaymentMethod = typeof PAYMENT_METHODS[number];

export const SORT_OPTIONS = {
  DATE_DESC: 'date-desc',
  DATE_ASC: 'date-asc',
  AMOUNT_DESC: 'amount-desc',
  AMOUNT_ASC: 'amount-asc',
  CATEGORY_ASC: 'category-asc',
} as const;

// src/constants/storage.ts
export const STORAGE_KEYS = {
  SETTINGS: '@expense_tracker:settings',
  FIRST_LAUNCH: '@expense_tracker:first_launch',
} as const;

// src/constants/index.ts
export * from './categories';
export * from './expense';
export * from './storage';
```

---

#### 🟢 LOW: Component Library Could Use More Reusable Components
**Severity:** Low
**Scope:** Components directory

**Current State:**
Only 2 custom components (`DatePicker`, `CurrencyInput`). Many patterns are repeated across screens.

**Recommended Components to Extract:**
1. `EmptyState` - Used in multiple list screens
2. `LoadingSpinner` - Loading states
3. `ErrorMessage` - Error display
4. `FormField` - Wrapper for form inputs with labels
5. `ConfirmDialog` - Reusable confirmation dialogs
6. `SectionHeader` - List section headers
7. `ListItem` - Styled list items

---

## 4. Cross-Cutting Concerns

### Issues Identified

#### 🔴 CRITICAL: No Logging Strategy
**Severity:** Critical
**Scope:** Entire application

**Problem:**
- `console.log` and `console.error` scattered everywhere
- No structured logging
- No log levels
- Cannot disable logs in production
- No context about where logs come from

**Recommended Solution:**
Create a logging service:

```typescript
// src/services/LogService.ts
enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3,
}

class LogService {
  private static level: LogLevel = __DEV__ ? LogLevel.DEBUG : LogLevel.ERROR;

  static debug(message: string, context?: Record<string, any>) {
    if (this.level <= LogLevel.DEBUG) {
      console.log(`[DEBUG] ${message}`, context || '');
    }
  }

  static info(message: string, context?: Record<string, any>) {
    if (this.level <= LogLevel.INFO) {
      console.log(`[INFO] ${message}`, context || '');
    }
  }

  static warn(message: string, context?: Record<string, any>) {
    if (this.level <= LogLevel.WARN) {
      console.warn(`[WARN] ${message}`, context || '');
    }
  }

  static error(message: string, error?: Error, context?: Record<string, any>) {
    if (this.level <= LogLevel.ERROR) {
      console.error(`[ERROR] ${message}`, error, context || '');
    }
    // Could add crash reporting service here (Sentry, etc.)
  }
}

export default LogService;

// Usage
LogService.info('Database initialized successfully');
LogService.error('Failed to load expenses', error, { userId: 123 });
```

---

#### 🟠 HIGH: No Environment Configuration
**Severity:** High
**Scope:** Configuration management

**Problem:**
- No `.env` files for configuration
- All values hardcoded
- Cannot easily switch between dev/staging/production
- Database name hardcoded

**Recommended Solution:**
Use `expo-constants` and `app.config.js`:

```javascript
// app.config.js
export default {
  expo: {
    name: process.env.APP_NAME || "Expense Tracker",
    extra: {
      databaseName: process.env.DB_NAME || 'expense_tracker.db',
      irsStandardRate: process.env.IRS_RATE || 0.67,
      environment: process.env.NODE_ENV || 'development',
    },
  },
};
```

```typescript
// src/config/index.ts
import Constants from 'expo-constants';

export const config = {
  databaseName: Constants.expoConfig?.extra?.databaseName || 'expense_tracker.db',
  irsStandardRate: Constants.expoConfig?.extra?.irsStandardRate || 0.67,
  environment: Constants.expoConfig?.extra?.environment || 'development',
  isDevelopment: Constants.expoConfig?.extra?.environment === 'development',
  isProduction: Constants.expoConfig?.extra?.environment === 'production',
};
```

---

#### 🟡 MEDIUM: No Crash Reporting
**Severity:** Medium
**Scope:** Production monitoring

**Problem:**
- No crash reporting service configured
- Production errors go unnoticed
- Cannot track error rates or patterns

**Recommended Solution:**
Integrate Sentry (or similar):

```bash
npm install @sentry/react-native
```

```typescript
// App.tsx
import * as Sentry from '@sentry/react-native';

if (!__DEV__) {
  Sentry.init({
    dsn: 'your-dsn-here',
    enableInExpoDevelopment: false,
    debug: false,
  });
}
```

---

#### 🟡 MEDIUM: No Performance Monitoring
**Severity:** Medium
**Scope:** App performance

**Problem:**
- No way to track app performance
- Cannot identify slow operations
- No metrics on render times, DB query speeds

**Recommended Solution:**
Add performance tracking utility:

```typescript
// src/utils/performance.ts
export class PerformanceMonitor {
  private static marks: Map<string, number> = new Map();

  static mark(name: string) {
    this.marks.set(name, Date.now());
  }

  static measure(name: string, startMark: string): number {
    const start = this.marks.get(startMark);
    if (!start) {
      console.warn(`No start mark found for ${startMark}`);
      return 0;
    }

    const duration = Date.now() - start;
    console.log(`[PERF] ${name}: ${duration}ms`);
    this.marks.delete(startMark);

    return duration;
  }
}

// Usage
PerformanceMonitor.mark('load-expenses-start');
await database.getAllExpenses();
PerformanceMonitor.measure('Load Expenses', 'load-expenses-start');
```

---

## 5. Scalability Concerns

### Issues Identified

#### 🟠 HIGH: Database Service Will Grow Too Large
**Severity:** High
**Scope:** DatabaseService.ts (currently 375 lines)

**Problem:**
Single DatabaseService file will become unwieldy as more features are added. Already handling:
- Expenses CRUD
- Mileage CRUD
- Categories CRUD
- Merchant categories
- Schema creation

**Recommended Solution:**
Split into repository pattern:

```typescript
// src/repositories/ExpenseRepository.ts
export class ExpenseRepository {
  constructor(private db: SQLite.SQLiteDatabase) {}

  async getAll(): Promise<Expense[]> { /* ... */ }
  async getById(id: number): Promise<Expense | null> { /* ... */ }
  async create(data: CreateExpenseData): Promise<number> { /* ... */ }
  async update(id: number, data: Partial<Expense>): Promise<void> { /* ... */ }
  async delete(id: number): Promise<void> { /* ... */ }
}

// src/repositories/MileageRepository.ts
export class MileageRepository {
  constructor(private db: SQLite.SQLiteDatabase) {}
  // ... mileage methods
}

// src/services/DatabaseService.ts (refactored)
export class DatabaseService {
  private db: SQLite.SQLiteDatabase | null = null;
  private initialized = false;

  expenses!: ExpenseRepository;
  mileage!: MileageRepository;
  categories!: CategoryRepository;

  async initialize(): Promise<void> {
    this.db = await SQLite.openDatabaseAsync('expense_tracker.db');

    // Initialize repositories
    this.expenses = new ExpenseRepository(this.db);
    this.mileage = new MileageRepository(this.db);
    this.categories = new CategoryRepository(this.db);

    await this.createTables();
    await this.createIndexes();

    this.initialized = true;
  }
}

// Usage
const { database } = useServices();
const allExpenses = await database.expenses.getAll();
```

**Benefits:**
- Better organization and separation of concerns
- Easier to test individual repositories
- Clearer boundaries between different data domains
- Easier to maintain as features grow

---

#### 🟡 MEDIUM: Navigation Type Safety Needs Improvement
**Severity:** Medium
**Scope:** Navigation types

**Problem:**
Navigation params are loosely typed in some places. Could be more type-safe.

**Current:**
```typescript
navigation.navigate('ExpenseForm', { expenseId: 123 });
// Type checking is okay but not enforced at call site
```

**Recommended Enhancement:**
Use a type-safe navigation helper:

```typescript
// src/navigation/navigationHelpers.ts
export const NavigationHelpers = {
  toExpenseForm: (navigation: NavigationProp<any>, params?: { expenseId: number }) => {
    navigation.navigate('ExpenseForm', params);
  },

  toExpenseDetail: (navigation: NavigationProp<any>, expenseId: number) => {
    navigation.navigate('ExpenseDetail', { expenseId });
  },
};

// Usage
NavigationHelpers.toExpenseDetail(navigation, expense.id);
```

---

#### 🟡 MEDIUM: Testing Strategy Needs Expansion
**Severity:** Medium
**Scope:** Test coverage

**Current State:**
- Good unit tests for services
- Some screen tests
- No integration tests
- No E2E tests

**Recommended Additions:**
1. **Integration Tests:** Test service + database interactions
2. **E2E Tests:** Use Detox for critical user flows
3. **Snapshot Tests:** For component UI consistency
4. **Performance Tests:** Database query performance with large datasets

```typescript
// Example integration test
describe('Expense Creation Flow (Integration)', () => {
  it('should create expense and save to database', async () => {
    const db = new DatabaseService();
    await db.initialize();

    const expenseId = await db.createExpense({
      amount: 50.00,
      date: '2024-01-01',
      category: 'Meals',
    });

    const saved = await db.getExpenseById(expenseId);
    expect(saved).toBeDefined();
    expect(saved?.amount).toBe(50.00);
  });
});
```

---

## 6. Developer Experience

### Issues Identified

#### 🟡 MEDIUM: No Pre-commit Hooks
**Severity:** Medium
**Scope:** Code quality enforcement

**Problem:**
- No automated checks before commits
- Linting and testing not enforced
- Potential for broken code to be committed

**Recommended Solution:**
Add Husky and lint-staged:

```bash
npm install --save-dev husky lint-staged
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
npm run lint
npm run typecheck
npm run test -- --bail --findRelatedTests
```

---

#### 🟡 MEDIUM: Missing VSCode Workspace Settings
**Severity:** Medium
**Scope:** Editor configuration

**Problem:**
- No shared editor configuration
- Inconsistent editor settings across developers
- No recommended extensions

**Recommended Solution:**
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "files.exclude": {
    "**/.expo": true,
    "**/.expo-shared": true,
    "**/node_modules": true
  }
}

// .vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "expo.vscode-expo-tools"
  ]
}
```

---

#### 🟢 LOW: Documentation Could Be Enhanced
**Severity:** Low
**Scope:** Developer onboarding

**Current State:**
- Good ROADMAP.md and design.md
- Missing API documentation
- No architecture diagrams
- No contribution guidelines

**Recommended Additions:**
1. `CONTRIBUTING.md` - How to contribute
2. `ARCHITECTURE.md` - System architecture overview
3. JSDoc comments on all public service methods
4. Component storybook (future consideration)

---

## 7. React Native Specific Concerns

### Issues Identified

#### 🟠 HIGH: AsyncStorage Used for Settings Without Encryption
**Severity:** High
**Scope:** SettingsService

**Problem:**
- Settings stored in plain text via AsyncStorage
- Potential sensitive data (mileage rates, preferences)
- No data encryption

**Context:**
Current implementation uses AsyncStorage directly for settings. While this is fine for non-sensitive data, future features may include sensitive information (passcode settings, biometric preferences).

**Recommended Solution:**
For future-proofing, consider using `expo-secure-store` for sensitive settings:

```typescript
// src/services/SettingsService.ts
import * as SecureStore from 'expo-secure-store';

export class SettingsService {
  async setPasscode(passcode: string): Promise<void> {
    await SecureStore.setItemAsync('user_passcode', passcode);
  }

  async getPasscode(): Promise<string | null> {
    return await SecureStore.getItemAsync('user_passcode');
  }
}
```

---

#### 🟡 MEDIUM: Potential Performance Issues with Large Lists
**Severity:** Medium
**Scope:** ExpenseListScreen, MileageListScreen

**Current State:**
Using `FlatList` with proper virtualization, which is good. However:

**Potential Issues:**
1. Grouping expenses client-side could be expensive with 1000+ expenses
2. No pagination or infinite scroll
3. Loading all expenses at once

**Recommended Optimization:**
```typescript
// Add pagination to DatabaseService
async getExpensesPaginated(
  page: number,
  pageSize: number = 50
): Promise<Expense[]> {
  const offset = page * pageSize;
  const expenses = await this.db.getAllAsync(
    'SELECT * FROM expenses ORDER BY date DESC, id DESC LIMIT ? OFFSET ?',
    [pageSize, offset]
  );
  return expenses as Expense[];
}

// In screen, use infinite scroll
const [page, setPage] = useState(0);
const [hasMore, setHasMore] = useState(true);

const loadMore = async () => {
  if (!hasMore || loading) return;

  const newExpenses = await database.getExpensesPaginated(page);
  if (newExpenses.length < 50) setHasMore(false);

  setExpenses([...expenses, ...newExpenses]);
  setPage(page + 1);
};
```

---

#### 🟡 MEDIUM: Navigation Memory Management
**Severity:** Medium
**Scope:** Navigation structure

**Problem:**
Screens are re-instantiating services and reloading data on every focus. This is inefficient.

**Current Pattern:**
```typescript
useEffect(() => {
  const unsubscribe = navigation.addListener('focus', () => {
    loadExpenses(); // Refetches every time
  });
  return unsubscribe;
}, [navigation]);
```

**Recommended Pattern:**
Use a more intelligent refresh strategy:

```typescript
// Only refresh if data might have changed
useEffect(() => {
  const unsubscribe = navigation.addListener('focus', () => {
    if (shouldRefresh()) {
      loadExpenses();
    }
  });
  return unsubscribe;
}, [navigation]);

const shouldRefresh = () => {
  // Check if we're coming back from a form that might have changed data
  const routes = navigation.getState().routes;
  const previousRoute = routes[routes.length - 2];
  return previousRoute?.name === 'ExpenseForm';
};
```

Or better, use the global state solution mentioned earlier where changes automatically propagate.

---

#### 🟡 MEDIUM: Theme Not Leveraging React Native Paper Fully
**Severity:** Medium
**Scope:** Theme implementation

**Problem:**
- Custom theme defined but not all screens use theme values
- Some hardcoded colors in StyleSheet definitions
- Not using theme's spacing and borderRadius consistently

**Examples:**
```typescript
// ExpenseListScreen.tsx
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff', // ❌ Hardcoded, should use theme
  },
  header: {
    paddingHorizontal: 16, // ❌ Should use theme.spacing.md
  },
});
```

**Recommended:**
```typescript
import { useTheme } from 'react-native-paper';

export default function ExpenseListScreen() {
  const theme = useTheme();

  const styles = StyleSheet.create({
    container: {
      flex: 1,
      backgroundColor: theme.colors.background,
    },
    header: {
      paddingHorizontal: theme.spacing.md,
    },
  });
}
```

Or create a `useThemedStyles` hook for consistency.

---

## 8. Security Considerations

### Issues Identified

#### 🟠 HIGH: No Input Sanitization
**Severity:** High
**Scope:** All form inputs

**Problem:**
- No sanitization of user inputs before database storage
- Potential for SQL injection (though SQLite parameterized queries help)
- XSS potential if data displayed in WebView later
- No validation of file uploads (future receipt capture)

**Recommended Solution:**
```typescript
// src/utils/sanitization.ts
export const sanitizeString = (input: string): string => {
  return input
    .trim()
    .replace(/[<>]/g, '') // Remove potential XSS chars
    .slice(0, 1000); // Limit length
};

export const sanitizeAmount = (input: string): number => {
  const amount = parseFloat(input);
  if (isNaN(amount) || amount < 0 || amount > 1000000) {
    throw new Error('Invalid amount');
  }
  return Math.round(amount * 100) / 100; // Round to 2 decimals
};

// Use in services
async createExpense(data: CreateExpenseData): Promise<number> {
  const sanitized = {
    amount: sanitizeAmount(data.amount.toString()),
    merchant: data.merchant ? sanitizeString(data.merchant) : null,
    description: data.description ? sanitizeString(data.description) : null,
    // ...
  };

  // Save sanitized data
}
```

---

#### 🟡 MEDIUM: File System Security
**Severity:** Medium
**Scope:** FileService

**Problem:**
- No validation of file paths
- No file type checking
- No file size limits enforced at service level

**Recommended Solution:**
```typescript
// src/services/FileService.ts
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const ALLOWED_EXTENSIONS = ['.jpg', '.jpeg', '.png', '.pdf'];

async saveReceipt(sourceUri: string, expenseId: number): Promise<string> {
  // Validate file exists
  const fileInfo = await FileSystem.getInfoAsync(sourceUri);
  if (!fileInfo.exists) {
    throw new Error('File does not exist');
  }

  // Check file size
  if (fileInfo.size && fileInfo.size > MAX_FILE_SIZE) {
    throw new Error('File too large. Maximum size is 10MB');
  }

  // Validate extension
  const extension = sourceUri.split('.').pop()?.toLowerCase();
  if (!extension || !ALLOWED_EXTENSIONS.includes(`.${extension}`)) {
    throw new Error('Invalid file type');
  }

  // Proceed with save
  // ...
}
```

---

## Strategic Recommendations

### Immediate Actions (Sprint 1)
1. **Implement Service Provider Pattern** - Highest priority, affects all screens
2. **Add ESLint Configuration** - Critical for code quality
3. **Create Error Handling Service** - Improve user experience
4. **Add Basic Logging Service** - Essential for debugging

### Short-term (Sprint 2-3)
1. **Create Hooks Layer** - Improve component reusability
2. **Add Environment Configuration** - Prepare for multiple environments
3. **Implement Repository Pattern** - Prevent DatabaseService from growing too large
4. **Add Pre-commit Hooks** - Enforce code quality

### Medium-term (Next Quarter)
1. **Add State Management (Zustand)** - If app complexity grows
2. **Implement Performance Monitoring** - Track app health
3. **Add Crash Reporting (Sentry)** - Production monitoring
4. **Create Reusable Component Library** - Reduce code duplication

### Long-term (Future Waves)
1. **E2E Testing with Detox** - Ensure critical flows work
2. **Component Storybook** - Document and test components in isolation
3. **Micro-frontend Architecture** - If app grows significantly
4. **GraphQL Layer** - If backend sync is added (Phase 2)

---

## Refactoring Priorities

### Priority Matrix

| Priority | Effort | Impact | Items |
|----------|--------|--------|-------|
| P0 (Do Now) | Medium | High | Service Provider Pattern, ESLint Config, Error Handling |
| P1 (Next Sprint) | Medium | High | Hooks Layer, Logging Service, Repository Pattern |
| P2 (This Quarter) | High | Medium | State Management, Performance Monitoring, Input Sanitization |
| P3 (Future) | High | Low | Component Library, Storybook, E2E Tests |

---

## Code Smells to Address

1. **God Object:** DatabaseService doing too much
2. **Duplicate Code:** Service instantiation pattern repeated everywhere
3. **Magic Numbers:** Hardcoded values throughout
4. **Feature Envy:** Screens reaching into service internals
5. **Long Method:** Some screen methods are too complex
6. **Primitive Obsession:** Using strings/numbers instead of types (e.g., payment methods)

---

## Technical Debt Assessment

**Current Technical Debt Score:** 6/10 (Moderate)

**Breakdown:**
- **Architecture Debt:** 7/10 - Some patterns need refinement
- **Code Debt:** 5/10 - Clean code but missing standards
- **Test Debt:** 4/10 - Good service tests, missing integration/E2E
- **Documentation Debt:** 6/10 - Good specs, missing inline docs
- **Infrastructure Debt:** 7/10 - Missing CI/CD, monitoring, error tracking

**Recommended Paydown Strategy:**
Focus on high-impact, medium-effort items first (Service Provider, ESLint, Error Handling). These will provide immediate value and prevent debt from accumulating further.

---

## Conclusion

The Expense Tracker codebase demonstrates **solid fundamentals** with a clear architecture, good TypeScript usage, and thoughtful design decisions. The local-first approach with SQLite is appropriate, and the service layer is well-structured.

However, to scale effectively and maintain velocity as features are added, the team should address the **service instantiation anti-pattern** and **missing cross-cutting concerns** (error handling, logging, environment config).

The recommended refactorings are **non-breaking** and can be implemented incrementally without disrupting current development. Prioritize the P0 items in the next sprint to establish better patterns before Wave 3 features are added.

**Readiness for Production:** 🟡 With Improvements

The app is ready for MVP release after addressing:
- Error handling (P0)
- Logging (P0)
- Input sanitization (P1)
- Crash reporting (P1)

---

## Appendix: File Count Summary

```
Total Files: ~40
- Screens: 9
- Components: 2
- Services: 4
- Navigation: 6
- Tests: 11
- Config: 8
```

**Lines of Code (estimated):**
- Source: ~3,500 lines
- Tests: ~1,200 lines
- Config: ~200 lines
- Total: ~4,900 lines

**Test Coverage:** Estimated 60% (services well-covered, screens partially covered)

---

**Report Generated:** January 9, 2026
**Next Review Recommended:** After Wave 3 completion

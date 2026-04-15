# FinanceKit Integration (Wave 5.6)

## Overview

This document describes the FinanceKit transaction import feature implemented in Wave 5.6. FinanceKit allows users to import real transactions from Apple Card, Apple Cash, Savings, and participating banks directly into the Expense Tracker app.

## Features

- **Availability Detection**: Automatically checks if FinanceKit is available (iOS 17.4+ in US, iOS 18.4+ in UK)
- **Authorization Management**: Handles user permission requests for accessing financial data
- **Transaction Fetching**: Uses Apple's transaction picker UI to let users select transactions
- **Smart Conversion**: Maps FinanceKit transactions to Expense format with category suggestions
- **Duplicate Detection**: Prevents importing the same transaction twice
- **Batch Import**: Imports multiple transactions efficiently with error handling
- **Graceful Degradation**: Works seamlessly on unsupported platforms/versions

## Requirements

### Apple Requirements

1. **iOS Version**:
   - United States: iOS 17.4 or later
   - United Kingdom: iOS 18.4 or later

2. **App Store Requirements**:
   - App must be listed in the Finance category in App Store Connect
   - Must be distributed through the App Store for iPhone
   - Must provide financial management tools (budgeting, spending trends, etc.)

3. **Entitlement**:
   - Requires FinanceKit entitlement from Apple Developer Program
   - Entitlements are granted per bundle ID
   - Must be requested by Account Holder

### Technical Requirements

1. **Package**: `expo-finance-kit@^0.2.23`
   ```bash
   npm install expo-finance-kit
   ```

2. **Development Build**: FinanceKit requires a development build (won't work in Expo Go)
   ```bash
   eas build --profile development --platform ios
   ```

3. **Capabilities**: Add FinanceKit capability in app.json/app.config.js

## Architecture

### Service Layer

**FinanceKitService** (`src/services/FinanceKitService.ts`)
- Singleton pattern for consistent state management
- Handles all FinanceKit operations
- Provides graceful degradation

### Type Definitions

**Types** (`src/types/index.ts`)
```typescript
interface FinanceKitTransaction {
  id: string;
  amount: number;
  date: Date;
  merchantName?: string;
  category?: string;
  transactionDescription?: string;
  accountId?: string;
  status: 'posted' | 'pending';
}

interface FinanceKitImportResult {
  success: boolean;
  importedCount: number;
  skippedCount: number;
  errors: string[];
  expenseIds: number[];
}

interface FinanceKitAvailability {
  isAvailable: boolean;
  reason?: 'unsupported_ios' | 'not_installed' | 'no_entitlement' | 'not_us_or_uk';
  message?: string;
}
```

## Usage

### Basic Import Workflow

```typescript
import { FinanceKitService } from '@/services';

const financeKitService = FinanceKitService.getInstance();

// Complete import workflow (checks availability, requests auth, fetches, imports)
try {
  const result = await financeKitService.importTransactionsWorkflow(30); // Last 30 days

  if (result.success) {
    console.log(`Imported ${result.importedCount} transactions`);
    console.log(`Skipped ${result.skippedCount} transactions`);
  } else {
    console.error(`Import had errors:`, result.errors);
  }
} catch (error) {
  console.error('Import failed:', error);
}
```

### Check Availability

```typescript
const availability = await financeKitService.checkAvailability();

if (!availability.isAvailable) {
  // Hide feature or show message
  console.log(availability.message);
  // Example: "FinanceKit requires iOS 17.4 or later"
}
```

### Manual Control

```typescript
// 1. Check availability
const availability = await financeKitService.checkAvailability();
if (!availability.isAvailable) {
  throw new Error(availability.message);
}

// 2. Request authorization
const granted = await financeKitService.requestAuthorization();
if (!granted) {
  throw new Error('User denied access');
}

// 3. Fetch transactions (shows Apple's picker UI)
const transactions = await financeKitService.fetchTransactions(60); // Last 60 days

// 4. Import selected transactions
const result = await financeKitService.importTransactions(transactions);
```

## Implementation Details

### Transaction Conversion

FinanceKit transactions are automatically converted to Expense format:

| FinanceKit Field | Expense Field | Notes |
|------------------|---------------|-------|
| `amount` | `amount` | Converted to positive (absolute value) |
| `date` | `date` | Converted to YYYY-MM-DD format |
| `merchantName` | `merchant` | Used as merchant name |
| `category` | `category` | Falls back to "Uncategorized" |
| `transactionDescription` | `description` | Transaction details |
| N/A | `payment_method` | Always set to "Credit" |

### Duplicate Detection

The service prevents duplicate imports by checking:
1. **Amount Match**: Within $0.01
2. **Merchant Match**: Case-insensitive comparison
3. **Date Match**: Within 1 day (86400000 milliseconds)

If all three match an existing expense, the transaction is skipped.

### Error Handling

The service handles errors gracefully:
- **Platform Errors**: Returns clear messages for unsupported platforms
- **Authorization Errors**: Throws meaningful errors for denied access
- **Import Errors**: Collects individual transaction errors without stopping batch import
- **Network Errors**: Handles native module failures gracefully

## Testing

### Unit Tests

Located in `src/__tests__/services/FinanceKitService.test.ts`

Run tests:
```bash
npm test src/__tests__/services/FinanceKitService.test.ts
```

Test coverage includes:
- ✅ Availability checks (platform, module, iOS version)
- ✅ Authorization (request, status, denial)
- ✅ Transaction fetching
- ✅ Transaction conversion
- ✅ Import logic (success, duplicates, errors)
- ✅ Complete workflow
- ✅ Singleton pattern

### Manual Testing

1. **Check Availability** (iOS Simulator won't work - needs real device with iOS 17.4+)
   ```typescript
   const availability = await financeKitService.checkAvailability();
   console.log(availability);
   ```

2. **Request Authorization**
   - Should show system permission dialog
   - Test both grant and deny scenarios

3. **Import Transactions**
   - Should show Apple's transaction picker UI
   - Select various transactions
   - Verify they appear in expense list

4. **Duplicate Detection**
   - Import same transactions twice
   - Verify second import skips duplicates

## Limitations

### Current Limitations

1. **US & UK Only**: FinanceKit is only available in United States and United Kingdom
2. **Apple Card Only (US)**: At launch, primarily supports Apple Card, Apple Cash, and Savings
3. **iOS 17.4+ Required**: Older iOS versions cannot use this feature
4. **App Store Required**: Must be distributed through App Store with Finance category
5. **Entitlement Required**: Needs approval from Apple Developer Program

### Future Considerations

1. **Category Mapping**: Could add custom mapping for FinanceKit categories to IRS categories
2. **Bank Selection**: When more banks supported, add UI for selecting specific accounts
3. **Automatic Sync**: Could add background sync for automatic transaction updates
4. **Receipt Matching**: Could match imported transactions with existing receipts

## Security & Privacy

- **Local Storage**: All transaction data stays on device in SQLite
- **No Server Communication**: FinanceKit data never leaves the device
- **User Control**: Users explicitly select which transactions to import
- **Permissions**: Requires explicit user authorization
- **Data Protection**: Uses iOS Data Protection for secure storage

## Resources

- [Apple FinanceKit Documentation](https://developer.apple.com/financekit/)
- [expo-finance-kit Package](https://www.npmjs.com/package/expo-finance-kit)
- [WWDC24: Meet FinanceKit](https://developer.apple.com/videos/play/wwdc2024/2023/)
- [FinanceKit Requirements](https://developer.apple.com/financekit/)

## Support

For issues related to:
- **FinanceKit API**: Contact Apple Developer Support
- **expo-finance-kit**: File issue on GitHub repository
- **Expense Tracker Integration**: Check service logs and availability status

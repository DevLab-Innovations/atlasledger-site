# Expense Tracker - Comprehensive Test Plan

**Version:** 1.0
**Date:** January 13, 2026
**Coverage:** Waves 1-4 (Foundation through Receipt Magic)

---

## Executive Summary

This test plan covers all completed features from Waves 1-4 of the Expense Tracker app. It includes unit tests, integration tests, component tests, and E2E tests targeting iOS simulator.

**Current Test Status:**
- ✅ Unit Tests: 27 passing (OCR service)
- ✅ Component Tests: Existing tests for various components
- 🔨 E2E Tests: To be implemented
- 🔨 UI Automation Tests: To be implemented

---

## Test Scope

### Wave 1: Foundation ✅
- Database initialization and migrations
- Table creation (expenses, mileage, categories, merchant_categories)
- File system setup (receipts, voice directories)
- Navigation structure (4 bottom tabs)
- Screen scaffolding

### Wave 2: Core Functionality ✅
- **Expense Management:**
  - Add expense with validation
  - Edit expense
  - Delete expense with confirmation
  - List view with date grouping
  - Swipe-to-delete gesture
  - Pull-to-refresh
  - Empty state

- **Mileage Tracking:**
  - Add mileage entry
  - Real-time rate calculation
  - Edit/delete mileage
  - Custom rate configuration

- **Settings:**
  - Mileage rate settings
  - Currency preference
  - About section

### Wave 3: Search & Filter ✅
- Real-time search (debounced)
- Filter modal with:
  - Date range presets & custom
  - Category multi-select
  - Payment method filter
  - Amount range filter
  - Has receipt toggle
- Filter chips (removable)
- Active filter count badge

- **Category Management:**
  - View all categories with counts
  - Add custom category
  - Edit custom category
  - Delete with reassignment
  - IRS default protection

### Wave 4: Receipt Magic ✅
- Camera permissions
- Camera capture
- Gallery picker
- Image compression
- Receipt storage
- Thumbnail display
- Full-screen viewer with zoom
- OCR text extraction
- Form pre-filling
- Confidence scoring

---

## Test Strategy

### 1. Unit Tests
**Target:** Individual functions and utilities
**Framework:** Jest
**Status:** ✅ Implemented for OCR service (27 tests)

**Coverage:**
- OCR amount extraction
- OCR date extraction
- OCR merchant extraction
- Confidence scoring
- Date grouping utilities
- Currency formatting
- Input validation

### 2. Integration Tests
**Target:** Service interactions and database operations
**Framework:** Jest with mocked services
**Status:** 🔨 To implement

**Coverage:**
- DatabaseService CRUD operations
- FileService with database updates
- OCRService with form pre-filling
- Navigation flows
- State management

### 3. Component Tests
**Target:** React components in isolation
**Framework:** React Native Testing Library
**Status:** ✅ Partially implemented

**Coverage:**
- ExpenseForm validation
- CategoryList interactions
- SearchBar functionality
- FilterModal behavior
- Receipt thumbnails
- Camera permissions UI

### 4. E2E Tests
**Target:** Complete user workflows
**Framework:** Detox (iOS Simulator)
**Status:** 🔨 To implement

**Coverage:**
- Critical user journeys
- Cross-screen workflows
- Data persistence
- Error handling
- Edge cases

---

## Critical User Flows (E2E Priority)

### Priority 1: Core Expense Management 🔴

**Flow 1.1: Add New Expense - Happy Path**
```
Given app is launched
When user navigates to Expenses tab
And taps "Add Expense" FAB
And enters amount "$45.67"
And selects date "Today"
And selects category "Meals"
And enters merchant "Starbucks"
And taps "Save"
Then expense appears in list
And amount is "$45.67"
And category is "Meals"
```

**Flow 1.2: Add Expense - Validation**
```
Given user is on Add Expense screen
When user taps "Save" without entering amount
Then validation error appears "Amount is required"
When user enters amount "0"
Then validation error appears "Amount must be greater than 0"
When user selects future date
Then validation error appears "Date cannot be in the future"
```

**Flow 1.3: Edit Expense**
```
Given expense exists in list
When user taps on expense
And taps "Edit Expense"
And changes amount to "$50.00"
And taps "Save"
Then expense list shows updated amount "$50.00"
```

**Flow 1.4: Delete Expense**
```
Given expense exists in list
When user swipes expense left
And taps "Delete"
Then confirmation dialog appears
When user confirms "Delete"
Then expense is removed from list
```

### Priority 2: Search and Filter 🟡

**Flow 2.1: Search Expenses**
```
Given multiple expenses exist
When user types "coffee" in search bar
Then list shows only expenses matching "coffee"
When user clears search
Then all expenses appear again
```

**Flow 2.2: Filter by Date Range**
```
Given expenses from multiple months exist
When user opens filter modal
And selects "This Month"
And applies filters
Then only current month expenses shown
And filter badge shows "1 filter"
```

**Flow 2.3: Multi-Filter Combination**
```
Given user opens filter modal
When user selects category "Meals"
And selects payment method "Credit"
And selects date range "This Week"
And applies filters
Then only matching expenses shown
And filter badge shows "3 filters"
When user taps filter chip to remove
Then that filter is removed
```

### Priority 3: Category Management 🟡

**Flow 3.1: Add Custom Category**
```
Given user navigates to Settings > Categories
When user taps "Add Category"
And enters name "Client Meals"
And taps "Save"
Then new category appears in list
And shows "0 expenses"
```

**Flow 3.2: Edit Custom Category**
```
Given custom category "Client Meals" exists
When user taps edit icon
And changes name to "Client Dinners"
And saves
Then category name is updated everywhere
```

**Flow 3.3: Delete Category with Reassignment**
```
Given custom category has 5 expenses
When user attempts to delete
Then reassignment modal appears
When user selects new category "Meals"
And confirms
Then category is deleted
And expenses are reassigned
```

### Priority 4: Receipt Magic 🔴

**Flow 4.1: Add Receipt - Camera**
```
Given user is editing an expense
When user taps "Add Receipt"
And grants camera permission
And takes photo
Then OCR processing begins
And preview screen shows extracted data
When user taps "Use This Data"
Then form pre-fills with amount, date, merchant
And green banner shows confidence score
```

**Flow 4.2: Add Receipt - Gallery**
```
Given user is on camera screen
When user taps gallery icon
And selects image from library
Then OCR processes image
And preview shows results
```

**Flow 4.3: View Receipt Full-Screen**
```
Given expense has receipt attached
When user views expense detail
And taps receipt thumbnail
Then full-screen viewer opens
When user pinches to zoom
Then image zooms in/out
When user double-taps
Then image toggles 2x zoom
```

**Flow 4.4: Delete Receipt**
```
Given expense has receipt in full-screen viewer
When user taps delete icon
Then confirmation appears
When user confirms
Then receipt is deleted
And thumbnail disappears
And receipt count updates
```

### Priority 5: Mileage Tracking 🟢

**Flow 5.1: Add Mileage Entry**
```
Given user navigates to Mileage tab
When user taps "Add Mileage" FAB
And enters miles "25.5"
And enters start location "Home"
And enters end location "Office"
And enters purpose "Client meeting"
Then reimbursement calculates in real-time
When user taps "Save"
Then mileage appears in list
```

---

## Test Data Requirements

### Seed Data
```javascript
// IRS Categories (22 default)
- Advertising
- Car and Truck Expenses
- Commissions and Fees
- Contract Labor
... (all 22 IRS categories)

// Test Expenses
const testExpenses = [
  { amount: 45.67, date: 'today', category: 'Meals', merchant: 'Starbucks' },
  { amount: 123.45, date: 'yesterday', category: 'Office Supplies', merchant: 'Staples' },
  { amount: 67.89, date: '2024-01-01', category: 'Travel', merchant: 'Uber' },
  // ... more test data
];

// Test Images for OCR
- Clear Starbucks receipt (expected: high confidence)
- Blurry receipt (expected: low confidence)
- Partial receipt (expected: medium confidence)
```

### Test User Scenarios
1. **New User:** First launch, onboarding, no data
2. **Active User:** 50+ expenses, multiple categories
3. **Heavy User:** 500+ expenses, performance testing

---

## Test Environment

### Simulator Configuration
```
Device: iPhone 15 Pro
iOS: 17.0+
Expo SDK: 52.0.0
Node: 18+
```

### Required Permissions
- Camera access
- Photo library access
- File system access

---

## Test Execution Plan

### Phase 1: Setup (Day 1)
- [ ] Install Detox
- [ ] Configure iOS simulator
- [ ] Set up test database
- [ ] Create test fixtures
- [ ] Write helper functions

### Phase 2: Core Tests (Days 2-3)
- [ ] Expense CRUD tests
- [ ] Navigation tests
- [ ] Search and filter tests
- [ ] Category management tests

### Phase 3: Advanced Tests (Days 4-5)
- [ ] Receipt capture flow
- [ ] OCR integration tests
- [ ] Image viewer tests
- [ ] Mileage tracking tests

### Phase 4: Edge Cases (Day 6)
- [ ] Validation tests
- [ ] Error handling
- [ ] Offline behavior
- [ ] Performance tests

### Phase 5: Reporting (Day 7)
- [ ] Test execution
- [ ] Bug reporting
- [ ] Coverage analysis
- [ ] Documentation

---

## Success Criteria

### Test Coverage Goals
- ✅ Unit Test Coverage: >80%
- 🎯 Integration Test Coverage: >70%
- 🎯 E2E Test Coverage: 100% of critical flows
- 🎯 Component Test Coverage: >60%

### Quality Gates
- All critical flows (Priority 1 & 4) must pass
- No blocking bugs in core functionality
- Performance: Expense list renders <1s for 500 items
- OCR: >70% accuracy on clear receipts

### Exit Criteria
- All P1 tests passing
- 90%+ of P2 tests passing
- Known issues documented
- Test automation integrated into CI/CD

---

## Known Limitations

1. **OCR Testing:** Requires real image processing, may vary by device
2. **Camera Testing:** Limited in simulator, needs device testing
3. **Performance:** Simulator may not reflect device performance
4. **Gestures:** Some gestures harder to automate (pinch-to-zoom)

---

## Bug Reporting Template

```markdown
## Bug Report

**ID:** BUG-001
**Title:** [Short description]
**Severity:** Critical | High | Medium | Low
**Priority:** P1 | P2 | P3 | P4

**Steps to Reproduce:**
1.
2.
3.

**Expected Result:**

**Actual Result:**

**Environment:**
- Device: iPhone 15 Pro Simulator
- iOS: 17.0
- App Version: 1.0.0

**Screenshots/Logs:**

**Related Test Case:**
```

---

## Next Steps

1. Install and configure Detox
2. Create E2E test suite structure
3. Implement Priority 1 tests (Core Expense Management)
4. Implement Priority 4 tests (Receipt Magic)
5. Run full test suite on iOS simulator
6. Generate test coverage report
7. Document findings and bugs

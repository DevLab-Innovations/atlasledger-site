# QA Report: Accountant Sharing Redesign (PRs #238 and #239)

**Date**: 2026-03-13
**Tester**: qa-tester agent
**Branch**: dev
**PRs**: #238 (feature/accountant-sharing-service), #239 (feature/accountant-sharing-ui)

---

## Results Overview

- Total Scenarios: 25
- Passed: 23
- Failed: 0
- Advisory (no unit test coverage): 2

---

## Automated Tests

**Command**: `npx jest --no-coverage`

**Result**: PASS

- Test Suites: 181 passed, 1 skipped (pre-existing skip), 182 total
- Tests: 3746 passed, 55 skipped (pre-existing), 3801 total
- No regressions introduced. All previously passing suites continue to pass.

Targeted suites confirmed passing:
- `src/__tests__/services/SubscriptionService.test.ts`
- `src/__tests__/services/SubscriptionService.pro.test.ts`
- `src/__tests__/services/SubscriptionService.upgrade.test.ts`
- `src/__tests__/services/SubscriptionService.team.test.ts`
- `src/__tests__/services/WebShareService.test.ts`
- `src/__tests__/screens/PaywallScreen.test.ts`

---

## Scenario Results

### 1. Access Gate Logic

| Scenario | Expected | Actual | Result |
|---|---|---|---|
| Free tier + personal account: `hasAccountantSharingAccess` | `false` | Returns `false` — only enters `isTierSufficientFor('free', 'premium')` which is `false` | PASS |
| Free tier + business account: `hasAccountantSharingAccess` | `true` | Returns `true` immediately (short-circuit on `accountType === 'business'`) | PASS |
| Premium + personal: `hasAccountantSharingAccess` | `true` | `isTierSufficientFor('premium', 'premium')` = `true` | PASS |
| Pro + any: `hasAccountantSharingAccess` | `true` | `isTierSufficientFor('pro', 'premium')` = `true` | PASS |

All four gate logic scenarios verified correct by code inspection of `SubscriptionService.hasAccountantSharingAccess()` (lines 952–956).

### 2. Share Link Limits

| Scenario | Expected | Actual | Result |
|---|---|---|---|
| `getShareLinkLimit('free', 'personal')` | `0` | Falls through to `SHARE_LINK_LIMITS['free']` = `0` | PASS |
| `getShareLinkLimit('free', 'business')` | `3` | Short-circuit returns `FREE_BUSINESS_LINK_LIMIT` = `3` | PASS |
| `getShareLinkLimit('premium', 'personal')` | `10` | Falls through to `SHARE_LINK_LIMITS['premium']` = `10` | PASS |
| `getShareLinkLimit('pro', 'personal')` | `Infinity` | Falls through to `SHARE_LINK_LIMITS['pro']` = `Infinity` | PASS |

Verified in both `src/utils/shareLimits.ts` and `src/services/SubscriptionService.ts` — both independently implement the same logic correctly.

**Edge case noted**: `SHARE_LINK_MAX_EXPIRY_DAYS['free']` = `30` in `src/types/index.ts`. The `getShareLinkMaxExpiryDays('free', 'personal')` fallthrough path returns `30`, which is the same value as the free+business short-circuit. This means free personal accounts (who are blocked from creating links at the gate) would get a 30-day max expiry if they somehow bypassed the gate. The gate blocks them before expiry is relevant, so this is not a runtime bug — but the type-level comment says free personal gets 0, yet the constant says 30. This is a documentation inconsistency, not a code defect.

### 3. WebShareService Limit Enforcement

| Scenario | Expected | Actual | Result |
|---|---|---|---|
| Generating a share when at limit throws with descriptive error | `Error: "Share link limit reached (N). Revoke..."` | `generateShare` calls `getShareLinkLimit`, filters non-expired shares, throws `Error('Share link limit reached (${limit})...')` | PASS |
| Expired links not counted against limit | Only `expiresAt > now` shares count | `activeNonExpired = activeShares.filter((s) => new Date(s.expiresAt) > new Date())` | PASS |
| `revokeAll` settles revocations independently | Each revocation independent — failures don't abort others | Uses `Promise.allSettled(...)` — confirmed | PASS |

Limit guard at `WebShareService.generateShare()` lines 442–451. Handles `Infinity` correctly: a finite count is never `>= Infinity`.

### 4. UI Correctness

| Scenario | Expected | Actual | Result |
|---|---|---|---|
| `src/utils/shareLimits.ts` exists and exports `getShareLinkLimit` and `getShareLinkMaxExpiryDays` | Both exported | Both exported (lines 24, 36) | PASS |
| ShareScreen uses `useFocusEffect` not `useEffect` | `useFocusEffect` | `useFocusEffect` imported from `@react-navigation/native` and used at line 111 | PASS |
| `atLimit` in ActiveSharesScreen uses `shareLimit !== Infinity` guard | `shareLimit !== Infinity && ...` | Line 201: `const atLimit = shareLimit !== Infinity && activeShares.length >= shareLimit;` | PASS |
| `atLimit` in ShareScreen uses `shareLimit !== Infinity` guard | `shareLimit !== Infinity && ...` | Line 133: `const atLimit = shareLimit !== Infinity && activeShareCount >= shareLimit;` | PASS |
| No hardcoded `#FFFFFF` in ActiveSharesScreen | No `#FFFFFF` | No matches found — uses `colors.surface` and `colors.onPrimary` semantic tokens | PASS |
| BusinessHint microcopy in CreateAccountScreen | `"Business · Unlocks Accountant Sharing for free"` | Line 185: present | PASS |
| BusinessHint microcopy in EditAccountScreen | `"Business · Unlocks Accountant Sharing for free"` | Line 271: present | PASS |
| BusinessHint microcopy in OnboardingCreateAccountScreen | `"Business · Unlocks Accountant Sharing for free"` | Line 268: present | PASS |
| FeatureGate `accountant_sharing` uses dynamic `Unlock ${tierName}` | Dynamic string, no hardcode | Line 106: `` {`Unlock ${tierName}`} `` — `tierName` comes from `getTierDisplayName(requiredTier)` | PASS |
| PaywallScreen FEATURE_MATRIX has `accountant_sharing` row with `free: 'Business acct'` | `{ feature: 'Accountant sharing', free: 'Business acct', premium: true, pro: true }` | Line 141: confirmed | PASS |
| `ADDON_FEATURES` is empty array | `[] as const` | `src/types/index.ts` line 744: `export const ADDON_FEATURES = [] as const;` | PASS |

### 5. Regression — Existing Tests

| Area | Result |
|---|---|
| SubscriptionService (all test files) | PASS — 179 tests |
| WebShareService | PASS — all 18+ tests |
| PaywallScreen | PASS |
| Full suite (181 suites / 3746 tests) | PASS |

---

## Advisory Findings (Not Blocking)

### ADV-1: No Unit Tests for New SubscriptionService Methods

**Severity**: Low
**Component**: `src/services/SubscriptionService.ts`

The three new methods — `hasAccountantSharingAccess()`, `getShareLinkLimit()`, and `getShareLinkMaxExpiryDays()` — have no dedicated unit tests in `src/__tests__/services/SubscriptionService.test.ts` or any related file. The logic is verified correct by code inspection and is covered indirectly by the UI integration, but isolated regression coverage is missing.

**Recommendation**: Add unit tests for these three methods across the four tier × account-type combinations (8 scenarios).

### ADV-2: No Unit Tests for `revokeAll` in WebShareService

**Severity**: Low
**Component**: `src/services/WebShareService.ts`

`revokeAll()` (lines 548–552) uses `Promise.allSettled` correctly, but has no unit test verifying that (a) all revocations are attempted even when one fails, and (b) partial failure is tolerated. The `revokeShare` base method has good test coverage, but `revokeAll` as an orchestrator is untested.

**Recommendation**: Add a test with one failing and one succeeding revocation to confirm independent settlement.

---

## Summary

All 23 functional scenarios pass. No blocking issues found. The two advisory findings are low-severity testing gaps that should be addressed in a follow-up PR — they do not block this release.

The code correctly:
- Gates free personal accounts at the link count level (limit = 0, `atLimit` fires immediately)
- Grants free business accounts 3 links with 30-day expiry
- Grants premium+ personal accounts the per-tier limits
- Uses `Promise.allSettled` for independent revocation (no aborts on partial failure)
- Uses `useFocusEffect` (not `useEffect`) for share state refresh
- Uses semantic color tokens (not `#FFFFFF`) throughout ActiveSharesScreen
- Has "Business · Unlocks Accountant Sharing for free" microcopy in all 3 account creation screens
- Has dynamic `Unlock ${tierName}` in FeatureGate (not hardcoded)
- PaywallScreen FEATURE_MATRIX correctly shows `'Business acct'` for free tier accountant sharing
- `ADDON_FEATURES` is an empty array — the old $1.99/mo add-on model is fully removed

---

## Recommendation

**[x] Ready for promotion to `main`**

The accountant sharing redesign is functionally correct and all automated tests pass. The two advisory items (unit test gaps) are recommended for a follow-up and do not block release.

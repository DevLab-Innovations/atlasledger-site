# iCloud Sync Architecture

> **Status**: PROPOSED - Not yet implemented
> **Last Updated**: 2026-01-21
> **Related**: Premium 1.1+ roadmap

## Overview

This document outlines the architecture for adding iCloud sync to the Atlas expense tracker app. The goal is to enable seamless data synchronization across a user's devices while maintaining the app's local-first, offline-capable nature.

## Current Architecture

```
┌─────────────────────────────────────┐
│           iOS Device                │
│  ┌───────────────────────────────┐  │
│  │      SQLite Database          │  │
│  │  (Local Source of Truth)      │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │      SecureStore              │  │
│  │  (Subscription Status)        │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Current State:**
- All data stored locally in SQLite
- Subscription status cached in Keychain (via SecureStore)
- No cross-device sync
- Data lost if device is lost/reset without backup

## Proposed iCloud Sync Architecture

### Option A: CloudKit (Recommended)

```
┌─────────────────────────────────────┐
│           iOS Device 1              │
│  ┌───────────────────────────────┐  │
│  │      SQLite Database          │  │
│  │  (Local Source of Truth)      │  │
│  └─────────────┬─────────────────┘  │
│                │                    │
│  ┌─────────────▼─────────────────┐  │
│  │      Sync Engine              │  │
│  │  (Change Tracking + Queue)    │  │
│  └─────────────┬─────────────────┘  │
└────────────────┼────────────────────┘
                 │
         ┌───────▼───────┐
         │   CloudKit    │
         │  Private DB   │
         └───────┬───────┘
                 │
┌────────────────┼────────────────────┐
│  ┌─────────────▼─────────────────┐  │
│  │      Sync Engine              │  │
│  └─────────────┬─────────────────┘  │
│  ┌─────────────▼─────────────────┐  │
│  │      SQLite Database          │  │
│  │  (Local Source of Truth)      │  │
│  └───────────────────────────────┘  │
│           iOS Device 2              │
└─────────────────────────────────────┘
```

**Why CloudKit:**
- Native Apple integration
- No server infrastructure to maintain
- Automatic iCloud account handling
- Built-in conflict detection
- Free tier generous for our use case
- Handles offline/online transitions

### Option B: Custom Backend (Future - Team Tier)

For Team tier with multi-user collaboration, a custom backend would be needed. This is out of scope for initial iCloud sync.

## Source of Truth Strategy

### Principle: Local-First, Cloud-Backed

```
┌─────────────────────────────────────────────────────────────┐
│                    SOURCE OF TRUTH                          │
├─────────────────────────────────────────────────────────────┤
│  LOCAL SQLite is ALWAYS the source of truth for the device │
│  CloudKit is a synchronization layer, not the source       │
└─────────────────────────────────────────────────────────────┘
```

**Rationale:**
1. App works fully offline
2. No network latency for local operations
3. User data available immediately on app launch
4. CloudKit syncs in background when connected

### Data Flow

```
User Action → Local SQLite → Sync Queue → CloudKit → Other Devices
                   ↑                           │
                   └───────────────────────────┘
                        (Background Sync)
```

## Conflict Resolution

### Strategy: Last-Write-Wins with Merge

| Conflict Type | Resolution Strategy |
|---------------|---------------------|
| Same record modified on 2 devices | Last modification timestamp wins |
| Record deleted on one, modified on other | Deletion wins (with grace period) |
| New records on multiple devices | Both kept (no conflict) |
| Category/Account references | Validate foreign keys, orphan handling |

### Implementation

```typescript
interface SyncableRecord {
  id: string;
  created_at: string;      // ISO timestamp
  updated_at: string;      // ISO timestamp
  deleted_at: string | null; // Soft delete
  device_id: string;       // Origin device
  sync_version: number;    // Incrementing version
}

function resolveConflict(local: SyncableRecord, remote: SyncableRecord): SyncableRecord {
  // Deletion always wins (within 30-day grace period)
  if (remote.deleted_at && !local.deleted_at) {
    return remote;
  }
  if (local.deleted_at && !remote.deleted_at) {
    return local;
  }

  // Last-write-wins based on updated_at
  const localTime = new Date(local.updated_at).getTime();
  const remoteTime = new Date(remote.updated_at).getTime();

  return localTime >= remoteTime ? local : remote;
}
```

### Conflict Notification

For important conflicts (e.g., account settings), notify user:

```typescript
interface ConflictNotification {
  recordType: 'expense' | 'income' | 'bill' | 'account';
  localValue: string;
  remoteValue: string;
  resolvedTo: 'local' | 'remote';
  timestamp: string;
}
```

## Reinstall / New Device Handling

### Scenario 1: Fresh Install with Existing iCloud Data

```
┌──────────────────────────────────────────────────────────┐
│                    FIRST LAUNCH FLOW                      │
├──────────────────────────────────────────────────────────┤
│  1. App launches                                          │
│  2. Check CloudKit for existing data                      │
│  3. If data exists:                                       │
│     → Show "Restore from iCloud?" prompt                  │
│     → User confirms → Download all records                │
│     → User declines → Start fresh (warn about overwrite)  │
│  4. If no data:                                           │
│     → Normal onboarding                                   │
│     → Enable sync for new data                            │
└──────────────────────────────────────────────────────────┘
```

### Scenario 2: Reinstall on Same Device

```
┌──────────────────────────────────────────────────────────┐
│                   REINSTALL FLOW                          │
├──────────────────────────────────────────────────────────┤
│  1. App launches, detects empty local DB                  │
│  2. Check device ID in CloudKit metadata                  │
│  3. If same device ID found:                              │
│     → Auto-restore without prompt                         │
│     → Show "Data restored from iCloud"                    │
│  4. If different device ID:                               │
│     → Follow "Fresh Install" flow above                   │
└──────────────────────────────────────────────────────────┘
```

### Scenario 3: Device Lost/Stolen

```
┌──────────────────────────────────────────────────────────┐
│                   LOST DEVICE FLOW                        │
├──────────────────────────────────────────────────────────┤
│  1. User sets up new device with same Apple ID            │
│  2. Installs app                                          │
│  3. CloudKit data automatically available                 │
│  4. Restore prompt shown                                  │
│  5. All data recovered                                    │
│                                                           │
│  Security: If user removes device from Apple ID,          │
│  CloudKit access is revoked for that device               │
└──────────────────────────────────────────────────────────┘
```

## Data Migration Path

### From Local-Only to Sync-Enabled

```typescript
async function migrateToSync(): Promise<void> {
  // 1. Add sync columns to existing tables
  await db.exec(`
    ALTER TABLE expenses ADD COLUMN sync_version INTEGER DEFAULT 0;
    ALTER TABLE expenses ADD COLUMN device_id TEXT;
    ALTER TABLE expenses ADD COLUMN deleted_at TEXT;
  `);

  // 2. Generate device ID
  const deviceId = await getOrCreateDeviceId();

  // 3. Mark all existing records with device ID
  await db.exec(`
    UPDATE expenses SET device_id = ? WHERE device_id IS NULL
  `, [deviceId]);

  // 4. Initial upload to CloudKit
  await uploadAllRecords();

  // 5. Enable sync listeners
  await enableSyncListeners();
}
```

## Subscription Status Sync

Subscription status is **NOT synced via CloudKit**. Instead:

```
┌──────────────────────────────────────────────────────────┐
│              SUBSCRIPTION STATUS HANDLING                 │
├──────────────────────────────────────────────────────────┤
│  1. Subscription is tied to Apple ID, not device         │
│  2. On app launch: Query StoreKit for current status     │
│  3. Cache in SecureStore for offline access              │
│  4. Refresh periodically and on app foreground           │
│                                                           │
│  This ensures:                                            │
│  - Subscription works across all devices automatically    │
│  - No sync conflicts for subscription state               │
│  - Apple handles all subscription management              │
└──────────────────────────────────────────────────────────┘
```

## Implementation Phases

### Phase 1: Foundation (Premium 1.1)
- [ ] Add sync columns to schema
- [ ] Implement device ID generation
- [ ] Create CloudKit container and schema
- [ ] Build basic sync engine (upload only)

### Phase 2: Full Sync (Premium 1.2)
- [ ] Implement download/merge logic
- [ ] Add conflict resolution
- [ ] Build restore flow for new devices
- [ ] Add sync status UI indicator

### Phase 3: Polish (Premium 1.3)
- [ ] Optimize sync performance
- [ ] Add manual sync trigger
- [ ] Implement sync error handling/retry
- [ ] Add "last synced" indicator

## Privacy Considerations

```
┌──────────────────────────────────────────────────────────┐
│                   PRIVACY NOTES                           │
├──────────────────────────────────────────────────────────┤
│  - All data stored in user's PRIVATE CloudKit database   │
│  - Only accessible with user's Apple ID                  │
│  - We (the developer) cannot access user data            │
│  - Data encrypted in transit and at rest by Apple        │
│  - User can delete all data via iCloud settings          │
│                                                           │
│  Privacy Policy must disclose:                            │
│  - Data is synced to iCloud                              │
│  - User controls sync via device settings                │
│  - Data retention follows Apple's iCloud policies        │
└──────────────────────────────────────────────────────────┘
```

## Technical Requirements

### CloudKit Setup
1. Enable CloudKit capability in Xcode
2. Create CloudKit container: `iCloud.com.devlabinnovations.atlasledger`
3. Define record types matching SQLite schema
4. Configure subscriptions for push notifications

### Dependencies
- `@react-native-community/cloudkit` or native module
- Background fetch capability for sync
- Push notification capability for real-time sync

### Testing Strategy
1. Unit tests for conflict resolution logic
2. Integration tests with CloudKit sandbox
3. Multi-device manual testing
4. Offline/online transition testing
5. Large dataset sync performance testing

## Open Questions

1. **Sync Frequency**: Real-time vs periodic batch sync?
2. **Bandwidth**: Sync images/receipts or just metadata?
3. **Sharing**: How does this interact with Team tier sharing?
4. **Export**: Should cloud data be included in Excel exports?

---

*This document will be updated as the sync feature is implemented.*

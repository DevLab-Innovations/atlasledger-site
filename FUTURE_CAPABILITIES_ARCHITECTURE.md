# Future Capabilities Architecture

> Atlas Ledger Extended Features Roadmap

**Version:** 1.0
**Last Updated:** January 2026
**Status:** Planning

---

## Table of Contents

1. [Vision & Principles](#vision--principles)
2. [Feature Overview](#feature-overview)
3. [Tier Structure](#tier-structure)
4. [Technical Architecture](#technical-architecture)
5. [Feature Specifications](#feature-specifications)
6. [Privacy & Security](#privacy--security)
7. [Size Budget](#size-budget)
8. [Implementation Phases](#implementation-phases)
9. [Dependencies](#dependencies)

---

## Vision & Principles

### Core Philosophy
Atlas Ledger is a **local-first, privacy-focused** expense tracker. All future capabilities must adhere to:

1. **Privacy First** - Data stays on-device by default
2. **Lean Core** - Base app stays under 70MB
3. **Progressive Enhancement** - Advanced features load on-demand
4. **Native Maximization** - Use iOS APIs before third-party libraries
5. **Graceful Degradation** - Features work offline where possible

### Design Principles
- **Invisible Intelligence** - AI enhances, never interrupts
- **Voice as First-Class** - Full app control via voice
- **Contextual Awareness** - Right suggestion at right time
- **Delightful Progress** - Gamification motivates, never nags

---

## Feature Overview

### Priority Features (User-Selected)

| Feature | Category | Tier | Priority |
|---------|----------|------|----------|
| Voice Queries | Voice/AI | Premium | P0 |
| Conversational Entry | Voice/AI | Premium | P0 |
| Siri Shortcuts | Voice/AI | Premium | P1 |
| Smart Predictions | AI/ML | Premium | P0 |
| Document Wallet | Storage | Premium | P1 |
| Gamification | Engagement | Premium | P2 |
| AR Receipt Scan | Capture | Pro | P1 |
| Advanced Voice (Whisper) | Voice/AI | Pro | P2 |
| Conversational AI (Cloud) | AI/ML | Pro | P2 |
| Team Features | Collaboration | Teams | Future |

---

## Tier Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                         FREE TIER                                │
│  Basic expense tracking, categories, budgets, reports           │
│  Single account, manual entry, local storage                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     PREMIUM TIER ($6.99/mo)                      │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │ Voice Queries│ │Conversational│ │    Smart     │            │
│  │  "How much   │ │   Entry      │ │ Predictions  │            │
│  │   on food?"  │ │ "Coffee $5"  │ │  Time/Place  │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │    Siri      │ │  Document    │ │ Gamification │            │
│  │  Shortcuts   │ │   Wallet     │ │   Streaks    │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │   Native     │ │  Spending    │ │  Multiple    │            │
│  │    Speech    │ │  Forecasts   │ │  Accounts    │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      PRO TIER ($14.99/mo)                        │
│                                                                  │
│  Everything in Premium, plus:                                    │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │  AR Receipt  │ │   Whisper    │ │ Advanced AI  │            │
│  │    Scan      │ │  (On-demand) │ │  Insights    │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │   E2E Sync   │ │  Priority    │ │   Tax        │            │
│  │ Multi-device │ │   Support    │ │ Optimization │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    TEAMS TIER (Future)                           │
│                                                                  │
│  Everything in Pro, plus:                                        │
│  - Shared budgets & expenses                                     │
│  - Approval workflows                                            │
│  - Team analytics                                                │
│  - Admin controls                                                │
│  - Accountant access                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Technical Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           PRESENTATION LAYER                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │   Screens   │ │  Widgets    │ │   Voice     │ │     AR      │       │
│  │   & Forms   │ │ Dashboard   │ │   Interface │ │   Overlay   │       │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          INTELLIGENCE LAYER                              │
│  ┌─────────────────────────────────────────────────────────────┐       │
│  │                    VoiceIntelligenceService                  │       │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐   │       │
│  │  │  Speech   │ │   NLP     │ │  Intent   │ │  Query    │   │       │
│  │  │Recognition│ │  Parser   │ │  Router   │ │  Engine   │   │       │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────┘   │       │
│  └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────┐       │
│  │                    PredictionService                         │       │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐   │       │
│  │  │  Context  │ │  Pattern  │ │  Anomaly  │ │ Suggestion│   │       │
│  │  │  Analyzer │ │  Matcher  │ │  Detector │ │  Ranker   │   │       │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────┘   │       │
│  └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────┐       │
│  │                    GamificationService                       │       │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐   │       │
│  │  │  Streak   │ │Achievement│ │ Challenge │ │  Reward   │   │       │
│  │  │  Tracker  │ │  Engine   │ │  Manager  │ │  System   │   │       │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────┘   │       │
│  └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            SERVICE LAYER                                 │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐│
│  │  Expense  │ │  Budget   │ │  Document │ │   Sync    │ │  Export   ││
│  │  Service  │ │  Service  │ │  Service  │ │  Service  │ │  Service  ││
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘│
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                             DATA LAYER                                   │
│  ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────┐   │
│  │      SQLite         │ │    Secure Store     │ │   File System   │   │
│  │   (Expenses, etc)   │ │  (Keys, Auth)       │ │ (Receipts, Docs)│   │
│  └─────────────────────┘ └─────────────────────┘ └─────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         NATIVE PLATFORM LAYER                            │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐│
│  │ SFSpeech  │ │  ARKit    │ │  Vision   │ │  Core ML  │ │ HealthKit ││
│  │Recognizer │ │           │ │           │ │           │ │           ││
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘│
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐│
│  │   Siri    │ │ CoreLoc   │ │ LocalAuth │ │  EventKit │ │  Contacts ││
│  │ Shortcuts │ │ ation     │ │           │ │           │ │           ││
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### On-Demand Module Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     MODULE REGISTRY                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ModuleManager.ts                                        │   │
│  │  - isModuleAvailable(moduleId): boolean                  │   │
│  │  - downloadModule(moduleId): Promise<void>               │   │
│  │  - getModuleSize(moduleId): number                       │   │
│  │  - unloadModule(moduleId): void                          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ CORE MODULE   │    │ WHISPER       │    │ ADVANCED AI   │
│ (Always       │    │ MODULE        │    │ MODULE        │
│  bundled)     │    │ (On-demand)   │    │ (Cloud)       │
├───────────────┤    ├───────────────┤    ├───────────────┤
│ - Basic UI    │    │ - whisper.rn  │    │ - E2E encrypt │
│ - SQLite      │    │ - Tiny model  │    │ - API client  │
│ - SFSpeech    │    │ - ~40MB       │    │ - Response    │
│ - Core ML     │    │               │    │   parser      │
│ - ~50MB       │    │               │    │ - ~2MB        │
└───────────────┘    └───────────────┘    └───────────────┘
```

---

## Feature Specifications

### 1. Voice Queries (Premium)

**Description:** Natural language queries about spending data.

**Examples:**
- "How much did I spend on food this month?"
- "What's my biggest expense category?"
- "Show me all Uber rides"
- "Am I over budget?"

**Technical Approach:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Voice     │───▶│   Speech    │───▶│   Intent    │───▶│   Query     │
│   Input     │    │Recognition  │    │   Parser    │    │   Engine    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                         │                   │                   │
                    SFSpeech           NLP Patterns         SQLite
                   Recognizer          (on-device)          Queries
```

**Implementation:**
```typescript
// src/services/VoiceQueryService.ts

interface QueryIntent {
  type: 'sum' | 'list' | 'compare' | 'status';
  category?: string;
  merchant?: string;
  dateRange: DateRange;
  aggregation?: 'total' | 'average' | 'count';
}

interface QueryResult {
  spoken: string;      // TTS response
  visual?: ReactNode;  // Optional UI
  data?: unknown;      // Raw data for display
}

class VoiceQueryService {
  async processQuery(transcript: string): Promise<QueryResult>;
  private parseIntent(transcript: string): QueryIntent;
  private executeQuery(intent: QueryIntent): Promise<unknown>;
  private formatResponse(intent: QueryIntent, data: unknown): QueryResult;
}
```

**Intent Patterns (On-Device NLP):**
```typescript
const QUERY_PATTERNS = {
  // Sum queries
  'how much (did I|have I) (spend|spent) on {category}': 'sum_category',
  'what (did I|have I) (spend|spent) (at|on) {merchant}': 'sum_merchant',
  'total (spending|expenses) (this|last) (month|week|year)': 'sum_period',

  // List queries
  'show (me )?(all )?{category} (expenses|purchases)': 'list_category',
  'what did I buy at {merchant}': 'list_merchant',

  // Status queries
  'am I (over|under) budget': 'budget_status',
  'how much (is|do I have) left': 'budget_remaining',

  // Comparison queries
  'compare (this|last) month to {period}': 'compare_periods',
};
```

**Size Impact:** 0MB (uses native SFSpeechRecognizer + local patterns)

---

### 2. Conversational Entry (Premium)

**Description:** Create expenses via natural language.

**Examples:**
- "Coffee at Starbucks, five fifty"
- "Uber to airport yesterday, twenty three dollars"
- "Lunch with Sarah, split forty dollars"

**Technical Approach:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Voice     │───▶│   Speech    │───▶│   Expense   │───▶│  Confirm    │
│   Input     │    │Recognition  │    │   Parser    │    │   & Save    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                            │
                              ┌─────────────┼─────────────┐
                              ▼             ▼             ▼
                         Amount         Merchant      Category
                         Extractor      Matcher       Predictor
```

**Parser Enhancement (extend existing NaturalLanguageService):**
```typescript
// Enhance src/services/NaturalLanguageService.ts

interface ConversationalExpense {
  amount: number;
  merchant?: string;
  category?: string;
  date: Date;
  description?: string;
  confidence: number;
  ambiguities?: string[];  // "Did you mean Starbucks on Main St?"
}

class NaturalLanguageService {
  // Existing methods...

  // Enhanced for voice
  parseVoiceInput(transcript: string): ConversationalExpense;
  resolveAmbiguity(expense: ConversationalExpense, choice: string): ConversationalExpense;

  // Voice-specific number parsing
  private parseSpokenNumber(text: string): number | null;
  // "five fifty" → 5.50
  // "twenty three dollars" → 23.00
  // "a hundred and fifty" → 150.00
}
```

**Size Impact:** ~5KB (pattern extensions to existing service)

---

### 3. Siri Shortcuts (Premium)

**Description:** iOS Shortcuts integration for quick actions.

**Shortcuts:**
| Shortcut | Action | Example Trigger |
|----------|--------|-----------------|
| Quick Add | Open expense form | "Hey Siri, log expense" |
| Voice Add | Conversational entry | "Hey Siri, add expense" |
| Daily Summary | Speak today's spending | "Hey Siri, spending today" |
| Budget Check | Speak budget status | "Hey Siri, budget status" |
| Scan Receipt | Open camera for OCR | "Hey Siri, scan receipt" |

**Implementation:**
```typescript
// src/services/SiriShortcutsService.ts
// Uses expo-shortcuts or native module

interface ShortcutDefinition {
  identifier: string;
  title: string;
  phrase: string;
  action: () => void | Promise<void>;
}

class SiriShortcutsService {
  registerShortcuts(): void;
  handleShortcut(identifier: string, params?: Record<string, unknown>): Promise<void>;
  donateInteraction(shortcutId: string): void;  // Improves Siri suggestions
}

// App.tsx integration
useEffect(() => {
  SiriShortcutsService.registerShortcuts();

  // Handle incoming shortcut
  Linking.addEventListener('url', ({ url }) => {
    if (url.startsWith('atlaslegder://shortcut/')) {
      const shortcutId = url.split('/').pop();
      SiriShortcutsService.handleShortcut(shortcutId);
    }
  });
}, []);
```

**Info.plist additions:**
```xml
<key>NSUserActivityTypes</key>
<array>
  <string>com.atlaslegder.quickadd</string>
  <string>com.atlaslegder.voiceadd</string>
  <string>com.atlaslegder.summary</string>
  <string>com.atlaslegder.budget</string>
  <string>com.atlaslegder.scan</string>
</array>
```

**Size Impact:** 0MB (iOS native API)

---

### 4. Smart Predictions (Premium)

**Description:** Context-aware expense suggestions.

**Prediction Triggers:**
| Context | Prediction |
|---------|------------|
| Location: Starbucks | "Coffee $5.50?" |
| Time: 12:30 PM | "Lunch?" |
| Day: Monday 8 AM | "Weekly commute?" |
| After gym check-in | "Gym membership?" |
| Near frequent merchant | Pre-fill merchant |

**Technical Approach:**
```
┌─────────────────────────────────────────────────────────────────┐
│                    CONTEXT COLLECTOR                             │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐       │
│  │ Location  │ │   Time    │ │  Calendar │ │  Health   │       │
│  │  Service  │ │  Service  │ │   Events  │ │   Kit     │       │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PATTERN MATCHER                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Analyze historical expenses with similar context        │   │
│  │  - Same location ± 100m                                  │   │
│  │  - Same time of day ± 1 hour                            │   │
│  │  - Same day of week                                      │   │
│  │  - Similar calendar event keywords                       │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SUGGESTION RANKER                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Rank by: frequency × recency × confidence              │   │
│  │  Filter: confidence > 0.7                               │   │
│  │  Limit: top 3 suggestions                               │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Implementation:**
```typescript
// src/services/PredictionService.ts

interface PredictionContext {
  location?: { lat: number; lng: number };
  time: Date;
  dayOfWeek: number;
  calendarEvents?: string[];
  recentActivity?: string[];  // HealthKit
}

interface ExpensePrediction {
  merchant: string;
  category: string;
  amount: number;
  confidence: number;
  reason: string;  // "You usually buy coffee here at this time"
}

class PredictionService {
  async getPredictions(context: PredictionContext): Promise<ExpensePrediction[]>;

  // Learn from user actions
  recordAcceptance(prediction: ExpensePrediction): void;
  recordRejection(prediction: ExpensePrediction): void;
  recordManualEntry(expense: Expense, context: PredictionContext): void;
}
```

**Database Schema Addition:**
```sql
CREATE TABLE prediction_patterns (
  id INTEGER PRIMARY KEY,
  merchant TEXT,
  category TEXT,
  avg_amount REAL,
  location_lat REAL,
  location_lng REAL,
  time_of_day INTEGER,  -- minutes from midnight
  day_of_week INTEGER,
  frequency INTEGER,
  last_occurrence TEXT,
  confidence REAL
);

CREATE INDEX idx_patterns_location ON prediction_patterns(location_lat, location_lng);
CREATE INDEX idx_patterns_time ON prediction_patterns(time_of_day, day_of_week);
```

**Size Impact:** ~10KB code + ~100KB data (grows with usage)

---

### 5. Document Wallet (Premium)

**Description:** Store and organize documents linked to expenses.

**Document Types:**
- Warranties
- Contracts
- Insurance policies
- Business cards
- Manuals/instructions
- Tax documents

**Features:**
- OCR text extraction for search
- Expiration date tracking
- Category organization
- Quick access from expense detail

**Implementation:**
```typescript
// src/services/DocumentService.ts

interface Document {
  id: number;
  expenseId?: number;
  type: 'warranty' | 'contract' | 'insurance' | 'card' | 'manual' | 'tax' | 'other';
  title: string;
  filePath: string;
  thumbnailPath: string;
  extractedText?: string;
  expirationDate?: string;
  metadata: Record<string, unknown>;
  createdAt: string;
}

class DocumentService {
  async saveDocument(file: string, metadata: Partial<Document>): Promise<Document>;
  async linkToExpense(documentId: number, expenseId: number): Promise<void>;
  async getDocumentsForExpense(expenseId: number): Promise<Document[]>;
  async searchDocuments(query: string): Promise<Document[]>;
  async getExpiringDocuments(daysAhead: number): Promise<Document[]>;
}
```

**UI Integration:**
- Document section in Expense Detail screen
- Standalone "Documents" tab in settings
- Expiration alerts as push notifications

**Size Impact:** ~15KB code

---

### 6. Gamification (Premium - Optional)

**Description:** Achievement and streak system to encourage consistent tracking.

**Features:**

**Streaks:**
| Streak | Description |
|--------|-------------|
| Daily Logger | Log expenses X days in a row |
| Budget Master | Stay under budget X weeks |
| Receipt Collector | Attach receipts X times |
| Voice Pro | Use voice entry X times |

**Achievements:**
| Badge | Criteria |
|-------|----------|
| First Steps | Log first expense |
| Centurion | Log 100 expenses |
| Budget Beginner | Create first budget |
| Category King | Use all categories |
| Voice Activated | First voice expense |
| Time Traveler | Log expense from past |
| Streak Legend | 30-day logging streak |

**Challenges (Weekly):**
- "Log 5 expenses this week"
- "Stay 10% under food budget"
- "Use voice entry 3 times"

**Implementation:**
```typescript
// src/services/GamificationService.ts

interface Achievement {
  id: string;
  title: string;
  description: string;
  icon: string;
  unlockedAt?: string;
  progress: number;  // 0-100
  target: number;
}

interface Streak {
  type: string;
  currentCount: number;
  longestCount: number;
  lastActivityDate: string;
}

interface Challenge {
  id: string;
  title: string;
  description: string;
  startDate: string;
  endDate: string;
  target: number;
  progress: number;
  reward?: string;
}

class GamificationService {
  // Achievements
  async checkAchievements(): Promise<Achievement[]>;
  async unlockAchievement(achievementId: string): Promise<void>;

  // Streaks
  async updateStreak(type: string): Promise<Streak>;
  async getStreaks(): Promise<Streak[]>;

  // Challenges
  async getActiveChallenges(): Promise<Challenge[]>;
  async updateChallengeProgress(challengeId: string, progress: number): Promise<void>;

  // Settings
  async setGamificationEnabled(enabled: boolean): Promise<void>;
}
```

**Settings Integration:**
```
Settings → Preferences → Gamification
  [Toggle] Enable achievements & streaks
  [Toggle] Show streak notifications
  [Toggle] Weekly challenges
```

**Size Impact:** ~20KB code + badge assets (~100KB)

---

### 7. AR Receipt Scan (Pro)

**Description:** Point camera at receipt, see expense preview overlaid in AR.

**User Flow:**
1. Open camera with AR mode
2. Point at receipt
3. See extracted data floating above receipt
4. Tap "Confirm" to save
5. Receipt auto-attached

**Technical Approach:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  ARKit      │───▶│   Vision    │───▶│   OCR       │───▶│  AR         │
│  Camera     │    │   Detect    │    │  Extract    │    │  Overlay    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                 │                  │                   │
  Real-time         Detect           Extract text         Show parsed
   camera          receipt           & structure          data in 3D
   feed            bounds                                  space
```

**Implementation:**
```typescript
// src/screens/ARReceiptScanScreen.tsx

import { ARView } from 'expo-ar';  // or ViroReact

function ARReceiptScanScreen() {
  const [detectedReceipt, setDetectedReceipt] = useState<ReceiptBounds | null>(null);
  const [extractedData, setExtractedData] = useState<ExtractedReceipt | null>(null);

  const onFrameUpdate = async (frame: ARFrame) => {
    // 1. Detect receipt in frame using Vision
    const receipt = await detectReceipt(frame);
    if (receipt) {
      setDetectedReceipt(receipt);

      // 2. Extract text via OCR
      const data = await extractReceiptData(frame, receipt.bounds);
      setExtractedData(data);
    }
  };

  return (
    <ARView onFrameUpdate={onFrameUpdate}>
      {extractedData && (
        <AROverlay
          position={detectedReceipt.centerPoint}
          data={extractedData}
          onConfirm={saveExpense}
        />
      )}
    </ARView>
  );
}
```

**Dependencies:**
- expo-camera (already installed)
- ARKit (iOS native, 0MB)
- ViroReact or expo-ar (if cross-platform needed)

**Size Impact:** 0-5MB depending on AR library choice

---

### 8. Advanced Voice - Whisper (Pro, On-Demand)

**Description:** Higher accuracy speech recognition for difficult environments.

**When to Use:**
- Noisy environments
- Non-native accents
- Multiple languages
- When native iOS fails

**Implementation:**
```typescript
// src/services/WhisperService.ts

import { initWhisper, transcribeRealtime } from 'whisper.rn';

class WhisperService {
  private isInitialized = false;
  private model: WhisperModel | null = null;

  async initialize(): Promise<void> {
    if (this.isInitialized) return;

    // Check if model is downloaded
    const modelPath = await this.getModelPath();
    if (!modelPath) {
      throw new Error('Whisper model not downloaded');
    }

    this.model = await initWhisper({ modelPath });
    this.isInitialized = true;
  }

  async downloadModel(
    modelSize: 'tiny' | 'base' = 'tiny',
    onProgress: (progress: number) => void
  ): Promise<void> {
    // Download from CDN to app documents
  }

  async transcribe(audioPath: string): Promise<string> {
    if (!this.model) throw new Error('Whisper not initialized');
    return this.model.transcribe(audioPath);
  }

  async transcribeRealtime(
    onPartialResult: (text: string) => void
  ): Promise<void> {
    // Real-time streaming transcription
  }
}
```

**Module Manager Integration:**
```typescript
// User enables Whisper in Settings
async function enableWhisper() {
  const size = await WhisperService.getModelSize('tiny');  // 39MB

  Alert.alert(
    'Download Voice Model',
    `This will download ${size}MB for enhanced voice recognition. Continue?`,
    [
      { text: 'Cancel', style: 'cancel' },
      {
        text: 'Download',
        onPress: async () => {
          await WhisperService.downloadModel('tiny', setProgress);
        }
      }
    ]
  );
}
```

**Size Impact:** 0MB base, +39MB when downloaded

---

### 9. Conversational AI - Cloud (Pro)

**Description:** Advanced AI queries via encrypted cloud service.

**Examples:**
- "Analyze my spending patterns and suggest improvements"
- "What subscriptions can I cancel to save money?"
- "Prepare my expenses for tax filing"
- "Compare my spending to similar households"

**Privacy Architecture:**
```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ON DEVICE                                        │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  1. User asks: "Analyze my spending patterns"                    │   │
│  │  2. Prepare anonymized summary:                                  │   │
│  │     - Category totals (not individual expenses)                  │   │
│  │     - Patterns (not specific merchants)                         │   │
│  │     - Trends (percentages, not absolute amounts if preferred)   │   │
│  │  3. Encrypt with user's key                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼ HTTPS + E2E Encrypted
┌─────────────────────────────────────────────────────────────────────────┐
│                         CLOUD SERVICE                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  - Cannot read user data (encrypted)                            │   │
│  │  - Processes encrypted blob                                      │   │
│  │  - Returns encrypted response                                    │   │
│  │  - Zero data retention                                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         ON DEVICE                                        │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  4. Decrypt response with user's key                            │   │
│  │  5. Display insights to user                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

**Implementation:**
```typescript
// src/services/CloudAIService.ts

class CloudAIService {
  private encryptionKey: CryptoKey;

  async askQuestion(question: string): Promise<AIResponse> {
    // 1. Prepare context (anonymized spending summary)
    const context = await this.prepareAnonymizedContext();

    // 2. Encrypt request
    const encryptedPayload = await this.encrypt({
      question,
      context,
      timestamp: Date.now(),
    });

    // 3. Send to API
    const response = await fetch(AI_ENDPOINT, {
      method: 'POST',
      body: encryptedPayload,
      headers: { 'X-Encryption-Version': '1' },
    });

    // 4. Decrypt response
    const decryptedResponse = await this.decrypt(await response.text());

    return decryptedResponse;
  }

  private async prepareAnonymizedContext(): Promise<SpendingSummary> {
    // Only send aggregated, anonymized data
    return {
      categoryTotals: await this.getCategoryTotals(),
      monthlyTrend: await this.getMonthlyTrend(),
      budgetStatus: await this.getBudgetStatus(),
      // NO: individual transactions, merchant names, exact amounts
    };
  }
}
```

**Size Impact:** ~5KB code (encryption handled by expo-crypto)

---

## Privacy & Security

### Data Classification

| Data Type | Storage | Sync | AI Processing |
|-----------|---------|------|---------------|
| Expenses | Local SQLite | E2E Encrypted | Anonymized only |
| Receipts | Local files | E2E Encrypted | On-device OCR |
| Voice recordings | Local files | Never synced | On-device only |
| Transcripts | Local SQLite | E2E Encrypted | On-device only |
| Documents | Local files | E2E Encrypted | On-device OCR |
| Patterns/ML | Local SQLite | Never synced | On-device only |
| Settings | Local + SecureStore | E2E Encrypted | Never |

### Encryption Standards

```typescript
// E2E Encryption for Sync (Pro tier)
const ENCRYPTION_CONFIG = {
  algorithm: 'AES-GCM',
  keyLength: 256,
  ivLength: 12,
  tagLength: 128,
  keyDerivation: 'PBKDF2',
  keyDerivationIterations: 100000,
};
```

### Privacy Controls (Settings)

```
Settings → Privacy
  ┌─────────────────────────────────────────────┐
  │ Data Processing                              │
  │ ○ All processing on-device (default)        │
  │ ○ Allow anonymized cloud AI (Pro)           │
  ├─────────────────────────────────────────────┤
  │ Location                                     │
  │ [Toggle] Use location for predictions       │
  │ [Toggle] Store location with expenses       │
  ├─────────────────────────────────────────────┤
  │ Voice                                        │
  │ [Toggle] Save voice recordings              │
  │ [Toggle] Auto-delete after transcription    │
  ├─────────────────────────────────────────────┤
  │ Analytics                                    │
  │ [Toggle] Anonymous usage analytics          │
  │ [Toggle] Crash reporting                    │
  └─────────────────────────────────────────────┘
```

---

## Size Budget

### Current State
| Component | Size |
|-----------|------|
| React Native runtime | ~15MB |
| Expo modules | ~20MB |
| App code (98K lines) | ~5MB |
| Assets | ~5MB |
| **Total** | **~45MB** |

### With All Premium Features
| Addition | Size |
|----------|------|
| Voice Query patterns | ~10KB |
| Prediction service | ~10KB |
| Gamification + badges | ~120KB |
| Document wallet | ~15KB |
| Siri Shortcuts | 0KB |
| **Premium Total** | **~50MB** |

### With Pro Features (On-Demand)
| Addition | Size | Notes |
|----------|------|-------|
| AR components | ~5MB | If using library |
| Cloud AI client | ~5KB | |
| Whisper model | +40MB | Downloaded separately |
| **Pro Total** | **~55MB + 40MB optional** |

### Size Budget Enforcement
```typescript
// scripts/checkBundleSize.ts
const SIZE_LIMITS = {
  total: 70 * 1024 * 1024,      // 70MB hard limit
  jsBundle: 10 * 1024 * 1024,   // 10MB JS
  assets: 10 * 1024 * 1024,     // 10MB assets
  nativeCode: 50 * 1024 * 1024, // 50MB native
};

// Run in CI
async function checkSize() {
  const sizes = await measureBundleSizes();
  for (const [component, limit] of Object.entries(SIZE_LIMITS)) {
    if (sizes[component] > limit) {
      throw new Error(`${component} exceeds limit: ${sizes[component]} > ${limit}`);
    }
  }
}
```

---

## Implementation Phases

### Phase 1: Voice Foundation (4-6 weeks)
**Goal:** Complete voice infrastructure

| Week | Tasks |
|------|-------|
| 1-2 | Native speech recognition integration (expo-speech-recognition) |
| 2-3 | Voice Query service with intent parsing |
| 3-4 | Conversational entry (extend NaturalLanguageService) |
| 4-5 | Voice-to-Text in expense form |
| 5-6 | Testing, edge cases, accessibility |

**Deliverables:**
- [ ] VoiceQueryService with 20+ query patterns
- [ ] Enhanced NaturalLanguageService for voice
- [ ] Voice input throughout app
- [ ] 95%+ recognition accuracy for common queries

### Phase 2: Intelligence Layer (4-6 weeks)
**Goal:** Smart predictions and context awareness

| Week | Tasks |
|------|-------|
| 1-2 | PredictionService with location/time context |
| 2-3 | Pattern learning from user behavior |
| 3-4 | Predictive UI (suggestions in form) |
| 4-5 | Siri Shortcuts integration |
| 5-6 | Testing and refinement |

**Deliverables:**
- [ ] PredictionService with context awareness
- [ ] Prediction accuracy >70% for frequent merchants
- [ ] 5 Siri Shortcuts registered
- [ ] Location-based suggestions

### Phase 3: Engagement & Storage (3-4 weeks)
**Goal:** Gamification and Document Wallet

| Week | Tasks |
|------|-------|
| 1-2 | GamificationService with achievements |
| 2-3 | Streak tracking and challenges |
| 3 | Document Wallet service |
| 3-4 | UI integration and polish |

**Deliverables:**
- [ ] 10+ achievements
- [ ] 5 streak types
- [ ] Weekly challenges system
- [ ] Document storage with OCR search

### Phase 4: Pro Features (4-6 weeks)
**Goal:** AR and advanced AI

| Week | Tasks |
|------|-------|
| 1-2 | AR Receipt Scan prototype |
| 2-3 | Whisper integration (on-demand download) |
| 3-4 | Cloud AI service with E2E encryption |
| 4-5 | Module download system |
| 5-6 | Testing and optimization |

**Deliverables:**
- [ ] AR receipt scanning
- [ ] Whisper as optional download
- [ ] Cloud AI with privacy guarantees
- [ ] Module manager for on-demand features

### Phase 5: Teams Foundation (Future)
**Goal:** Lay groundwork for team features

- Shared expense model
- Invitation system
- Permission framework
- Team analytics

---

## Dependencies

### New Dependencies Required

| Package | Purpose | Size | Phase |
|---------|---------|------|-------|
| expo-speech-recognition | Native speech | ~1MB | 1 |
| whisper.rn | Advanced speech | ~2MB + 40MB model | 4 |
| expo-ar (optional) | AR overlay | ~5MB | 4 |

### Native Capabilities Required

| Capability | Info.plist Key | Purpose |
|------------|----------------|---------|
| Speech Recognition | NSSpeechRecognitionUsageDescription | Voice queries |
| Microphone | NSMicrophoneUsageDescription | Voice input |
| Location | NSLocationWhenInUseUsageDescription | Predictions |
| Camera | NSCameraUsageDescription | Receipt scan |
| Siri | NSSiriUsageDescription | Shortcuts |

### Backend Services (Pro tier)

| Service | Purpose | Privacy |
|---------|---------|---------|
| AI API | Complex queries | E2E encrypted |
| Sync API | Multi-device | E2E encrypted |
| Push notifications | Alerts | Device token only |

---

## Appendix

### A. Query Intent Patterns (Full List)

```typescript
// See: src/services/patterns/queryPatterns.ts

export const QUERY_PATTERNS = {
  // Summation
  SUM_CATEGORY: /how much (did I|have I) (spend|spent) on (\w+)/i,
  SUM_MERCHANT: /what (did I|have I) (spend|spent) at (.+)/i,
  SUM_PERIOD: /total (spending|expenses) (this|last) (month|week|year)/i,
  SUM_TODAY: /(how much|what) (did I|have I) (spend|spent) today/i,

  // Listing
  LIST_CATEGORY: /show( me)?( all)? (\w+) (expenses|purchases)/i,
  LIST_MERCHANT: /what did I buy at (.+)/i,
  LIST_RECENT: /show( me)?( my)? (recent|last|latest) (expenses|\d+ expenses)/i,

  // Status
  BUDGET_STATUS: /am I (over|under) budget/i,
  BUDGET_REMAINING: /how much (is|do I have) left/i,
  CATEGORY_BUDGET: /(\w+) budget (status|remaining|left)/i,

  // Comparison
  COMPARE_PERIODS: /compare (this|last) (month|week) to (.+)/i,
  COMPARE_CATEGORIES: /compare (\w+) (to|vs|versus) (\w+)/i,

  // Insights
  TOP_CATEGORY: /what('s| is) my (biggest|top|highest) (expense|category|spending)/i,
  UNUSUAL: /(any )?(unusual|strange|weird) (expenses|spending|charges)/i,
  TREND: /how (is|has) my (\w+) spending (trending|changed)/i,
};
```

### B. Gamification Achievements (Full List)

```typescript
// See: src/services/gamification/achievements.ts

export const ACHIEVEMENTS = [
  // Onboarding
  { id: 'first_expense', title: 'First Steps', icon: 'footsteps' },
  { id: 'first_budget', title: 'Budget Beginner', icon: 'calculator' },
  { id: 'first_receipt', title: 'Receipt Keeper', icon: 'receipt' },

  // Volume
  { id: 'expenses_10', title: 'Getting Started', target: 10 },
  { id: 'expenses_100', title: 'Centurion', target: 100 },
  { id: 'expenses_500', title: 'Expense Expert', target: 500 },
  { id: 'expenses_1000', title: 'Tracking Master', target: 1000 },

  // Features
  { id: 'voice_first', title: 'Voice Activated', icon: 'microphone' },
  { id: 'voice_10', title: 'Voice Pro', target: 10 },
  { id: 'all_categories', title: 'Category King', icon: 'crown' },
  { id: 'multi_account', title: 'Account Manager', icon: 'wallet' },

  // Streaks
  { id: 'streak_7', title: 'Week Warrior', target: 7 },
  { id: 'streak_30', title: 'Monthly Master', target: 30 },
  { id: 'streak_100', title: 'Streak Legend', target: 100 },

  // Budget
  { id: 'under_budget_1', title: 'Budget Win', icon: 'trophy' },
  { id: 'under_budget_4', title: 'Budget Month', target: 4 },
  { id: 'under_budget_12', title: 'Budget Year', target: 12 },
];
```

### C. Document Types Schema

```typescript
// See: src/types/document.ts

export interface DocumentType {
  id: string;
  label: string;
  icon: string;
  hasExpiration: boolean;
  defaultRetention: number; // days, 0 = forever
}

export const DOCUMENT_TYPES: DocumentType[] = [
  { id: 'warranty', label: 'Warranty', icon: 'shield-check', hasExpiration: true, defaultRetention: 0 },
  { id: 'receipt', label: 'Receipt', icon: 'receipt', hasExpiration: false, defaultRetention: 2555 }, // 7 years
  { id: 'contract', label: 'Contract', icon: 'file-document', hasExpiration: true, defaultRetention: 0 },
  { id: 'insurance', label: 'Insurance', icon: 'umbrella', hasExpiration: true, defaultRetention: 0 },
  { id: 'tax', label: 'Tax Document', icon: 'file-percent', hasExpiration: false, defaultRetention: 2555 },
  { id: 'manual', label: 'Manual', icon: 'book-open', hasExpiration: false, defaultRetention: 0 },
  { id: 'card', label: 'Business Card', icon: 'card-account-details', hasExpiration: false, defaultRetention: 1095 }, // 3 years
  { id: 'other', label: 'Other', icon: 'file', hasExpiration: false, defaultRetention: 365 },
];
```

---

*Document generated: January 2026*
*Next review: After Phase 1 completion*

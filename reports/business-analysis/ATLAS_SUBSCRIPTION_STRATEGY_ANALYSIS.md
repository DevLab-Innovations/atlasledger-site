# Feature-to-Value-Chain Report: Atlas Subscription Tier Strategy

**Analysis Date:** January 19, 2026
**Analyst:** Business Analyst Agent
**Document:** Strategic Analysis for Modularization and Growth

---

## Executive Summary

Atlas is positioning itself as a local-first expense tracker expanding into comprehensive cash flow management through a 4-tier subscription model. This analysis evaluates the strategic soundness of this approach, identifies modularization priorities for future Enterprise expansion, and assesses competitive positioning against established players like Monarch Money, YNAB, and Expensify.

**Key Finding:** The tier structure is strategically sound with clear value escalation, but requires careful modularization to avoid technical debt and enable future Enterprise growth. The "Owner Pays" model for Team tier is a market differentiator but introduces infrastructure complexity.

---

## 1. Feature Overview

### Summary
Atlas proposes a 4-tier subscription model (Free, Premium at $6.99/mo, Pro at $14.99/mo, Team at $39.99/mo) with a future Enterprise tier. Each tier unlocks progressively more accounts, features, and collaboration capabilities.

### Jobs To Be Done (JTBD) Analysis

| Tier | Primary Job | Secondary Jobs |
|------|-------------|----------------|
| **Free** | "Help me track where my money goes so I can file taxes accurately" | Receipt organization, mileage deduction capture |
| **Premium** | "Help me understand my complete financial picture across personal and business" | Bill management, income tracking, tax preparation |
| **Pro** | "Help me plan and optimize my cash flow across multiple businesses" | Budgeting, forecasting, paycheck allocation |
| **Team** | "Help my family/business collaborate on shared finances with accountability" | Approval workflows, spending attribution, shared visibility |

### Kano Model Categorization

| Feature Category | Kano Type | Rationale |
|------------------|-----------|-----------|
| Expense tracking + OCR | **Basic** | Table stakes - must work flawlessly or users churn |
| Smart categories | **Basic** | Expected from modern expense apps |
| Multiple accounts | **Performance** | Direct correlation: more accounts = more perceived value |
| Cash flow dashboard | **Performance** | Deeper visibility = higher satisfaction |
| Pay period budgeting | **Delighter** | Unexpected value - most competitors lack this |
| Approval workflows | **Performance** | Expected at team tier, more features = better |
| Paycheck allocation wizard | **Delighter** | Novel approach to common pain point |

**Strategic Insight:** The tier structure correctly gates Delighters at higher tiers while ensuring Basics are available universally.

---

## 2. Context Mapping

### Business Model Canvas Impact

```
+-------------------+-------------------+-------------------+
| Key Partners      | Key Activities    | Value Propositions|
| - Apple (StoreKit)| - Feature dev     | FREE: Simple tax  |
| - Supabase (Team) | - OCR optimization|   tracking        |
|                   | - Sync infra      | PREMIUM: Complete |
|                   |                   |   financial view  |
|                   |                   | PRO: Multi-biz    |
|                   |                   |   optimization    |
|                   |                   | TEAM: Collaborative|
|                   |                   |   accountability  |
+-------------------+-------------------+-------------------+
| Key Resources     |                   | Customer Relations|
| - Local SQLite    |                   | - Self-service    |
| - ML models (OCR) |                   | - Priority support|
| - Supabase infra  |                   |   (Pro+)          |
+-------------------+-------------------+-------------------+
| Cost Structure                        | Revenue Streams   |
| - Development (fixed)                 | - Premium $6.99/mo|
| - Supabase (variable, Team only)      | - Pro $14.99/mo   |
| - Apple 30% cut                       | - Team $39.99/mo  |
| - Support (scales with Pro+)          | - Annual plans    |
+-------------------+-------------------+-------------------+
```

### Ecosystem Dependencies

| Component | Dependency | Risk Level | Mitigation |
|-----------|------------|------------|------------|
| Subscriptions | Apple StoreKit 2 | Low | Industry standard, well-documented |
| Team Sync | Supabase | Medium | Abstract sync layer, could switch providers |
| OCR | On-device ML | Low | No external dependency |
| Local Storage | SQLite | Very Low | Battle-tested, no vendor lock-in |

---

## 3. Value Chain Mapping

### Porter's Value Chain Analysis

#### Primary Activities

| Activity | Free Impact | Premium Impact | Pro Impact | Team Impact |
|----------|-------------|----------------|------------|-------------|
| **Inbound Logistics** | Receipt capture, manual entry | + Recurring detection, bank statement understanding | + Multiple data streams | + Multi-user data aggregation |
| **Operations** | Expense categorization, storage | + Income/bill processing, tax calculations | + Cash flow projections, budget calculations | + Approval routing, conflict resolution |
| **Outbound Logistics** | Excel export | + Tax-ready reports | + Custom reports | + Team dashboards, member exports |
| **Marketing & Sales** | Free tier acquisition | Upgrade prompts, trial conversion | Feature-driven upsell | Referral from accountants |
| **Service** | Self-service | Self-service | Priority support | Dedicated support (Enterprise) |

**Value Driver Ranking by Tier:**

1. **Free Tier:** Operations (categorization) > Inbound (OCR) > Outbound (export)
2. **Premium Tier:** Inbound (income/bill tracking) > Operations (tax features) > Outbound (tax reports)
3. **Pro Tier:** Operations (projections) > Outbound (dashboards) > Inbound (multiple accounts)
4. **Team Tier:** Service (collaboration) > Operations (approvals) > Outbound (team visibility)

#### Support Activities

| Activity | Feature Mapping | Tier | Strategic Value |
|----------|-----------------|------|-----------------|
| **Technology Development** | AI categorization, OCR | All | Core differentiator |
| **Technology Development** | Sync infrastructure | Team | Enables collaboration moat |
| **Technology Development** | Pay period algorithms | Pro | Delighter feature |
| **Infrastructure** | StoreKit integration | All | Revenue enabler |
| **Infrastructure** | Supabase backend | Team | Scale enabler |
| **HR Management** | Priority support | Pro+ | Retention driver |
| **Procurement** | N/A (no external APIs) | - | Cost advantage |

### Wardley Map Positioning

```
                    Genesis    Custom    Product    Commodity
                       |          |         |          |
User Need (Track $)    |          |         |    [===] |
                       |          |         |          |
Expense Entry          |          |         |   [===]  |
OCR                    |          |  [===]  |          |
Smart Categories       |    [===] |         |          |
                       |          |         |          |
Income Tracking        |          |   [===] |          |
Bill Management        |          |   [===] |          |
Cash Flow Dashboard    |    [===] |         |          |
                       |          |         |          |
Pay Period Budgeting   | [===]    |         |          |
Paycheck Allocation    | [===]    |         |          |
                       |          |         |          |
Approval Workflows     |          |  [===]  |          |
Team Sync              |          |  [===]  |          |
                       |          |         |          |
SQLite Storage         |          |         |          | [===]
StoreKit               |          |         |          | [===]
```

**Strategic Insight:** Atlas's Genesis-stage features (Pay Period Budgeting, Paycheck Allocation Wizard) are key differentiators that justify Pro tier pricing. These should be protected and enhanced.

---

## 4. Strategic Analysis

### VRIO Framework

| Feature | Valuable | Rare | Inimitable | Organized | Competitive Position |
|---------|----------|------|------------|-----------|---------------------|
| Local-first architecture | Yes | Moderate | Low | Yes | **Temporary Advantage** |
| Smart OCR + categorization | Yes | Low | Low | Yes | Parity |
| Pay period budgeting | Yes | Yes | Moderate | Yes | **Sustained Advantage** |
| Paycheck allocation wizard | Yes | Yes | High | Partial | **Sustained Advantage** |
| Owner Pays Team model | Yes | Yes | Low | Partial | **Temporary Advantage** |
| IRS category mapping | Yes | Moderate | Low | Yes | Parity |

**Key Insight:** Pay period budgeting and the paycheck allocation wizard are Atlas's strongest competitive moats. These should receive continued investment.

### SWOT Analysis

| **Strengths** | **Weaknesses** |
|---------------|----------------|
| Local-first = privacy + offline | No cloud sync until Team tier |
| Pay period budgeting innovation | Complex Team tier infrastructure |
| Clean tier progression | No bank integrations (yet) |
| IRS category mapping for freelancers | iOS only (Android future) |
| No third-party API costs | Limited enterprise features |
| Owner Pays model reduces friction | Single-user until Team tier |

| **Opportunities** | **Threats** |
|-------------------|-------------|
| FinanceKit iOS 17+ integration | Monarch/YNAB brand recognition |
| Enterprise expansion | Apple building native expense tracking |
| Accountant referral network | Expensify downmarket movement |
| Android expansion | Free alternatives (spreadsheets) |
| API/integrations marketplace | Subscription fatigue |
| White-label for bookkeepers | Privacy regulation changes |

### PESTEL Analysis

| Factor | Impact | Relevance to Tiers |
|--------|--------|-------------------|
| **Political** | IRS reporting requirements increasing | Strengthens Premium tax features |
| **Economic** | Inflation driving cost-consciousness | Supports pay period budgeting value |
| **Economic** | Gig economy growth | Expands Premium target market |
| **Social** | Privacy concerns rising | Validates local-first approach |
| **Technological** | FinanceKit adoption | Enables Premium+ differentiators |
| **Technological** | AI commoditization | Erodes OCR advantage over time |
| **Environmental** | N/A | - |
| **Legal** | Data protection regulations | Local-first = compliance advantage |

### BCG Matrix (Feature Portfolio)

```
                    High Market Growth
                           |
    Stars                  |    Question Marks
    - Pay period budgeting |    - Approval workflows
    - Cash flow dashboard  |    - Team sync
                           |    - Savings goals
    -----------------------+------------------------
                           |
    Cash Cows              |    Dogs
    - Expense tracking     |    - Voice notes (low adoption)
    - OCR receipt capture  |    - Natural language entry
    - Excel export         |
                           |
                    Low Market Growth
```

**Portfolio Recommendation:**
- **Invest:** Pay period budgeting, cash flow dashboard (Stars)
- **Selective:** Approval workflows, Team sync (Question Marks - potential Stars)
- **Maintain:** Core expense tracking (Cash Cows - fund Stars)
- **Evaluate:** Voice notes, NL entry (low adoption - consider deprecation or repositioning)

---

## 5. Prioritization

### Impact/Effort Matrix

```
                        HIGH IMPACT
                             |
    Quick Wins               |    Major Projects
    - Recurring detection    |    - Supabase Team sync
    - Split transactions     |    - Approval workflows
    - Merchant insights      |    - Multi-user permissions
                             |
    -------------------------+---------------------------
                             |
    Fill-ins                 |    Avoid/Defer
    - Color coding           |    - Multi-currency
    - Year-over-year         |    - Bank integrations
                             |    - Android app
                             |
                        LOW IMPACT
        LOW EFFORT <------------------> HIGH EFFORT
```

### Cost-Benefit Summary

| Feature | Dev Cost | Revenue Impact | Priority |
|---------|----------|----------------|----------|
| Income tracking | Medium | High (unlocks Premium) | **P0** |
| Bill tracking | Medium | High (Premium core) | **P0** |
| Cash flow dashboard | High | High (Pro differentiator) | **P1** |
| Pay period budgeting | High | High (Pro differentiator) | **P1** |
| Recurring detection | Low | Medium (Premium sweetener) | **P2** |
| Approval workflows | Very High | Medium (Team niche) | **P2** |
| Supabase sync | Very High | High (enables Team) | **P1** |

### RICE Scoring

| Feature | Reach | Impact | Confidence | Effort | Score |
|---------|-------|--------|------------|--------|-------|
| Income tracking | 8 | 8 | 90% | 3 | **19.2** |
| Bill tracking | 8 | 8 | 90% | 3 | **19.2** |
| Cash flow dashboard | 6 | 9 | 85% | 5 | **9.2** |
| Pay period budgeting | 4 | 9 | 80% | 6 | **4.8** |
| Team sync (Supabase) | 2 | 10 | 70% | 10 | **1.4** |
| Approval workflows | 2 | 7 | 60% | 8 | **1.1** |

**RICE Interpretation:** Income and bill tracking score highest - correctly prioritized as Premium tier unlocks. Team features score lower due to smaller reach but are necessary for tier completeness.

---

## 6. Strategic Impact

### Balanced Scorecard Mapping

| Perspective | Free | Premium | Pro | Team |
|-------------|------|---------|-----|------|
| **Financial** | Acquisition funnel | $7/mo ARPU | $15/mo ARPU | $40/mo + network effects |
| **Customer** | "It just works" | "I see the whole picture" | "I'm in control" | "We're aligned" |
| **Internal Process** | Minimal support | Moderate support | Priority support | Dedicated support |
| **Learning & Growth** | Usage analytics | Feature adoption | Power user patterns | Team dynamics data |

### Critical Value Drivers

1. **Premium Conversion Rate** (Critical)
   - 7-day trial to paid conversion is the business model's lifeblood
   - Target: 15%+ trial-to-paid conversion

2. **Pro Upgrade Rate** (Critical)
   - Premium users discovering multi-business needs
   - Target: 10% Premium-to-Pro within 6 months

3. **Team Network Effects** (High)
   - Each Team subscriber brings 4 collaborators who may convert their personal use
   - Target: 20% collaborator-to-subscriber conversion

4. **Retention by Tier** (Critical)
   - Free: N/A (no revenue)
   - Premium: 85% annual retention
   - Pro: 90% annual retention
   - Team: 95% annual retention (higher switching costs)

5. **Feature-to-Upgrade Correlation** (High)
   - Which Free features predict Premium upgrade?
   - Which Premium usage predicts Pro upgrade?

### Strategic Risks and Mitigations

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| **Team tier complexity delays** | High | High | Modular architecture; ship Premium/Pro first |
| **Competitor copies pay period feature** | Medium | Medium | Patent consideration; continuous innovation |
| **Free tier too generous** | Medium | Low | Monitor conversion funnel; adjust limits |
| **Supabase costs exceed Team revenue** | High | Medium | Row-level metering; generous free tier limits |
| **Owner Pays exploitation** | Low | Medium | 5-collaborator limit; anti-abuse monitoring |
| **Premium cannibalizes Pro** | Medium | Low | Clear feature differentiation; Pro-only projections |

---

## 7. Feature Modularity Assessment

### Recommended Module Architecture

```
+------------------------------------------------------------------+
|                        CORE MODULE (All Tiers)                    |
| - Expense CRUD               - SQLite storage                    |
| - Category management        - Receipt OCR                       |
| - Mileage tracking           - Excel export                      |
| - Smart categorization       - Settings                          |
+------------------------------------------------------------------+
                                |
        +-----------------------+-----------------------+
        |                       |                       |
+-------v-------+       +-------v-------+       +-------v-------+
|   PREMIUM     |       |     PRO       |       |     TEAM      |
|   MODULE      |       |    MODULE     |       |    MODULE     |
+---------------+       +---------------+       +---------------+
| - Income CRUD |       | - Dashboard   |       | - Supabase    |
| - Bill CRUD   |       | - Projections |       | - User mgmt   |
| - Recurring   |       | - Pay periods |       | - Permissions |
|   detection   |       | - Budgeting   |       | - Approvals   |
| - Split trans |       | - Savings     |       | - Activity    |
| - Tax reports |       | - Custom rpts |       |   feed        |
| - Merchant    |       | - Trend       |       | - Team dash   |
|   insights    |       |   indicators  |       |               |
+---------------+       +---------------+       +---------------+
        |                       |                       |
        |               +-------v-------+               |
        |               |  ENTERPRISE   |               |
        |               |    MODULE     |               |
        |               +---------------+               |
        |               | - SSO/SAML    |               |
        |               | - Audit logs  |               |
        |               | - Admin panel |               |
        |               | - Unlimited   |               |
        |               |   accounts    |               |
        |               +---------------+               |
        |                       |                       |
        +-----------------------+-----------------------+
                                |
                    +-----------v-----------+
                    |   INTEGRATION MODULE  |
                    |      (Future)         |
                    +-----------------------+
                    | - FinanceKit import   |
                    | - Bank connections    |
                    | - Accounting exports  |
                    +-----------------------+
```

### Coupling Assessment

| Module Pair | Current Coupling | Recommended | Action Required |
|-------------|------------------|-------------|-----------------|
| Core <-> Premium | Tight | Loose | Extract income/bill services |
| Core <-> Pro | Tight | Loose | Abstract dashboard data layer |
| Premium <-> Pro | Moderate | Loose | Pro extends Premium, doesn't replace |
| Pro <-> Team | Loose | Very Loose | Team is pure add-on |
| All <-> Enterprise | N/A | Plugin | Design for extension |

### Modularization Priorities for Enterprise

1. **Authentication Abstraction** (P0)
   - Current: Sign in with Apple only
   - Enterprise needs: SSO/SAML, OAuth providers
   - Action: Create AuthProvider interface now

2. **Sync Layer Abstraction** (P0)
   - Current: Direct Supabase calls
   - Enterprise needs: Custom backends, on-premise
   - Action: SyncProvider interface with Supabase implementation

3. **Permission System Extensibility** (P1)
   - Current: Fixed 4-role model (Owner/Admin/Editor/Viewer)
   - Enterprise needs: Custom roles, attribute-based access
   - Action: RBAC foundation with configurable permissions

4. **Audit Logging Infrastructure** (P1)
   - Current: None
   - Enterprise needs: Complete audit trail
   - Action: Event sourcing pattern for Team tier

5. **Multi-tenancy Architecture** (P2)
   - Current: Single-tenant (per-user)
   - Enterprise needs: Organization-level isolation
   - Action: Tenant context throughout stack

### Technical Debt Considerations

| Area | Current State | Debt Risk | Recommendation |
|------|---------------|-----------|----------------|
| Account context | Global provider | Low | Good pattern, extend |
| Feature flags | UserDefaults | Medium | Move to typed enum system |
| Subscription check | Per-render | Medium | Cache with observer pattern |
| Database queries | Service methods | Low | Good, add query builders |
| Sync architecture | Not built | High (if rushed) | Invest in proper CRDT |

---

## 8. Upgrade Path Optimization

### "Aha Moment" Analysis

| Transition | Current Trigger | Optimized Trigger | Implementation |
|------------|-----------------|-------------------|----------------|
| Free -> Premium | Hit account limit | See income/expense gap | Show "You earned $X this month" prompt after 30 days |
| Free -> Premium | Tap locked feature | Tax season approach | Q1 tax reminder with Premium CTA |
| Premium -> Pro | Hit 2-account limit | Cash flow anxiety | Dashboard teaser: "Your projected balance in 30 days..." |
| Premium -> Pro | Manual forecasting | Paycheck timing pain | "Assign this bill to a paycheck" prompt |
| Pro -> Team | Share data manually | Spouse/partner request | "Share this account" as core action |
| Pro -> Team | Export for accountant | Accountant asks for access | "Invite your accountant (read-only)" |

### Upgrade Friction Points

| Friction | Severity | Solution |
|----------|----------|----------|
| Price anchoring ($0 to $7) | High | Emphasize trial value, annual savings |
| Feature discovery | Medium | Contextual feature education |
| Data loss fear | Low | "All data preserved on downgrade" messaging |
| Commitment anxiety | Medium | Monthly option prominent, easy cancellation |

### Recommended Upgrade Flow Enhancements

1. **In-Context Upgrade Prompts**
   - Show upgrade when user attempts locked action
   - Display feature preview before paywall
   - "See how this works" demo mode

2. **Usage-Based Triggers**
   - After 10 expenses: "Track income too?"
   - After first tax export: "Get tax-ready reports?"
   - After 30 days: "Your spending vs last month" (Pro preview)

3. **Social Proof Integration**
   - "50,000 freelancers use Atlas Premium"
   - "Pro users save 2 hours/week on budgeting"

---

## 9. Competitive Positioning

### Market Comparison

| Feature | Atlas Pro | Monarch | YNAB | Expensify |
|---------|-----------|---------|------|-----------|
| **Price** | $14.99/mo | $14.99/mo | $14.99/mo | $5/mo+ |
| **Local-first** | Yes | No | No | No |
| **Offline support** | Full | Limited | Limited | Limited |
| **Multi-account** | 5 accounts | Unlimited | 1 budget | Unlimited |
| **Pay period budgeting** | Yes | No | No | No |
| **Team collaboration** | Add-on tier | Included | 6 users | Included |
| **Bank integration** | No (FinanceKit future) | Yes | Yes | Yes |
| **Receipt OCR** | Yes | No | No | Yes |
| **Mileage tracking** | Yes | No | No | Yes |

### Positioning Strategy

**Target Segments by Tier:**

| Tier | Primary Segment | Secondary Segment |
|------|-----------------|-------------------|
| Free | Privacy-conscious individuals | Expense tracking newcomers |
| Premium | Solo freelancers, gig workers | Side hustle operators |
| Pro | Multi-business owners | Financial planners |
| Team | Small businesses (1-20) | Active households |

**Competitive Differentiation:**

1. **vs Monarch/YNAB:** Local-first privacy, offline-first, pay period innovation
2. **vs Expensify:** Simpler pricing, solo/small focus, no enterprise complexity
3. **vs Spreadsheets:** OCR automation, mobile-first, tax-ready exports

### Pricing Position Analysis

```
Price/mo
$50 |                                    [Enterprise competitors]
    |
$40 |                              [Atlas Team $39.99]
    |
$30 |                   [Expensify Team ~$30]
    |
$20 |
    |
$15 | [Atlas Pro $14.99] [Monarch $14.99] [YNAB $14.99]
    |
$10 |
    |
$7  | [Atlas Premium $6.99]
    |
$5  |                              [Expensify Collect $5]
    |
$0  | [Atlas Free]
    +-------------------------------------------------->
       Solo        Freelance      SMB        Enterprise
```

**Pricing Assessment:** Atlas is competitively positioned. Premium at $6.99 undercuts the $14.99 tier of competitors while providing income/bill tracking. Pro matches competitor pricing with differentiated features.

---

## 10. Cannibalization Risk Assessment

### Tier Cannibalization Analysis

| Risk | Description | Likelihood | Impact | Mitigation |
|------|-------------|------------|--------|------------|
| Free too generous | Users never upgrade | Low | High | Current limits appropriate |
| Premium sufficient | No Pro upgrade need | Medium | Medium | Strong Pro differentiators |
| Pro undervalues Team | Solo users buy Team for accounts | Low | Low | 5-account Pro limit is sufficient for most |

### Feature Cannibalization Analysis

| Feature | Risk | Analysis |
|---------|------|----------|
| 2 accounts in Premium | Could be enough | Monitor: what % of Premium users have 2 accounts at capacity? |
| Cash flow dashboard in Pro | Premium users want it | Intentional - this is the upgrade driver |
| 5 collaborators in Team | Small teams covered | Appropriate - Enterprise upsell path |

**Recommendation:** Current structure has appropriate gates. Monitor upgrade funnel metrics closely.

---

## 11. Recommendation Summary

### Should this tier structure be implemented?

**Yes, with modifications.**

### Rationale

1. **Value progression is clear:** Each tier solves a distinct job with appropriate pricing
2. **Modular potential is high:** Feature groupings enable clean technical boundaries
3. **Competitive position is strong:** Differentiated on privacy, offline, and pay period budgeting
4. **Enterprise path exists:** Team tier sets foundation for upmarket expansion

### Critical Modifications

1. **Decouple Team infrastructure early:** Do not let sync complexity delay Premium/Pro
2. **Invest in pay period budgeting:** This is the strategic moat - patent and enhance
3. **Add integration module foundation:** FinanceKit and future bank connections need abstraction
4. **Implement proper feature flagging:** Current UserDefaults approach will not scale

### Suggested Implementation Sequence

**Phase 1 (Months 1-3): Premium Foundation**
- Income tracking module
- Bill tracking module
- Recurring detection
- Premium subscription flow

**Phase 2 (Months 4-6): Pro Expansion**
- Cash flow dashboard
- Pay period budgeting
- Projections engine
- Pro subscription flow

**Phase 3 (Months 7-12): Team Infrastructure**
- Supabase backend abstraction
- Authentication provider interface
- Multi-user permission system
- Approval workflows
- Team subscription flow

**Phase 4 (Year 2): Enterprise Preparation**
- SSO/SAML integration
- Audit logging
- Admin console
- Custom role system

### Key Success Metrics

| Metric | Target | Timeline |
|--------|--------|----------|
| Trial-to-Premium conversion | 15% | Month 6 |
| Premium-to-Pro upgrade | 10% | Month 12 |
| Pro-to-Team upgrade | 5% | Month 18 |
| Premium annual retention | 85% | Year 1 |
| Pro annual retention | 90% | Year 1 |
| Team annual retention | 95% | Year 1 |

---

## Appendix A: Competitive Research Sources

- [Monarch Money Pricing](https://www.monarch.com/pricing) - $14.99/month or $99.99/year
- [YNAB vs Monarch Comparison](https://robberger.com/ynab-vs-monarch-money/)
- [Expensify Pricing](https://www.expensify.com/pricing) - Collect $5/month, Control custom pricing
- [Best Expense Tracking Apps 2025](https://use.expensify.com/resource-center/guides/best-business-expense-tracking-app)

---

## Appendix B: Framework References

- **Porter's Value Chain:** Porter, M.E. (1985). Competitive Advantage
- **VRIO Framework:** Barney, J. (1991). Firm Resources and Sustained Competitive Advantage
- **Jobs To Be Done:** Christensen, C. (2016). Competing Against Luck
- **Kano Model:** Kano, N. (1984). Attractive Quality and Must-Be Quality
- **Wardley Mapping:** Wardley, S. (2016). Wardley Maps
- **RICE Scoring:** Intercom Product Team

---

*Report generated by Business Analyst Agent*
*File: /Users/neptune/repos/agenticSdlc/docs/reports/business-analysis/ATLAS_SUBSCRIPTION_STRATEGY_ANALYSIS.md*

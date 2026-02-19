# Effort Estimate: Taxi / Livery Dispatch Log System

---

## Tech Stack Assumption

The spec lists multiple platform options (React+API, Notion, Excel, Desktop). This estimate assumes **Web Application (React + API)** — the spec's own recommendation for mid-to-large fleets. Specifically:

- **Frontend:** React (Next.js or CRA), Material-UI or similar component library
- **Backend:** Node.js/Express or Python/FastAPI REST API
- **Database:** PostgreSQL
- **Real-time:** WebSockets or polling for dispatch board updates
- **Auth:** Role-based (Dispatcher, Driver, Fleet Manager, Admin)

If a different platform is chosen (e.g., Notion workspace), the estimate would be dramatically different — likely 80-90% smaller.

---

## Phase 0: Spec Quality Assessment

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Behavioral Clarity | 4/5 | Trip lifecycle is well-defined with state diagram. Screen descriptions are functional but lack wireframes or interaction specifics (e.g., drag-to-assign? dropdown? click-to-edit?). |
| Scope Boundaries | 4/5 | Clear Phase 2 exclusions (SMS, mobile app, online booking, maps integration, invoicing, shift mgmt, dynamic pricing). Open questions section is honest about unknowns. Integrations are explicitly marked "Optional." |
| Data Contracts | 5/5 | Exceptional. Five complete entity schemas with field types, relations, and notes. Fare formula is explicit. This is the strongest part of the spec. |
| Success Criteria | 3/5 | NFRs exist (2s response, 99.5% uptime, 10+ concurrent users, 7-year retention) but no explicit "definition of done" per feature. No acceptance test scenarios. |
| Technical Specificity | 3/5 | Platform options listed but not committed. No specific framework versions, hosting platform, or deployment target. Integrations are optional/undefined. |
| Ambiguity Level | 3/5 | 6 open questions listed. Jurisdiction-specific compliance is unresolved. No clarity on whether GPS tracking is in scope. Driver mobile view mentioned but not specified. "Simplified mobile view" for drivers is vague. |
| **Average** | **3.7/5** | **Spec Quality Multiplier: 0.8x (Good)** |

**Assessment:** This is a good spec — the data model is production-ready and the trip lifecycle is clear. The main gaps are interaction design specifics and unresolved platform/integration decisions. The 0.8x multiplier applies.

---

## Phase 1: Spec Decomposition

### Functional Area Inventory

| # | Area | Category | Description |
|---|------|----------|-------------|
| 1 | Database Schema & Migrations | Data | 5 tables (Trips, Customers, Drivers, Vehicles, Dispatch Log) with relations, auto-IDs, timestamps |
| 2 | Auth & RBAC | Infrastructure | 4 roles with distinct permission sets, login, session management |
| 3 | Trip CRUD & Lifecycle Engine | Core Logic | Create/read/update trips, enforce state machine (Pending → Dispatched → En Route → In Progress → Completed/Cancelled/No-Show), auto-log to Dispatch Log |
| 4 | Customer Management | UI/UX + Data | Customer CRUD, search by name/phone, ride history, "Book Again" flow, default address pre-fill |
| 5 | Driver Management | UI/UX + Data | Driver CRUD, status tracking, license/expiry tracking, assignment to vehicles |
| 6 | Vehicle Management | UI/UX + Data | Vehicle CRUD, status tracking, insurance/inspection expiry, assignment to drivers |
| 7 | Rate Card & Fare Calculation | Core Logic | Configurable rate card, fare formula engine, surcharge rules (after-hours, holiday, airport), flat rate zones, manual override |
| 8 | Live Dispatch Board | UI/UX | Real-time table of active/upcoming trips, color-coded status, quick-assign driver, inline status updates, filters (date/status/driver/type) |
| 9 | Trip History & Search | UI/UX | Full-text search across trips, filters, CSV export |
| 10 | Driver Dashboard | UI/UX | Per-driver view: today's trips, revenue, status. Weekly/monthly rollup |
| 11 | Fleet Overview | UI/UX | Vehicle grid with status, expiry highlights, links to driver/trip history |
| 12 | Reporting & Analytics | Core Logic + UI | Daily summary report (auto-generated), weekly/monthly reports, KPI dashboard (8 metrics), revenue breakdowns |
| 13 | Compliance & Alerts | Polish | License/insurance/inspection expiry alerts (30-day, 7-day), immutable dispatch log audit trail, CSV/PDF export for regulators |
| 14 | Payment Logging | Data + UI | Payment method recording, payment status tracking, integration with fare calculation |
| 15 | Infrastructure & Deployment | Infrastructure | Hosting, CI/CD, daily backups with PITR, 99.5% uptime target, environment config |

---

## Phase 2: Complexity Classification

| # | Area | Complexity Type | Legacy Hours | Rationale |
|---|------|----------------|:---:|-----------|
| 1 | Database Schema & Migrations | Boilerplate | 12 | 5 well-defined tables, relations are explicit. Standard ORM work. |
| 2 | Auth & RBAC | Library Integration | 16 | 4 roles, standard auth library (NextAuth, Passport, etc.). Well-trodden ground. |
| 3 | Trip CRUD & Lifecycle Engine | Algorithmic | 30 | State machine with 7 states, validation rules, auto-logging side effects. Core complexity. |
| 4 | Customer Management | Boilerplate | 14 | Standard CRUD + search + "Book Again" pre-fill. |
| 5 | Driver Management | Boilerplate | 12 | Standard CRUD + status tracking + vehicle assignment. |
| 6 | Vehicle Management | Boilerplate | 10 | Standard CRUD + status + expiry dates. |
| 7 | Rate Card & Fare Calculation | Algorithmic | 18 | Configurable rules engine: base + mileage + wait + surcharges + flat rates + minimum + override. Not trivial but well-specified. |
| 8 | Live Dispatch Board | Creative/UX + Integration Glue | 35 | The hardest UI. Real-time updates, color-coding, quick-assign, filters. Needs WebSocket/polling + responsive table. |
| 9 | Trip History & Search | Library Integration | 12 | Full-text search, filters, pagination, CSV export. Standard data table with search library. |
| 10 | Driver Dashboard | Boilerplate | 10 | Filtered view of trips + aggregations. Standard dashboard. |
| 11 | Fleet Overview | Boilerplate | 8 | Grid view with status badges + expiry highlights. |
| 12 | Reporting & Analytics | Algorithmic + Creative/UX | 30 | 5 report types, 8 KPIs, aggregation queries, chart visualizations. Reporting is always more work than it looks. |
| 13 | Compliance & Alerts | Library Integration | 12 | Expiry date checks, scheduled alerts, PDF/CSV export. Cron job + export library. |
| 14 | Payment Logging | Boilerplate | 6 | Recording payment method/status on trips. No actual payment processing (that's Phase 2 / optional integration). |
| 15 | Infrastructure & Deployment | Architectural | 15 | Hosting decisions, backup strategy, CI/CD, environment config, uptime monitoring. |
| | **TOTAL** | | **240** | |

---

## Complexity Mix Profile

| Complexity Type | Areas | Legacy Hours | % of Effort | Factor | Weighted |
|---|---|:---:|:---:|:---:|:---:|
| Boilerplate | Schema, Customer, Driver, Vehicle, Driver Dashboard, Fleet Overview, Payment Logging | 72 | 30% | 12x | 3.60 |
| Library Integration | Auth/RBAC, Trip Search, Compliance/Alerts | 40 | 17% | 8x | 1.36 |
| Algorithmic | Trip Lifecycle, Fare Calc, Reporting (partial) | 60 | 25% | 6x | 1.50 |
| Creative/UX | Dispatch Board (partial), Reporting charts (partial) | 30 | 12.5% | 4x | 0.50 |
| Integration Glue | Dispatch Board real-time, cross-entity relations | 23 | 9.5% | 4x | 0.38 |
| Architectural | Infrastructure & Deployment | 15 | 6% | 2.5x | 0.15 |
| **Blended** | | **240** | **100%** | | **7.49x** |

**Blended Factor: 7.5x**

This lands squarely in the "Full-stack feature, well-integrated" profile (expected 5-7x). Slightly higher than the range midpoint because the spec's data model is exceptionally well-defined, pushing more work into boilerplate territory.

---

## Phase 4: Effort Estimation

### Applying the Formula

```
AI-Single = Legacy ÷ Blended Factor × Spec Quality Multiplier × (1 + Friction)
         = 240 ÷ 7.5 × 0.8 × (1 + 0.40)
         = 32.0 × 0.8 × 1.40
         = 35.8 hrs
```

**Friction breakdown for this project:**
- Wrong approach: +0.15 (multi-domain, several areas to get wrong)
- SDK errors: +0.10 (auth library, real-time updates, export libraries)
- Misunderstood request: +0.05 (spec is good, lower than typical)
- Integration bugs: +0.10 (frontend ↔ backend ↔ DB, but well-defined contracts reduce this)
- **Total friction: +0.40**

### Parallelism Analysis

| Workstream | Areas | Sequential? | Parallel Factor |
|---|---|---|:---:|
| Data layer (schema + migrations) | 1 | Must go first | Sequential |
| Backend API (CRUD + lifecycle + fare calc) | 3, 7, 14 | After schema, independent of UI | 3 agents |
| Entity management UI (Customer, Driver, Vehicle) | 4, 5, 6 | Independent of each other after API exists | 3 agents |
| Dashboard views (Dispatch Board, Driver, Fleet) | 8, 10, 11 | Independent of each other after API exists | 3 agents |
| Auth & RBAC | 2 | Can parallel with entity UI | 1 agent |
| Reporting | 12 | Needs data + API first | Sequential after backend |
| Search & compliance | 9, 13 | After core entities exist | 2 agents |
| Infrastructure | 15 | Can start immediately, finalize last | 1 agent |

**Max effective parallelism:** 3-4 agents working simultaneously after schema is done.

**AI-Parallel estimate:**
- Phase A (sequential): Schema + core API skeleton = ~3 hrs
- Phase B (3-4 agents parallel): All entity UI, dashboards, auth = ~8 hrs wall clock
- Phase C (sequential): Reporting, integration testing, deployment = ~5 hrs
- Orchestration overhead: +3 hrs (defining contracts, reviewing, merging)
- **Total: ~19 hrs wall-clock**

---

## Summary

| Metric | Legacy (2023) | AI-Single | AI-Parallel |
|--------|:---:|:---:|:---:|
| **Total Hours** | 240 | 36 | 19 |
| **Calendar Days** (6hr/day) | 40 days | 6 days | 3.2 days |
| **Human Decision Hours** | — | 14 | 10 |
| **Agent Execution Hours** | — | 22 | 9 (wall-clock) |

**Complexity profile match:** "Medium multi-domain, integration-heavy" → expected 15-40 hrs AI-single. Our 36 hrs fits this range.

---

## Decomposition (Detailed)

### 1. Database Schema & Migrations
- **What:** 5 tables with relations, auto-IDs, timestamps, indexes
- **Complexity Type:** Boilerplate
- **Legacy:** 12 hrs — schema design, migration scripts, seed data
- **AI-Single:** 1.3 hrs — (12 ÷ 12 × 0.8 × 1.20). Schema is fully specified in the spec. AI can generate migrations directly.
- **AI-Parallel:** 1.3 hrs — must run first, no parallelism
- **Confidence:** High — data model is the best part of this spec

### 2. Auth & RBAC
- **What:** 4 roles (Dispatcher, Driver, Fleet Manager, Admin), login, session, route protection
- **Complexity Type:** Library Integration
- **Legacy:** 16 hrs — auth setup, role middleware, protected routes
- **AI-Single:** 2.1 hrs — (16 ÷ 8 × 0.8 × 1.30). Standard auth library. +0.10 SDK friction, +0.15 wrong approach, +0.05 misunderstood.
- **AI-Parallel:** 2.1 hrs — can run parallel with entity UI
- **Confidence:** High — standard pattern

### 3. Trip CRUD & Lifecycle Engine
- **What:** Full trip state machine (7 states), validation, auto-logging to dispatch log
- **Complexity Type:** Algorithmic
- **Legacy:** 30 hrs — state transitions, validation, side effects, edge cases
- **AI-Single:** 5.6 hrs — (30 ÷ 6 × 0.8 × 1.40). State machine is well-specified but needs careful validation. Full friction applied.
- **AI-Parallel:** 3.5 hrs — API and state machine can be split across agents
- **Confidence:** High — lifecycle diagram is clear

### 4. Customer Management
- **What:** CRUD, search by name/phone, ride history, "Book Again" pre-fill
- **Complexity Type:** Boilerplate
- **Legacy:** 14 hrs — standard CRUD with search
- **AI-Single:** 1.3 hrs — (14 ÷ 12 × 0.8 × 1.15). Minimal friction, well-specified fields.
- **AI-Parallel:** 0.6 hrs — fully parallel with other entity UIs
- **Confidence:** High

### 5. Driver Management
- **What:** CRUD, status tracking, license expiry, vehicle assignment
- **Complexity Type:** Boilerplate
- **Legacy:** 12 hrs
- **AI-Single:** 1.1 hrs — (12 ÷ 12 × 0.8 × 1.15)
- **AI-Parallel:** 0.6 hrs — fully parallel
- **Confidence:** High

### 6. Vehicle Management
- **What:** CRUD, status tracking, insurance/inspection expiry, driver assignment
- **Complexity Type:** Boilerplate
- **Legacy:** 10 hrs
- **AI-Single:** 0.9 hrs — (10 ÷ 12 × 0.8 × 1.15)
- **AI-Parallel:** 0.5 hrs — fully parallel
- **Confidence:** High

### 7. Rate Card & Fare Calculation
- **What:** Configurable rate card, fare formula, surcharges, flat rate zones, manual override
- **Complexity Type:** Algorithmic
- **Legacy:** 18 hrs — rule engine, surcharge stacking, time-based rules
- **AI-Single:** 3.4 hrs — (18 ÷ 6 × 0.8 × 1.40). Formula is explicit which helps, but configurability (admin can change rates, flat rate zones) adds complexity.
- **AI-Parallel:** 2.0 hrs — can parallel with trip lifecycle
- **Confidence:** High — formula is specified

### 8. Live Dispatch Board
- **What:** Real-time trip table, color-coded status, quick-assign, filters, 2-second update target
- **Complexity Type:** Creative/UX + Integration Glue
- **Legacy:** 35 hrs — real-time UI, responsive table, interactions, WebSocket setup
- **AI-Single:** 8.0 hrs — (35 ÷ 4 × 0.8 × 1.40 for creative portion; 35 ÷ 4 × 0.8 × 1.50 for glue). This is the hardest piece. Real-time updates + responsive design + quick-assign UX + filtering = high iteration. Blended to ~8 hrs.
- **AI-Parallel:** 5.0 hrs — partially parallelizable (table component vs. WebSocket layer vs. filter logic)
- **Confidence:** Medium — "real-time" and interaction design specifics are underspecified

### 9. Trip History & Search
- **What:** Full-text search, filters, pagination, CSV export
- **Complexity Type:** Library Integration
- **Legacy:** 12 hrs
- **AI-Single:** 1.6 hrs — (12 ÷ 8 × 0.8 × 1.30)
- **AI-Parallel:** 1.0 hrs — parallel with other views
- **Confidence:** High — standard data table pattern

### 10. Driver Dashboard
- **What:** Per-driver trip list, revenue summary, status, weekly/monthly rollup
- **Complexity Type:** Boilerplate
- **Legacy:** 10 hrs
- **AI-Single:** 0.9 hrs — (10 ÷ 12 × 0.8 × 1.15)
- **AI-Parallel:** 0.5 hrs — fully parallel
- **Confidence:** High

### 11. Fleet Overview
- **What:** Vehicle grid with status badges, expiry highlights, linked driver/trip history
- **Complexity Type:** Boilerplate
- **Legacy:** 8 hrs
- **AI-Single:** 0.7 hrs — (8 ÷ 12 × 0.8 × 1.15)
- **AI-Parallel:** 0.4 hrs — fully parallel
- **Confidence:** High

### 12. Reporting & Analytics
- **What:** 5 report types, 8 KPIs, aggregation queries, chart visualizations, daily auto-generation
- **Complexity Type:** Algorithmic + Creative/UX
- **Legacy:** 30 hrs — aggregation queries, chart components, report layouts, scheduled generation
- **AI-Single:** 5.6 hrs — (30 ÷ 5 × 0.8 × 1.40). Blended factor ~5x (mix of algorithmic aggregation + chart UX). Reporting always takes longer than expected.
- **AI-Parallel:** 3.0 hrs — individual reports can parallel, but share aggregation layer
- **Confidence:** Medium — report design specifics not detailed

### 13. Compliance & Alerts
- **What:** Expiry alerts (30-day, 7-day), immutable audit log, CSV/PDF export
- **Complexity Type:** Library Integration
- **Legacy:** 12 hrs — scheduled checks, notification system, export formatting
- **AI-Single:** 1.6 hrs — (12 ÷ 8 × 0.8 × 1.30)
- **AI-Parallel:** 1.0 hrs — parallel with reporting
- **Confidence:** High — standard cron + export pattern

### 14. Payment Logging
- **What:** Record payment method and status on trips, no actual payment processing
- **Complexity Type:** Boilerplate
- **Legacy:** 6 hrs
- **AI-Single:** 0.6 hrs — (6 ÷ 12 × 0.8 × 1.15). Trivial — just fields on the trip record.
- **AI-Parallel:** 0.3 hrs — part of trip API work
- **Confidence:** High

### 15. Infrastructure & Deployment
- **What:** Hosting, CI/CD, backups with PITR, monitoring, environment config
- **Complexity Type:** Architectural
- **Legacy:** 15 hrs — decisions + implementation + testing
- **AI-Single:** 6.7 hrs — (15 ÷ 2.5 × 0.8 × 1.40). Human must decide hosting platform, backup strategy, monitoring approach. AI implements.
- **AI-Parallel:** 3.5 hrs — can start early (Docker, CI), finalize after app is built
- **Confidence:** Medium — depends on hosting platform choice (not specified)

---

## Area Totals Verification

| # | Area | Legacy | AI-Single | AI-Parallel |
|---|------|:---:|:---:|:---:|
| 1 | Database Schema | 12 | 1.3 | 1.3 |
| 2 | Auth & RBAC | 16 | 2.1 | 2.1 |
| 3 | Trip Lifecycle | 30 | 5.6 | 3.5 |
| 4 | Customer Mgmt | 14 | 1.3 | 0.6 |
| 5 | Driver Mgmt | 12 | 1.1 | 0.6 |
| 6 | Vehicle Mgmt | 10 | 0.9 | 0.5 |
| 7 | Fare Calculation | 18 | 3.4 | 2.0 |
| 8 | Dispatch Board | 35 | 8.0 | 5.0 |
| 9 | Trip Search | 12 | 1.6 | 1.0 |
| 10 | Driver Dashboard | 10 | 0.9 | 0.5 |
| 11 | Fleet Overview | 8 | 0.7 | 0.4 |
| 12 | Reporting | 30 | 5.6 | 3.0 |
| 13 | Compliance | 12 | 1.6 | 1.0 |
| 14 | Payment Logging | 6 | 0.6 | 0.3 |
| 15 | Infrastructure | 15 | 6.7 | 3.5 |
| | **Subtotal** | **240** | **41.4** | **25.3** |
| | Orchestration overhead (parallel) | — | — | +3.0 |
| | **TOTAL** | **240** | **~42** | **~28** |

*Note: Bottom-up sum (42 hrs AI-Single) is slightly higher than the top-down formula (36 hrs). This is expected — the top-down blended factor is an approximation. Using the **bottom-up detailed number** as the more accurate estimate.*

### Adjusted Summary

| Metric | Legacy (2023) | AI-Single | AI-Parallel |
|--------|:---:|:---:|:---:|
| **Total Hours** | 240 | 42 | 28 |
| **Calendar Days** (6hr/day) | 40 days | 7 days | 4.7 days |
| **Human Decision Hours** | — | 16 | 12 |
| **Agent Execution Hours** | — | 26 | 16 (wall-clock) |

**Sanity check:** "Medium multi-domain, integration-heavy" profile → expected 15-40 hrs. Our 42 hrs is slightly above the range, which is reasonable given this is a full application (not a single feature) with 5 entity types, real-time UI, reporting, compliance, and role-based auth. The estimate is credible.

---

## Risk Map

| Risk | Impact on Estimate | Likelihood | Mitigation |
|------|:---:|:---:|---|
| Tech stack not committed — wrong choice adds rework | +8-12 hrs | Medium | Decide stack before starting. Estimate assumes React + Node/FastAPI + PostgreSQL. |
| "Real-time" dispatch board has hidden complexity (concurrent edits, optimistic updates, conflict resolution) | +4-8 hrs | Medium | Define WebSocket contract early. Use optimistic UI with server reconciliation. |
| Reporting underspecified — chart types, exact layouts, export formats not defined | +3-6 hrs | High | Get report mockups or at minimum approve wireframes before building. |
| Jurisdiction-specific compliance requirements emerge mid-build | +5-15 hrs | Medium | Resolve open question about TLC/PUC regulations before starting compliance module. |
| "Simplified mobile view" for drivers is vague — could balloon into responsive redesign | +5-10 hrs | Medium | Clarify: is this a separate mobile-optimized page, or just responsive dispatch board? Exclude native app (that's Phase 2). |
| GPS/AVL tracking desired in Phase 1 (open question) | +15-25 hrs | Low | Keep out of Phase 1. If required, treat as a separate estimate. |

---

## Recommended Agent Strategy

### Parallel Workstreams

```
Timeline (AI-Parallel):

Day 1 (hrs 0-6):
├── Agent 1: Database schema + migrations + seed data [1.3 hrs] → then Trip API + lifecycle engine [3.5 hrs]
├── Agent 2: Infrastructure scaffolding (Docker, CI, project skeleton) [2 hrs] → then Auth & RBAC [2.1 hrs]
└── Agent 3: Rate card config + fare calculation engine [2 hrs] → then Payment logging [0.3 hrs]

Day 2 (hrs 6-12):
├── Agent 1: Customer management (CRUD + search + Book Again) [0.6 hrs] → Driver management [0.6 hrs] → Vehicle management [0.5 hrs]
├── Agent 2: Live Dispatch Board [5 hrs — longest single piece]
└── Agent 3: Trip Search + History [1 hr] → Driver Dashboard [0.5 hrs] → Fleet Overview [0.4 hrs]

Day 3 (hrs 12-18):
├── Agent 1: Reporting & Analytics (queries + KPIs) [1.5 hrs]
├── Agent 2: Reporting & Analytics (charts + UI) [1.5 hrs]
├── Agent 3: Compliance & Alerts [1 hr]
└── Human: Review all modules, integration testing, deployment finalization [3 hrs]

Day 4 (hrs 18-22):
└── Integration testing, bug fixes, polish, deploy [4 hrs]
```

### Human Review Checkpoints

1. **After schema + core APIs** (end of Day 1): Verify data model, test trip lifecycle transitions
2. **After Dispatch Board MVP** (mid Day 2): This is the critical UX piece — review before building other views
3. **After all CRUD views** (end of Day 2): Quick pass to ensure consistency
4. **After reporting** (mid Day 3): Verify aggregation accuracy, approve chart layouts
5. **Final review** (Day 4): End-to-end walkthrough, compliance check

---

## Research Prompt

```markdown
# Estimation Research: Taxi / Livery Dispatch Log System

## Context
Building a full-stack web application for taxi/livery dispatch management. React frontend,
REST API backend, PostgreSQL database. Key features: real-time dispatch board, trip lifecycle
management, fare calculation, fleet/driver/customer management, reporting, compliance logging.

## Research Questions

### Libraries & SDKs
- [ ] What is the best React data table library in 2026 for real-time updating tables with
      inline editing and color-coded rows? (AG Grid, TanStack Table, MUI DataGrid?)
- [ ] What is the recommended WebSocket library for React + Node/FastAPI in 2026?
      (Socket.io still dominant, or has something replaced it?)
- [ ] What is the best charting library for business reporting dashboards in React?
      (Recharts, Nivo, Chart.js, Tremor?)
- [ ] What PDF generation library works best server-side for compliance report exports?
      (Puppeteer, PDFKit, React-PDF?)
- [ ] What is the current best practice for full-text search in PostgreSQL?
      (Built-in tsvector, pg_trgm, or external like Meilisearch?)

### Architecture
- [ ] What is the standard approach for real-time dispatch boards in 2026?
      WebSocket pub/sub vs. Server-Sent Events vs. polling?
- [ ] Are there open-source taxi/livery dispatch systems that could serve as reference
      architecture? (Even partial implementations)
- [ ] What is the recommended state management pattern for real-time collaborative
      tables in React? (Optimistic updates, conflict resolution)

### Integration Points
- [ ] What is the simplest role-based auth setup for a React + API stack in 2026?
      (NextAuth v5, Clerk, Auth.js, custom JWT?)
- [ ] How to handle scheduled jobs (expiry alerts, daily report generation) in a
      Node/FastAPI backend? (node-cron, APScheduler, separate worker?)

### Unknowns
- [ ] Does the "simplified mobile view" for drivers imply responsive web or a
      separate mobile-optimized route?
- [ ] What jurisdiction-specific compliance requirements exist for taxi/livery
      trip logging? (NYC TLC format, state PUC requirements)
- [ ] Is GPS/AVL tracking expected in Phase 1? This would significantly change
      the architecture and estimate.

### Benchmarks
- [ ] Find real-world examples of dispatch management systems built with React + API —
      what was the actual build time?
- [ ] What parallel agent strategies have worked for full-stack CRUD applications
      with 5+ entity types?

## Estimation-Critical Findings
[To be filled by research]
```

---

## Research Gaps

The following areas have **Medium** confidence and would benefit from the research prompt above:

1. **Real-time dispatch board implementation** — The specific library and WebSocket approach affects the Dispatch Board estimate by +/- 3 hours
2. **Reporting specifics** — Without mockups or chart type decisions, reporting could be 3-6 hours larger
3. **Compliance requirements** — Jurisdiction-specific formats could add a significant chunk if they require custom export formatting
4. **Driver mobile view** — If this means a fully responsive design vs. a simplified page, the difference is 2-5 hours

---

*Generated by Spec Estimator skill — calibrated for AI-assisted development (2026)*

# Product Requirements Document (PRD)

## Product
Vault Radar Prototype

## Document status
Draft - based on current implementation in `radar.html`

## Date
2026-04-15

## 1. Overview

Vault Radar is a single-file prototype for secret risk operations. It demonstrates how security teams can move from raw secret detection to prioritized remediation by combining composite risk scoring, ownership context, and action-oriented workflows.

The prototype represents a product direction where secret findings are triaged, explained, and acted on in one interface, with supporting admin workflows for resource criticality configuration and initial system onboarding.

## 2. Problem Statement

Secret scanning tools often stop at detection. Teams still struggle with:

1. Backlog overload and poor prioritization
2. Severity labels that lack operational context
3. Slow manual triage and ownership routing
4. Weak visibility into risk posture over time
5. Difficulty tuning risk scoring to match business-critical resources

## 3. Goals

1. Prioritize findings using composite risk context, not severity alone.
2. Help analysts take remediation actions without leaving the workflow.
3. Let admins tune resource importance and see immediate risk impact.
4. Provide segmented operational views for decision-making by source, team, and trend.
5. Bootstrap initial org context through a setup wizard.

## 4. Non-Goals (current prototype)

1. Real backend integrations or live data ingestion
2. Persistence beyond browser session/local state
3. Full policy automation builder UI and execution engine
4. Production auth, RBAC, audit logging, and compliance controls
5. Ticketing or rotation API execution (buttons are UX stubs)

## 5. Target Users

1. Security Analyst
Needs to triage findings quickly, understand blast radius, and initiate remediation.

2. Security/Platform Lead
Needs to rebalance risk signal quality and assign responsibility at scale.

3. Security Admin
Needs to configure resource importance and bootstrap classification rules.

## 6. Jobs To Be Done

1. When I have many findings, I want a risk-ranked queue so I can fix the highest-impact leaks first.
2. When I open a finding, I want score explanation and remediation guidance so I can decide and act quickly.
3. When risk ranking feels off, I want to adjust resource importance so the queue reflects business reality.
4. When onboarding the system, I want to define patterns and crown jewels so baseline prioritization is usable.
5. When reporting posture, I want views by resource type, team, and trends so I can guide operational focus.

## 7. Product Scope and Functional Requirements

### 7.1 Findings View (`findings`)

Requirements:

1. Display top metrics (total findings, critical/live, auto-resolved, avg time to fix).
2. Show findings table with:
- Severity
- Secret type
- Resource and resource type
- Environment
- Composite risk score
- Validity
- Owner
- Age
- SLA breach indicator
3. Support filtering by:
- Quick chips: all, critical, live, unowned, SLA breach
- Search text
- Resource type
- Environment
- Sort key
4. Support bulk selection and bulk action stubs.
5. Show right-side detail panel with:
- Summary
- Score factor breakdown
- Blast radius
- Finding metadata
- SLA state
- Remediation recommendation
- Action button stubs

### 7.2 Views Section (left nav)

#### A) By Resource Type (`resourceType`)

Requirements:

1. Show summary cards for type-level posture.
2. Show per-type table:
- Resource count
- Finding count
- Live count
- Critical count
- Average score
- Top risk signal
3. Show compact visual comparison of average score across types.

#### B) By Team (`team`)

Requirements:

1. Aggregate findings into ownership groups (team, individual owners, unowned).
2. Show team-level KPIs:
- Teams with findings
- Largest backlog
- Lowest SLA compliance
- Unowned backlog
3. Show team table:
- Findings
- Live
- Critical
- Avg score
- SLA compliance
- Oldest age signal

#### C) Risk Trends (`riskTrends`)

Requirements:

1. Show trend summary cards:
- Open high-risk findings
- Avg risk score
- MTTR
- SLA compliance
2. Show weekly backlog trend visualization.
3. Show week-level trend table with discovered/resolved/open and operational metrics.

### 7.3 Resource Importance Settings (`resourceConfig`)

Requirements:

1. Show tier counts (critical/high/medium/low).
2. Show score distribution chart with before/after preview when overrides exist.
3. Show resource table with inferred tier, current tier override, findings count, avg score.
4. Recalculate finding scores live when tier changes.
5. Allow per-row reset and global reset.
6. Allow save operation that commits overrides as new baseline.
7. Show resource detail panel with associated findings and score deltas.

### 7.4 Onboarding Wizard

Requirements:

1. Step 1: Connect sources (simulated integrations + discovery counters).
2. Step 2: Teach rules (prod/dev naming patterns, crown jewels, policy toggles).
3. Step 3: Review generated classifications and apply.
4. Applying wizard updates inferred tiers and recalculates scoring baselines.

## 8. Data Model (Prototype)

1. `findings[]`
- Finding metadata, score, score factors, ownership, SLA state, remediation text.

2. `resources[]`
- Resource metadata, inferred/current tier, findings count, avg score.

3. `originalScores{}`
- Baseline score snapshot used for delta/preview and recalculation.

4. `tierMultipliers{}`
- Mapping used to scale finding scores from resource tier changes.

## 9. Interaction Model

1. `switchView(...)` controls nav active state, title, visible view container, and right panel behavior.
2. Findings table updates from filter and sort controls via `filterTable()`.
3. Resource config updates cascade through `recalcFindingScores()` and rerender dependent UI sections.
4. Analytics views (`resourceType`, `team`, `riskTrends`) are rendered as aggregated read models from existing prototype data.

## 10. UX and Visual Constraints

1. Flat surfaces and thin border system
2. Tokenized color and typography system
3. Score bars as primary risk encoding
4. Dark mode via media query
5. Right panel co-locates context and actions in operational workflows

## 11. Success Metrics (for eventual product)

1. Median time from finding detection to assignment
2. Median time to remediation by severity and environment
3. SLA compliance rate for high-risk findings
4. Percentage of unowned findings over time
5. Percentage of findings auto-triaged/auto-resolved
6. Reduction in open high-risk backlog week over week

## 12. Risks and Gaps

1. No real persistence or backend means behavior is demo-only.
2. Trend view uses synthetic series rather than historical telemetry.
3. Policy and integration nav sections are placeholders.
4. Saved views are represented in UI controls but not persisted as first-class objects.

## 13. Future Requirements

1. Policy Builder
- Condition/action rule editor with execution preview and trigger history.

2. Integrations Management
- Persistent connector states, sync metadata, and setup diagnostics.

3. Saved Views
- Persisted, shareable filter presets with ownership and defaults.

4. Team Drill-Down
- Click-through from team aggregates to scoped findings queue.

5. Historical Analytics
- Real event-driven trend computation and SLA forecasting.

## 14. Technical Context

1. Single-file implementation (`radar.html`) with HTML/CSS/JavaScript.
2. No build step and no external runtime dependencies.
3. Runs locally via static server (`python3 -m http.server 8765`).

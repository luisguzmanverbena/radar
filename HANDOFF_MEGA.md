# Vault Radar - Mega Handoff

This file consolidates context from:
- CONTEXT.md
- HANDOFF_2.md
- handoff.md

## How to use this file
- Read the Consolidated Brief first for fastest onboarding.
- Use the Source Snapshots section for full original context from each file.

## Consolidated Brief

### Project Overview
Vault Radar is a secret-risk operations prototype focused on helping teams remediate the most dangerous leaked secrets first.
It shifts from a passive findings list to an action-oriented triage workspace.
It also helps a SecOps admin understand the secret sprawl of their estate - where secrets exist, how widely they are distributed, which systems accumulate risk, and where ownership or monitoring coverage is weak.

### Problem Statement
Secret scanners can detect leaks, but teams still struggle with remediation at scale due to backlog volume and poor prioritization.
The core challenge is attention routing: which findings must be handled now.
There is a second operational problem as well: secret sprawl. Even before a specific incident is escalated, SecOps teams need to understand how secrets are spread across repos, cloud resources, collaboration tools, environments, and teams, because sprawl increases blind spots around exposure, rotation hygiene, and accountability.

### Product Approach
Composite risk scoring combines multiple signals, including severity, resource importance, exposure, validity, age, ownership, and environment context.
Critical Queue and analytics views convert that score into action lanes and operational insight.

### Secret Sprawl Management
Radar is not only a remediation workspace. It is also intended to be a visibility layer for secret sprawl management.

That means helping a SecOps admin answer questions such as:
- Where are secrets concentrated across the estate?
- Which teams, environments, or systems carry disproportionate secret risk?
- Which resources are monitored versus unmonitored?
- Where are there too many active secrets, unclear ownership, or weak revocation hygiene?

In product terms, secret sprawl visibility is the inventory and control-plane side of the experience, while remediation is the response side.

### Primary Users
- SecOps admins responsible for visibility, control coverage, and internal reporting
- Security engineers and AppSec or CloudSec operators
- Platform and SRE teams owning remediation
- Security program leads monitoring backlog and SLA health

### Current Implemented Scope in radar.html
- Findings view with filters, search, sorting, bulk actions, and finding detail panel
- Critical Queue with breached and near-breach lanes
- Views by resource type, by team, and risk trends to expose concentration and ownership patterns
- Resource Importance settings with tier overrides and live score recalculation to model which systems matter most
- Reports view for internal compliance and reporting workflows
- Three-step onboarding wizard for source setup and classification rules

### Important Runtime Notes
- Single-file prototype in radar.html (HTML, CSS, JS)
- No framework, no backend, no real integration calls
- Local run: python3 -m http.server 8765, then open /radar.html

### Core Data Structures
- findings[]
- resources[]
- originalScores{}
- tierMultipliers{}

### Key Functions to Start With
- switchView
- filterTable
- renderDetail
- renderCriticalQueueView
- renderResourceTypeView, renderTeamView, renderRiskTrendsView
- renderResourceConfig, recalcFindingScores, renderResourceTable
- showOnboarding, renderObStep, computeClassifications

### Known Constraints
- Trend data is synthetic
- SLA queue timing is age-based approximation
- Policies and integrations are partially placeholder surfaces
- Saved views are not persisted presets

### Recommended Next Steps
1. Add explicit detectedAt and slaDeadline fields and refactor queue timing.
2. Implement full SLA breaches and Unowned dedicated views.
3. Build policy builder and map onboarding toggles to editable policy objects.
4. Build integrations management with connection state and sync metadata.
5. Add persistence for saved views and filter presets.
6. Add dedicated secret sprawl reporting for concentration hotspots, ownership gaps, environment spread, and monitoring coverage drift.

## Source Snapshots

### CONTEXT.md

# Vault Radar - Agent Context (Source of Truth: `radar.html`)

## Purpose
This file is a handoff context for another agent. Scope is only what is implemented in `radar.html`.

## What This Project Is About
Vault Radar is about reducing secret-exposure risk in engineering systems by helping teams focus on the **most dangerous findings first**.

In practical terms, this prototype models a security operations workspace where users can:
- Identify which leaked secrets are truly urgent (not just numerous)
- Prioritize by business impact and remediation urgency (severity + environment + validity + ownership + age + resource criticality)
- Route and act on high-risk items quickly (especially critical findings and near-SLA breaches)
- Tune risk scoring behavior through resource importance and onboarding rules

Primary audience:
- Security engineers and AppSec/CloudSec operators
- Platform/SRE teams responsible for remediation ownership
- Security program leads who need backlog and trend visibility

Core product intent:
- Move from a passive “list of findings” experience to an action-oriented triage and remediation cockpit.

## Product Summary
Vault Radar is a single-file HTML/CSS/JS prototype for secret risk operations. It demonstrates how teams triage, prioritize, and remediate leaked secrets using composite context (severity, environment, validity, ownership, age, and resource importance).

## Runtime
- File: `radar.html`
- No framework, no build pipeline, no backend
- Run locally:
  - `cd /Users/luisguzman/Documents/2026/radar`
  - `python3 -m http.server 8765`
  - open `http://localhost:8765/radar.html`

## Implemented Navigation and Views

### Findings section
1. **All findings** (`findings`)
- Metrics cards
- Findings table (8 mock findings)
- Filter chips (All, Critical, Live only, Unowned, SLA breach)
- Search and select filters (resource/env/sort)
- Bulk action bar
- Right detail panel with score breakdown, blast radius, remediation actions

2. **Critical queue** (`criticalQueue`)
- Action-first queue for live high-risk findings
- KPI strip: critical live, breached now, breach < 4h, unowned in queue
- Two urgency lanes:
  - Breached now
  - Breach in next 4 hours
- Card-level quick actions (auto-rotate, escalate, create ticket)
- Countdown uses a prototype approximation from finding age vs SLA target

### Views section
3. **By resource type** (`resourceType`)
- Type-level KPI cards
- Summary table by source type (GitHub/Slack/AWS/Wiki)
- Compact bar comparison of average score by type

4. **By team** (`team`)
- Ownership aggregation (team names, individual owners, unowned)
- Team KPI cards
- Team backlog and SLA table

5. **Risk trends** (`riskTrends`)
- Trend KPI cards
- Weekly backlog bars
- Trend details table (new, resolved, open, avg risk, MTTR, SLA)
- Trend series is synthetic demo data generated in code

### Settings section
6. **Resource importance** (`resourceConfig`)
- Tier stats cards
- Distribution chart with before/after preview
- Resource table with inferred/current tier and sorting
- Live score recalculation when tier changes
- Per-row reset, global reset, save baseline changes
- Right panel resource detail with associated finding deltas

## Onboarding Wizard
Modal onboarding flow is implemented and re-runnable:
1. Connect sources (simulated integrations + discovery counts)
2. Teach what matters (prod/dev patterns, crown jewels, quick policies)
3. Review/apply generated classifications

Applying onboarding updates inferred tiers and recalculates baseline finding scores.

## Data Model (Client-side)
- `findings[]`: finding-level mock data (8 entries)
- `resources[]`: resource inventory mock data (16 entries)
- `originalScores{}`: baseline score snapshots for previews and recalculation
- `tierMultipliers{ critical, high, medium, low }`

## Key Functions
- View control: `switchView(view)`
- Findings filtering: `filterTable()`
- Finding detail: `renderDetail()`
- Critical queue render: `renderCriticalQueueView()`
- Analytics views: `renderResourceTypeView()`, `renderTeamView()`, `renderRiskTrendsView()`
- Resource config: `renderResourceConfig()`, `recalcFindingScores()`, `renderResourceTable()`
- Onboarding: `showOnboarding()`, `renderObStep*()`, `computeClassifications()`

## Current UX Behavior Notes
- Right panel is visible for `findings` and `resourceConfig`, hidden for analytics and critical queue views.
- `findingsActions` topbar buttons remain visible for findings/critical queue/analytics views.
- `configActions` topbar buttons are visible only in resource importance view.

## Known Prototype Constraints
1. No persistent storage besides local browser state for onboarding trigger.
2. No real integrations or API actions.
3. Trend and SLA countdown logic is simulated.
4. Policies/integrations pages in sidebar are placeholders.
5. Saved views behavior is not fully implemented as persistent objects.

## Good Next Steps (if another agent continues)
1. Replace synthetic trend data with derived history model.
2. Introduce explicit `detectedAt` and `slaDeadline` fields to remove age-based SLA approximation.
3. Implement real focused views for remaining Findings nav items (`SLA breaches`, `Unowned`).
4. Build policy builder and integrations management screens from placeholders.
5. Add persisted saved views and filter presets.

### HANDOFF_2.md

# HANDOFF_2 - Vault Radar Agent Continuation Context

## 1) What This Project Is About
Vault Radar is a product prototype for security teams that need to triage and remediate leaked secrets quickly.

The central idea is to shift from a passive findings list to an action-oriented risk operations workspace:
- Prioritize by real impact and urgency, not just raw severity labels.
- Route ownership fast (especially for unowned and near-SLA findings).
- Let admins tune scoring behavior through resource importance tiers and onboarding rules.

Primary users:
- AppSec / CloudSec engineers
- Platform and SRE teams handling remediation
- Security program leads tracking backlog and SLA health

## 2) Source of Truth and Scope
Source of truth for implemented behavior: radar.html.

This handoff is intentionally scoped to what is currently present in radar.html.

## 3) Current Product Surface (Implemented)

### Navigation and views
- Findings: All findings table with filters, search, bulk actions, and right-side finding detail panel.
- Critical queue: Action-first queue for live high-risk findings, split into breached and near-breach lanes.
- By resource type: Aggregated risk and volume by source category.
- By team: Ownership and SLA aggregation by team.
- Risk trends: Synthetic weekly trend series and KPI summary.
- Resource importance: Settings view to classify resources by Critical/High/Medium/Low and recalculate scores.

### Right panel behavior
- Visible in Findings and Resource importance views.
- Hidden in Critical queue and analytics views.

### Onboarding wizard
Three-step flow:
1. Connect simulated integrations
2. Teach classification rules (patterns, crown jewels, policy toggles)
3. Review and apply inferred tier classification

Applying onboarding updates inferred tiers and resets score baselines.

## 4) Core Product Logic

### Composite scoring model
Finding risk score is influenced by:
- secret type
- resource importance
- exposure
- validity
- age

Resource importance tier multipliers:
- critical: 1.0
- high: 0.75
- medium: 0.5
- low: 0.25

Tier overrides in Resource importance trigger live score recomputation for linked findings.

### Critical queue logic
Queue is built from live findings where severity is critical or score >= 80.
SLA urgency is approximated from finding age:
- critical findings target 4 hours
- high-risk findings target 12 hours

Items are bucketed into:
- breached now
- breach in next 4 hours

## 5) Data Model (In-Memory)
- findings[]: mock finding records, factors, ownership, SLA flags, scores.
- resources[]: mock resource inventory with inferred/current tier and metadata.
- originalScores{}: baseline scores for before/after projections.
- tierMultipliers{}: tier-to-weight mapping.

No backend persistence exists for these models.

## 6) Important Functions to Know First
- switchView(view): global view routing and topbar state.
- filterTable(): finding-level filtering/search/sorting.
- renderDetail(): right-panel finding detail rendering.
- renderCriticalQueueView(): lane generation and urgent action cards.
- renderResourceTypeView(), renderTeamView(), renderRiskTrendsView(): analytics views.
- renderResourceConfig(), recalcFindingScores(), renderResourceTable(): resource importance settings and recomputation.
- showOnboarding(), renderObStep(), computeClassifications(): onboarding workflow and rule application.

## 7) Known Constraints / Prototype Gaps
- Static, client-side prototype only (no API/backend).
- Trend data is synthetic.
- SLA countdown is age-based approximation, not deadline-based.
- Some sidebar entries are placeholders (for example policies/integrations screens as full pages).
- Saved views are not persisted as reusable user-defined presets.

## 8) Recommended Next Work (Priority)
1. Add explicit detectedAt and slaDeadline fields to findings and refactor queue timing logic off deadlines.
2. Implement full screens for SLA breaches and Unowned nav entries under Findings.
3. Build policy builder UI and connect onboarding policy toggles to editable policy objects.
4. Build integrations management view with connection state and sync metadata.
5. Persist saved filter/view presets and restore them on reload.

## 9) Run and Validate
- From project directory, run: python3 -m http.server 8765
- Open: http://localhost:8765/radar.html
- Manual smoke check:
  - switch across all sidebar views
  - open/close finding detail panel
  - change resource tiers and confirm score + distribution updates
  - run onboarding and confirm classifications apply

## 10) Files in This Workspace
- radar.html: main executable prototype and only runtime code artifact.
- CONTEXT.md: concise implementation-focused context.
- handoff.md: broader handoff/spec narrative from earlier phase.
- radar-prd.md: PRD-style product framing.
- radar-alternative.html, radar-forecast-lab.html: alternative concept prototypes.

### handoff.md

# Vault Radar — Agent Handoff Document

## What this project is

**Vault Radar** is a secret scanning dashboard prototype (inspired by IBM/HashiCorp's product). It's a single-file HTML/JS/CSS prototype (`radar.html`) that demonstrates how a security team would manage leaked secrets (API keys, tokens, passwords) found across their org's repos, Slack channels, cloud accounts, and wikis.

The full product vision is documented in `CONTEXT.md` (if present in your workspace). The prototype is in `radar.html`.

---

## The core problem

Most secret scanning tools are good at **finding** secrets but terrible at helping teams **manage** and **remediate** them at scale. Enterprise backlogs hit tens of thousands of findings. The volume problem is really an **attention problem** — teams need to spend time on the right findings, not just see fewer of them.

### The proposed solution: Composite risk scoring

Instead of flat severity labels (critical/high/medium/low), every finding gets a **composite risk score (0–100)** computed from six multiplied factors:

| Factor | What it measures | Example |
|---|---|---|
| **Secret type severity** | Inherent danger of the secret type | AWS key > Slack webhook |
| **Resource importance** | How critical the resource is to the org | `payments-api` (Critical) vs `sandbox-experiments` (Low) |
| **Environment signal** | Prod vs staging vs dev vs archived | Prod = highest weight |
| **Exposure surface** | How broadly accessible the leak is | Public GitHub repo > private Slack DM |
| **Validity** | Is the secret still live? | Live/confirmed = maximum urgency |
| **Age** | How long it's been unactioned | Older = more urgent |

**Key insight:** These are multiplicative. A medium-severity secret in a prod repo with public exposure and confirmed validity scores **higher** than a critical-type secret in an archived dev sandbox.

**Resource importance** and **exposure surface** are orthogonal axes — both matter independently and should never be conflated in the UI.

---

## What's been built

### 1. Findings view (main dashboard)

The primary view showing all discovered secrets. Includes:

- **Metrics bar**: Total findings (2,847), critical/live count, auto-resolved count, avg time to fix
- **Findings table**: 8 mock findings with severity badges, secret type, resource name, resource type icon (GitHub/Slack/AWS/Wiki), environment tag (Prod/Staging/Dev), composite risk score with visual bar, validity indicator, owner avatar, age, SLA breach indicator
- **Filtering**: Filter chips (All, Critical, Live only, Unowned, SLA breach), dropdowns for resource type/environment/sort order, text search
- **Bulk actions**: Checkbox selection with assign/suppress/escalate/close bar
- **Detail panel**: Click a finding row → right panel shows risk score breakdown by factor, blast radius chips, finding details, SLA status, remediation guidance, action buttons (auto-rotate, assign, create Jira ticket, suppress)

### 2. Resource importance configuration

A settings view (sidebar → Settings → Resource importance) where admins manage how the system classifies their resources. Includes:

- **Tier stats cards**: Count of Critical/High/Medium/Low resources
- **Score distribution chart**: Shows how findings distribute across risk score ranges (76–100, 51–75, 26–50, 0–25)
- **Resource table**: 16 mock resources showing name, type, env, inferred tier with reasoning, current tier dropdown, finding count, avg score
- **Live score recalculation**: Changing a tier immediately recalculates all associated finding risk scores using tier multipliers (`critical: 1.0, high: 0.75, medium: 0.5, low: 0.25`)
- **Before/after distribution preview**: When overrides are pending, the distribution chart shows delta indicators (+1/-1) and ghost bars showing the original distribution
- **Resource detail panel**: Click a resource row → right panel shows resource metadata and all associated findings with their live/recalculated scores
- **Per-resource reset**: "reset" link on overridden rows to undo individual changes
- **Sortable columns**: Click column headers (Resource, Type, Tier, Findings, Avg score) to sort
- **Toast notifications**: Visual confirmation on save, reset, and tier changes

### 3. Onboarding wizard

A 3-step modal flow that appears on first visit to bootstrap the system's understanding of the org:

**Step 1 — Connect your sources**
- 6 integration cards (GitHub, Slack, AWS, Jira, Okta, Wiki) with click-to-connect
- Discovery stats show counts (47 repos, 12 channels, etc.) as sources connect

**Step 2 — Teach us what matters**
- **Naming pattern inputs**: Pre-filled tag pills for prod patterns (`*-prod`, `*-production`, `*-api`) and dev patterns (`*-dev`, `*-sandbox`, `*-test`). Add/remove patterns.
- **Crown jewel selector**: Star specific services to force them Critical regardless of patterns
- **Quick policy toggles**: "Public repos = High", "Staging = Medium", "Archived repos auto-close"

**Step 3 — Review & apply**
- Tier distribution bars showing the computed result
- Resource breakdown with each resource, assigned tier, and the matching reason
- "Rules generated" summary showing all patterns → tier mappings
- "Apply & start scanning" generates `computeClassifications()` and applies results

The wizard can be re-triggered via "Run setup wizard" button in the resource config topbar.

---

## What remains to be built

These are the remaining features from the original CONTEXT.md roadmap, in priority order:

### 1. Policy builder (auto-triage rules)

> [!IMPORTANT]
> This is arguably the most impactful remaining feature — it's how admins manage volume without manual triage.

**Problem**: Admins need to define rules like "auto-close findings in archived repos" or "auto-escalate unactioned critical findings after 72h" without writing code.

**Proposed solution**: A condition → action rule editor:
- **Conditions**: resource type, tier, severity, age, validity, owner (is/is not assigned), SLA status
- **Actions**: auto-assign to team/person, auto-suppress, auto-escalate, auto-close, create Jira ticket
- **UI**: Card-based rule list showing active rules, match counts, last triggered. Add/edit modal with condition builder (AND/OR groups) and action picker.
- **Integration with onboarding**: The quick policies from onboarding step 2 (public repos = high, archived = auto-close) should appear as editable rules here.

### 2. Risk trends chart

**Problem**: No way to see whether the security posture is improving or getting worse over time.

**Proposed solution**: A time-series chart (probably in the main findings view or a dedicated "Risk trends" view from the sidebar):
- Finding volume over time, stacked by severity tier
- Mean time-to-remediation trend line
- New vs resolved findings per week
- SLA compliance percentage over time

### 3. Team view

**Problem**: Findings are shown flat — no way to see which team has the biggest backlog or worst SLA compliance.

**Proposed solution**: A view (sidebar → Views → By team) showing:
- Team cards with aggregate risk score, finding count, SLA compliance %
- Click a team → filtered findings table for that team
- Team-level trend sparklines

### 4. Integrations page

**Problem**: The onboarding wizard simulates connections but there's no persistent integrations management page.

**Proposed solution**: A full settings page (sidebar → Automation → Integrations) showing:
- Connected/available integration cards with status, last sync time, resource counts
- Connection settings per integration (OAuth flow simulation, scope selection)
- Progressive enrichment indicators (e.g., "Connect Okta to auto-assign team ownership")

### 5. Saved views

**Problem**: CONTEXT.md describes saved views ("Critical resources with live secrets", "Unowned findings > 14 days") but they're only sidebar labels with no actual filter persistence.

**Proposed solution**: Make the sidebar items under "Findings" actually apply filter presets. Add ability to save custom filter combinations as new sidebar items.

---

## Architecture notes

### Single-file prototype
Everything is in one `radar.html` file (~1900 lines). No build tools, no framework, no external dependencies. Serves via `python3 -m http.server 8765`.

### Data model
All data is client-side mock data defined as JS arrays/objects:
- `findings[]` — 8 mock findings with all score factors
- `resources[]` — 16 mock resources with inferred tiers
- `originalScores{}` — baseline scores for before/after comparison during tier changes
- `tierMultipliers{}` — `{ critical: 1.0, high: 0.75, medium: 0.5, low: 0.25 }`

### View switching
`switchView('findings' | 'resourceConfig')` toggles visibility of view containers and updates the topbar/sidebar active state. The right panel is reused for both finding details and resource details.

### Score recalculation
`recalcFindingScores()` runs whenever a tier changes. It uses the ratio of old multiplier to new multiplier against the original baseline score: `newScore = originalScore * (newMultiplier / oldMultiplier)`, clamped to 1–100.

### Onboarding classification engine
`computeClassifications()` applies user-defined patterns via regex matching (`*` → `.*`), crown jewel overrides, and policy toggles to generate tier assignments for all resources.

---

## Design system constraints

> [!WARNING]
> Maintaining the design system is critical. The prototype follows a Mercury.com-inspired aesthetic. Deviating from these principles will make the prototype feel inconsistent.

- **Flat surfaces** — no drop shadows, no gradients, no glassmorphism
- **0.5px borders** — thin, subtle borders (`var(--color-border-tertiary)`)
- **Typography**: Inter font family, muted colors, sentence case everywhere
- **Score bars** as primary visual encoding for risk — not just color badges
- **Color palette**:
  - Critical/danger: `#E24B4A` / `#A32D2D`
  - High/warning: `#EF9F27`
  - Medium/info: `#185FA5`
  - Low/success: `#639922`
  - Brand accent: `#1D9E75` (green, used for primary buttons, toggles, active states)
- **Remediation always co-located with context** — no context switching to a separate page
- **Resource importance + exposure surface always surfaced together** — never conflate them

### CSS custom properties in use
The file defines a full design token system at `:root` — colors, spacing, typography, border radii. All components reference these variables. Dark mode is handled via `@media (prefers-color-scheme: dark)`.

---

## How to run

```bash
cd /Users/luisguzman/Documents/2026/radar
python3 -m http.server 8765
# Open http://localhost:8765/radar.html
```

The onboarding wizard will auto-show on first visit (uses `localStorage` key `vr_onboarded` to track). Clear localStorage or use "Run setup wizard" button to re-trigger.

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

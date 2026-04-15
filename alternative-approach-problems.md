# Problems This Alternative Approach Addresses

This document captures the core problems targeted by the alternative Vault Radar prototype in `radar-alternative.html`.

## Why this exists

The original prototype is strong for inspection and detail analysis, but this alternative approach emphasizes execution flow and action sequencing.

## Problems addressed

1. Attention overload
Teams drown in long finding tables and lose sight of what needs action now.

2. Severity-only ranking fails
Raw severity misses context like exposure, ownership, validity, and remediation friction.

3. Manual triage does not scale
Repeated manual decisions across analysts cause slow response and inconsistent outcomes.

4. Weak ownership routing
Unowned findings sit idle and breach SLA even when risk is obvious.

5. Hard to convert insight into action
Many tools explain risk but do not make next actions explicit, sequenced, and capacity-aware.

## How the alternative UI responds

- Organizes findings into time-based execution lanes: Do Now, Next 24h, This Week, Monitor.
- Uses an actionability score (risk + urgency + ownership gap + blast radius) for ordering.
- Shows shift-level budget pressure to support realistic planning.
- Exposes a remediation ladder (Verify, Contain, Rotate, Close) for faster operator follow-through.

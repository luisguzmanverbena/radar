# Problems Addressed by Forecast Lab Approach

This document describes the core problems addressed by the third prototype in [radar-forecast-lab.html](radar-forecast-lab.html).

## Core problems

1. Attention overload from high finding volume
Security teams can review findings, but they struggle to decide where limited remediation capacity should go first.

2. Severity labels are insufficient for operational planning
Severity communicates risk, but it does not indicate queue pressure, ownership friction, or likely SLA impact under current team capacity.

3. Manual triage does not scale for enterprise backlogs
When analysts triage item by item without policy leverage, response times degrade and outcomes become inconsistent.

4. Ownership gaps create hidden delay
Unowned findings amplify queue aging and increase chance of SLA breaches, even for findings that are otherwise straightforward to remediate.

5. Teams cannot predict if posture is improving before acting
Most workflows show current state only. Teams need a short-horizon forecast to test policy decisions before spending effort.

## What this approach does differently

- Starts with a policy simulator instead of a findings table.
- Projects 7-day queue outcomes from policy settings and team capacity.
- Surfaces intervention queue based on expected operational drag, not just static severity.
- Connects policy choices to outcomes: SLA trend, MTTR trend, auto-resolved volume, and analyst time recovered.

## Jobs-to-be-done this supports

- Decide which automations to enable before triaging manually.
- Estimate whether current staffing can absorb risk backlog this week.
- Prioritize interventions that reduce operational drag the fastest.
- Improve ownership routing and escalation timing to protect SLA.

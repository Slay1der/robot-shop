# Postmortem: <incident title>

**Date:** <YYYY-MM-DD> · **Author:** Abdulquddus Yusuf · **Status:** Draft / Final
**Severity:** <critical / warning> · **Duration:** <start> to <end> (<minutes> min)

## Summary
<2-3 sentences: what broke, who it affected, how it was fixed.>

## Impact
- Users affected: <e.g. storefront unavailable / checkout failing>
- SLO impact: <minutes of downtime>, using <X%> of the monthly error budget (7h 12m)

## Timeline (UTC+1)
| Time | Event |
|---|---|
| hh:mm | Fault starts (<cause>) |
| hh:mm | Alert `<name>` fires in Slack #robot-shop-alerts |
| hh:mm | Engineer acknowledges, opens runbook |
| hh:mm | Root cause found via <dashboard / Loki query / kubectl> |
| hh:mm | Mitigation applied (<rollback / restart / scale>) |
| hh:mm | Probes green, Slack RESOLVED |

## Root cause
<What actually failed and why. Be blameless: describe systems, not people.>

## Detection
- How we found out: <alert / user report>
- Time to detect: <min> · Time to recover: <min>
- What went well: <...>
- What made it harder: <...>

## Action items
| Action | Owner | Due |
|---|---|---|
| <prevent it happening again> | | |
| <detect it faster> | | |
| <improve the runbook> | | |

## Evidence
<Screenshots: Grafana panel during the incident, Slack alert and resolve, Loki errors, kubectl output.>

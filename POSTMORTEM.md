# Postmortem: Robot Shop storefront outage (failure drill)

**Date:** 2026-10-28 · **Author:** Abdulquddus Yusuf · **Status:** Final
**Severity:** critical · **Duration:** about 10:51 to 11:18 UTC+1 (about 27 min)

## Summary
The web (storefront) Deployment was scaled to 0 replicas as a planned failure drill. With no pods behind the web Service, every request failed. The blackbox probe detected it within seconds, and both critical alerts fired. Scaling web back to 1 replica restored service.

## Impact
- Users affected: the whole storefront was unavailable (no pages served)
- SLO impact: about 27 min of downtime, which is about 6% of the monthly error budget (7h 12m at 99%)
- Backend services (catalogue, cart, payment) stayed healthy the whole time

## Timeline (UTC+1)
| Time | Event |
|---|---|
| ~10:51 | `kubectl scale deploy/web -n robot-shop --replicas=0` (fault injected) |
| ~10:51 | Blackbox probe for web starts failing (`probe_success = 0`) |
| ~10:52 | `RobotShopServiceDown` fires after its 1m `for:` window |
| ~10:52 | `RobotShopWebErrorBudgetFastBurn` fires (failure ratio over 14.4% on the 5m and 1h windows) |
| 11:17 | Engineer reviews alerts in the Prometheus UI (screenshot 03, 04) and checks the web Deployment |
| 11:18 | `kubectl scale deploy/web -n robot-shop --replicas=1` (mitigation) |
| ~11:19 | Probe succeeds and `RobotShopServiceDown` resolves (screenshot 05) |
| ~11:24 | FastBurn resolves once the 5m failure ratio drops below the threshold |

## Root cause
The web Deployment had 0 replicas, so the web Service had no endpoints and every connection was refused. This was a planned drill. In real life the same symptom comes from a bad release, a crash loop, or a scale-down by mistake.

## Detection
- How we found out: alerts (`RobotShopServiceDown` and FastBurn), and the Grafana availability panel dropped to 0%
- Time to detect: about 1 min · Time to recover after a human looked: about 1 min
- What went well: alerts fired fast and carried the `service` label and a `runbook_url`; the dashboard showed exactly which service was down
- What made it harder: Slack was not connected, so nobody was paged and the alert sat for about 25 min unnoticed. In a real incident, that wait is most of the downtime.
- FastBurn kept firing for about 4–5 min after recovery. That's expected, because it averages over a 5m window, and it stops the alert flapping.
- Side finding: a placeholder Slack URL made the operator reject the Alertmanager config (`PrometheusOperatorSyncFailed`). Fixed in Helm revision 3 with a valid-format URL.

## Action items
| Action | Owner | Due |
|---|---|---|
| Connect the Alertmanager Slack webhook so critical alerts page a human | Abdulquddus | next session |
| Run web with 2+ replicas and a PodDisruptionBudget so a single change can't take it to zero | Abdulquddus | next session |
| Add an alert for Deployments with 0 available replicas (`kube_deployment_status_replicas_available == 0`) | Abdulquddus | next session |
| Validate Alertmanager config in CI (`amtool check-config`) before `helm upgrade` | Abdulquddus | next session |

## Evidence
See [docs/evidence.md](docs/evidence.md) and `docs/screenshots/`.

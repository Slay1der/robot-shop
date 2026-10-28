# Robot Shop SLOs

## What we promise
| Service | SLI (how we measure) | SLO (target) |
|---|---|---|
| web (storefront) | % of blackbox probes to `http://web:8080/` that return 2xx | **99%** over 30 days |
| catalogue, cart, payment | % of probes to `/health` that return 2xx | 99% over 30 days |

## Error budget
- 99% means 1% of the time can fail: about **7h 12m per 30 days** (about 14 min per day).
- **Fast burn alert:** fires if web is failing 14.4x faster than the budget allows (more than 14.4% failed probes over both 5m and 1h). At that rate the monthly budget is gone in about 2 days. This page needs a human now.
- **Service down alert:** any probe failing for 1 minute.

## Golden signals on the dashboard ("Robot Shop - Golden Signals and SLO")
| Signal | Source |
|---|---|
| Latency | `probe_duration_seconds` from the blackbox exporter |
| Traffic | `sold_count_total` (payment) and `items_added` (cart) from `/metrics` |
| Errors | `1 - probe_success` |
| Saturation | built-in kube-prometheus-stack node and pod dashboards (CPU, memory) |

## Why blackbox probes
Only cart and payment expose Prometheus metrics. Probing from outside the app measures what a user sees, and it works for every service the same way.

## Limits
- Prometheus keeps 2 days of data, so the budget panel uses a 24h window as a demo. Production would keep 30 days.
- Probes run inside the cluster, so they don't catch problems at the internet edge.

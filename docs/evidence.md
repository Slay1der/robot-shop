# Capstone 2 evidence

## Robot Shop running on EKS (Helm revision 1)
```
$ helm install robot-shop K8s/helm -n robot-shop
STATUS: deployed
REVISION: 1

$ kubectl get pods -n robot-shop
cart-dc9c44f79-8sfnm         1/1   Running   0   100s
catalogue-86b87b8d98-6nq6z   1/1   Running   0   113s
dispatch-7f849ccb6b-8n6xn    1/1   Running   0   109s
load-64cbdb98f8-h2p58        1/1   Running   0   2m
mongodb-7d5c9874d8-cj5rx     1/1   Running   0   94s
mysql-847c56f488-xmbp2       1/1   Running   0   108s
payment-6797fcf8b6-w8zk2     1/1   Running   0   107s
rabbitmq-54f9bfcfb6-7k6nh    1/1   Running   0   83s
ratings-5f78cb68c4-5wws7     1/1   Running   0   2m21s
redis-0                      1/1   Running   0   108s
shipping-657cfbbfbd-7bslh    1/1   Running   0   3m18s
user-854856fd58-nq77r        1/1   Running   0   100s
web-6957797cd7-27xg5         1/1   Running   0   110s
```

## Monitoring stack healthy
```
$ kubectl get pods -n monitoring
alertmanager-kps-kube-prometheus-stack-alertmanager-0   2/2   Running
blackbox-prometheus-blackbox-exporter-66999fdc5-kgrvb   1/1   Running
fluent-bit-9g457                                        1/1   Running
fluent-bit-t6tgg                                        1/1   Running
kps-grafana-69d8845b78-knlg8                            3/3   Running
kps-kube-prometheus-stack-operator-6d9b6ff4-877sr       1/1   Running
kps-kube-state-metrics-747b7b996-vsm7t                  1/1   Running
kps-prometheus-node-exporter-qh2mx                      1/1   Running
kps-prometheus-node-exporter-vrrz8                      1/1   Running
loki-74d59c795d-g48wv                                   1/1   Running
prometheus-kps-kube-prometheus-stack-prometheus-0       2/2   Running

$ helm history kps -n monitoring
REVISION  STATUS      DESCRIPTION
1         superseded  Install complete
2         superseded  Upgrade complete
3         deployed    Upgrade complete   # valid Alertmanager config, operator sync fixed
```

## Screenshots
| File | Shows |
|---|---|
| `screenshots/01-robot-shop-storefront-on-eks.png` | Storefront served from EKS via port-forward |
| `screenshots/02-grafana-golden-signals-dashboard.png` | Golden signals / SLO dashboard auto-loaded, all services at 100% |
| `screenshots/03-drill-robot-shop-alerts-firing.png` | Drill: both Robot Shop alerts FIRING |
| `screenshots/04-drill-alert-details-runbook-link.png` | Alert details: expression, `for: 1m`, labels, runbook link |
| `screenshots/05-recovery-servicedown-resolved.png` | Recovery: ServiceDown resolved, FastBurn draining (value 0.6) |
| `screenshots/06-eks-controller-manager-false-positive.png` | Known EKS false positive: KubeControllerManagerDown (managed control plane can't be scraped) |

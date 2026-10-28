# Install monitoring (PowerShell, run from the robot-shop folder)

```powershell
# 0. (recommended) free capacity: Capstone 1 app off the cluster
helm uninstall voting

# 1. Chart repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

# 2. Prometheus + Grafana + Alertmanager (the Slack webhook stays out of Git)
kubectl create namespace monitoring
$env:SLACK_WEBHOOK = "https://hooks.slack.com/services/XXX/YYY/ZZZ"
helm install kps prometheus-community/kube-prometheus-stack -n monitoring -f monitoring/kps-values.yaml --set alertmanager.config.global.slack_api_url=$env:SLACK_WEBHOOK

# 3. Blackbox exporter (the release name must be "blackbox", because the probes point at it)
helm install blackbox prometheus-community/prometheus-blackbox-exporter -n monitoring

# 4. Loki + Fluent Bit
kubectl apply -f monitoring/loki.yaml
helm install fluent-bit fluent/fluent-bit -n monitoring -f monitoring/fluent-bit-values.yaml

# 5. Probes, scraping, SLO alerts, dashboard
kubectl apply -f monitoring/probes.yaml -f monitoring/servicemonitors.yaml -f monitoring/slo-rules.yaml -f monitoring/dashboard-configmap.yaml

# 6. Check
kubectl get pods -n monitoring

# 7. Grafana (user: admin)
$p = kubectl get secret -n monitoring kps-grafana -o jsonpath="{.data.admin-password}"
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($p))
kubectl port-forward -n monitoring svc/kps-grafana 3000:80

# Prometheus targets page (check the probes are UP): new terminal
kubectl port-forward -n monitoring svc/kps-kube-prometheus-stack-prometheus 9090:9090
# then open http://localhost:9090/targets
```

Slack webhook: api.slack.com/apps, Create App, Incoming Webhooks, turn it on, add it to #robot-shop-alerts, copy the URL.

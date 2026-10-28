# Robot Shop Runbook

Start with these commands:
```bash
kubectl get pods -n robot-shop
kubectl get events -n robot-shop --sort-by=.lastTimestamp | tail -20
```
Grafana: `kubectl port-forward -n monitoring svc/kps-grafana 3000:80`, then open http://localhost:3000.
Logs: Grafana, then Explore, then Loki: `{namespace="robot-shop", service="<name>"}`, or `{level="error"}`.

## RobotShopServiceDown
**Meaning:** the blackbox probe for one service has failed for at least 1 minute.

1. Find the pod: `kubectl get pods -n robot-shop -l service=<service>`
2. If it's CrashLoopBackOff or Error: `kubectl logs -n robot-shop deploy/<service> --previous`
3. If it's ImagePullBackOff: check the image tag with `kubectl describe pod`, then roll back with `helm rollback robot-shop -n robot-shop`
4. If it's Pending: `kubectl describe pod` and look for "Insufficient cpu/memory". Free capacity or scale nodes.
5. If the pod is Running but the probe fails: `kubectl exec -n robot-shop deploy/web -- curl -s http://<service>:8080/health`. Then check its dependency (mongodb, mysql, redis, rabbitmq).
6. Quick mitigation: `kubectl rollout restart -n robot-shop deploy/<service>`

**Escalate** if it isn't fixed in 15 minutes.

## RobotShopWebErrorBudgetFastBurn
**Meaning:** more than 14.4% of storefront checks are failing. Users can't shop, and the monthly error budget will be gone in about 2 days.

1. Did anything just change? `helm history robot-shop -n robot-shop`. If there was a recent upgrade, **roll back first and debug after**: `helm rollback robot-shop <last-good-revision> -n robot-shop`
2. Is web up? `kubectl get pods -n robot-shop -l service=web`
3. Are the services behind web up (catalogue, user, cart)? Check the dashboard availability panel. Web fails if they fail.
4. Find the first errors: Loki `{namespace="robot-shop", level="error"}`
5. After recovery: confirm availability is back to 100% on the dashboard, wait for the Slack RESOLVED message, then write a postmortem (POSTMORTEM.md).

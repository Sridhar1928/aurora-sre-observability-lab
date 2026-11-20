## Incident 2: Deployment Availability Outage
### Scenario
A controlled workload outage was introduced by scaling the demo-app Deployment from one replica to zero replicas.

The outage was triggered using:
```bash
kubectl scale deployment/demo-app --replicas=0 -n aurora-lab
```
The Deployment reached zero available replicas and no application Pods remained in the aurora-lab namespace.

## Monitoring Detection
kube-state-metrics exposed the Deployment availability state to Prometheus.

The following PromQL query monitored the available replicas of demo-app:
```promql
kube_deployment_status_replicas_available{namespace="aurora-lab",deployment="demo-app"}
```
During the outage, the metric changed from 1 available replica to 0.

Grafana evaluated the demo-app unavailable alert rule and transitioned the alert into the firing state.

The notification was delivered to Aurora through the configured Grafana webhook contact point.

## Aurora Incident
The Grafana webhook created an Aurora incident for demo-app unavailable.

The incident confirmed that the monitoring and alert-delivery pipeline successfully reached Aurora.

## Recovery
The application was restored by scaling the Deployment back to one replica:
```bash
kubectl scale deployment/demo-app --replicas=1 -n aurora-lab
```
Recovery was verified using:
```bash
kubectl get pods -n aurora-lab
```
The demo-app Pod returned to a healthy 1/1 Running state

## Result
This incident validated the complete monitoring and alert-delivery pipeline:
```text
Kubernetes outage
      ↓
kube-state-metrics
      ↓
Prometheus
      ↓
Grafana Alerting
      ↓
Aurora Webhook
      ↓
Aurora Incident
      ↓
Application Recovery
```
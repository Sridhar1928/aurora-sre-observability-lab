### Monitoring Configuration
## Overview
This directory documents the monitoring configuration used for the Aurora SRE Observability Lab.

The monitoring pipeline is:

Kubernetes → kube-state-metrics → Prometheus → Grafana → Aurora

## Prometheus
Prometheus collects metrics from the Kubernetes monitoring stack.

For the local lab, Prometheus was accessed using:
```bash
kubectl port-forward service/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
```
The Kubernetes Deployment availability metric used in this lab was:
```promql
kube_deployment_status_replicas_available{namespace="aurora-lab",deployment="demo-app"}
```
## Grafana Alerting
Grafana used the Prometheus metric to monitor the availability of the demo-app Deployment.

The alert rule was named:

`demo-app unavailable`

The alert condition triggered when the number of available replicas dropped below 1.

The alert notification was sent to Aurora through the configured Grafana webhook contact point.

## Result
The monitoring pipeline successfully validated:
```text
Kubernetes → kube-state-metrics → Prometheus → Grafana → Aurora
```
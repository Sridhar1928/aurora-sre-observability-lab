## Architecture
### Overview
This lab implements an end-to-end SRE observability and incident-management workflow connecting Kubernetes monitoring with Grafana alerting and Aurora incident investigation.

## Architecture Flow
```text
Kubernetes / Docker Desktop
          ↓
   kube-state-metrics
          ↓
      Prometheus
          ↓
       Grafana
          ↓
    Webhook Alert
          ↓
       Aurora
          ↓
AI-assisted Incident Investigation
```

## Component Responsibilities
- Kubernetes hosts the demo-app workload.
- kube-state-metrics exposes Kubernetes workload state as metrics.
- Prometheus collects and stores monitoring metrics.
- Grafana queries Prometheus and evaluates the demo-app unavailable alert.
- Aurora receives Grafana webhook alerts and creates incidents for investigation.

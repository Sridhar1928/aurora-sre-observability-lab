# Aurora SRE Observability Lab

A hands-on Site Reliability Engineering and observability lab demonstrating an end-to-end Kubernetes incident detection and AI-assisted investigation pipeline using Kubernetes, Prometheus, Grafana, and Aurora.

## Project Overview
The goal of this project was to build and test a realistic local SRE incident-management workflow.

The lab connects Kubernetes workload state to monitoring, alerting, and AI-assisted incident investigation.

## Architecture
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
## Technology Stack
- Docker Desktop
- Kubernetes
- kubectl
- Helm
- Kubernetes Metrics Server
- Prometheus
- Prometheus Operator
- kube-state-metrics
- Prometheus Node Exporter
- Alertmanager
- Grafana
- Aurora by Arvo AI
- OpenRouter
- Ubuntu / WSL

## Repository Structure
```text
aurora-sre-observability-lab/
├── README.md
├── .gitignore
├── kubernetes/
│   └── demo-app.yaml
├── monitoring/
│   └── README.md
├── docs/
│   ├── architecture.md
│   ├── setup.md
│   ├── imagepullbackoff-incident.md
│   ├── deployment-outage-incident.md
│   └── troubleshooting.md
└── screenshots/
```

## Kubernetes Demo Application
The lab uses a lightweight NGINX application deployed in the aurora-lab namespace.
Deploy the application with:
```bash
kubectl apply -f kubernetes/demo-app.yaml
```

Verify the workload:
```bash
kubectl get pods -n aurora-lab
```

Healthy state: 
```text
READY   STATUS
1/1     Running
```

## Kubernetes Metrics
Kubernetes Metrics Server was configured so CPU and memory resource metrics could be queried using:
```bash
kubectl top nodes
```
```bash
kubectl top pods -n aurora-lab
```
During setup, Metrics Server initially could not scrape the Docker Desktop kubelet because of local certificate validation.

For this development lab, Metrics Server was configured with:
```text
--kubelet-insecure-tls
```
This workaround was used only for the local test environment.

## Prometheus Monitoring
The Prometheus Community kube-prometheus-stack was deployed using Helm.

The monitoring environment included:

- Prometheus
- Prometheus Operator
- Alertmanager
- kube-state-metrics
- Prometheus Node Exporter
The stack's bundled Grafana installation was disabled because this lab uses a separate Grafana instance integrated with Aurora.

## Grafana and Prometheus
Grafana was connected to the Prometheus instance monitoring the Kubernetes cluster.

A Prometheus port-forward was used for the local lab:
```bash
kubectl port-forward service/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
```
Grafana connected to Prometheus using:
```text
 http://host.docker.internal:9090
 ```

 ## Kubernetes Workload Metric
The following PromQL query was used to monitor the available replicas of the demo-app Deployment:
```promql
kube_deployment_status_replicas_available{namespace="aurora-lab",deployment="demo-app"}
```
The healthy value observed during the lab was:
```text
1
```
A Grafana alert rule named demo-app unavailable was configured to trigger when the number of available replicas dropped below 1.

The alert notification was routed to Aurora through the configured Grafana webhook contact point.

## Incident 1: ImagePullBackOff
The first controlled incident tested Kubernetes deployment troubleshooting.

An invalid NGINX image was intentionally configured:

nginx:does-not-exist

The new Pod entered the following failure states:

ErrImagePull and ImagePullBackOff

Kubernetes events showed that the container image could not be retrieved.

Aurora's connected Kubernetes agent was used to investigate the workload and Kubernetes state.

The deployment was recovered using:
```bash
kubectl rollout undo deployment/demo-app -n aurora-lab
```
Recovery was verified using:
```bash
kubectl rollout status deployment/demo-app -n aurora-lab
```
The application returned to the working nginx:latest image and a healthy 1/1 Running state.

Detailed incident notes are available in:

docs/imagepullbackoff-incident.md

## Incident 2: Deployment Availability Outage
The second controlled incident tested the complete Kubernetes monitoring and alerting pipeline.

The demo-app Deployment was intentionally scaled to zero replicas to simulate a workload outage.

The outage was triggered using:
```bash
kubectl scale deployment/demo-app --replicas=0 -n aurora-lab
```
The Deployment reached zero available replicas, and kube-state-metrics exposed the state change to Prometheus.

Grafana evaluated the demo-app unavailable alert rule and transitioned the alert into the firing state.

The alert was delivered through the Grafana webhook to Aurora, where a new incident was automatically created.

After validating the alert pipeline, the application was restored using:
```bash
kubectl scale deployment/demo-app --replicas=1 -n aurora-lab
```
Recovery was verified using:
```bash
kubectl get pods -n aurora-lab
```
The demo-app Pod returned to a healthy 1/1 Running state.

Detailed incident notes are available in:
`docs/deployment-outage-incident.md`

## End-to-End Result
The lab successfully validated the following incident flow:
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
```

The workload recovery path was also validated by restoring the Kubernetes Deployment to one healthy replica.

## Troubleshooting Highlights

### Metrics API Not Available
Initially, kubectl top nodes returned:

Metrics API not available

Kubernetes Metrics Server was installed and configured to provide resource metrics.

### Metrics Server Certificate Error
Metrics Server initially failed to scrape the Docker Desktop kubelet because of TLS certificate validation.

The issue was identified through Metrics Server logs and resolved for the local lab environment.

### Prometheus Node Exporter CrashLoopBackOff
Prometheus Node Exporter initially entered CrashLoopBackOff.

The Pod events showed a Docker Desktop mount-propagation compatibility issue. The DaemonSet configuration was adjusted for the local environment, after which Node Exporter reached 1/1 Running.

### Grafana and Prometheus Networking
Grafana ran in a separate Docker container while Prometheus ran inside Kubernetes.

Prometheus was exposed locally using Kubernetes port-forwarding, and Grafana connected through host.docker.internal:9090.

### Aurora AI Summary Limitation
The Kubernetes monitoring and alert-ingestion pipeline successfully delivered the real workload incident to Aurora.

The final AI-generated summary encountered an HTTP 402 response from OpenRouter because the configured account did not have sufficient remaining credit for the requested maximum token budget.

This limitation occurred at the external LLM request stage and did not affect validation of the Kubernetes → Prometheus → Grafana → Aurora alert-ingestion pipeline.

## Skills Demonstrated
- Kubernetes deployment and workload management
- Kubernetes incident troubleshooting
- Docker Desktop and WSL
- Helm package management
- Kubernetes Metrics Server
- Prometheus monitoring
- PromQL
- kube-state-metrics
- Grafana alerting
- Webhook integrations
- Observability architecture
- SRE incident simulation
- Root cause investigation
- Docker networking
- Kubernetes rollout and recovery
- AI-assisted SRE workflows

## Security
Secrets and environment-specific credentials are intentionally excluded from this repository.

The repository does not include:

- OpenRouter API keys
- .env files
- Vault tokens
- Passwords
- Private kubeconfig files
- Aurora agent tokens
- User-specific webhook URLs
Never commit credentials, API keys, or other secrets to a public Git repository.

## Credits
This project uses Aurora by Arvo AI as the open-source AI-powered incident-management and root-cause-analysis platform.

Aurora project:

https://github.com/Arvo-AI/aurora

Aurora is an external open-source project used and integrated within this lab. This repository contains my independent Kubernetes, monitoring, incident-simulation, troubleshooting, and observability work.

## Project Purpose
This portfolio project demonstrates a practical combination of:

SRE + Kubernetes + Prometheus + Grafana + Observability + AI-assisted Incident Management
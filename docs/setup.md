## Lab Setup
### Prerequisites
The lab environment uses:

- Docker Desktop
- Docker Desktop Kubernetes
- Ubuntu / WSL
- kubectl
- Helm
- Grafana
- Aurora

## Deploy Demo Application
Apply the Kubernetes manifest:
```bash
kubectl apply -f kubernetes/demo-app.yaml
```
Verify the Deployment:
```bash
kubectl get pods -n aurora-lab
```
A healthy deployment should show the demo-app Pod in the 1/1 Running state.

## Monitoring Setup
The Prometheus monitoring stack was installed using Helm.

The stack included:

- Prometheus
- Prometheus Operator
- Alertmanager
- kube-state-metrics
- Prometheus Node Exporter
The bundled Grafana installation was disabled because the lab uses a separate Grafana instance.

## Prometheus Access
Prometheus was exposed locally using:
```bash
kubectl port-forward service/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
```
Grafana connected to Prometheus using:
```text
http://host.docker.internal:9090
```

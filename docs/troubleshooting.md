### Troubleshooting
This document summarizes the main technical issues encountered while building the local SRE observability lab.

## Metrics API Not Available
Initially, the following command failed:
```bash
kubectl top nodes
```
The cluster returned:
```text
Metrics API not available
```
Kubernetes Metrics Server was installed to provide the Metrics API.

## Metrics Server TLS Certificate Error
Metrics Server initially failed to scrape the Docker Desktop kubelet.

The logs reported a certificate validation error because the kubelet certificate did not contain the node IP address in its Subject Alternative Names.

For this local development lab, Metrics Server was configured with:
```text
--kubelet-insecure-tls
```
After applying the configuration, Metrics Server became healthy and kubectl top successfully returned node and Pod metrics.

This workaround was used only for the local lab environment.

## Prometheus Node Exporter CrashLoopBackOff
Prometheus Node Exporter initially failed to start and entered CrashLoopBackOff.

The Pod details reported:
```text
path / is mounted on / but it is not a shared or slave mount
```
The issue was caused by mount-propagation compatibility in the local Docker Desktop Kubernetes environment.

The Node Exporter DaemonSet configuration was adjusted for the local lab environment.

After the change, Node Exporter reached a healthy state:

'1/1 Running'

## Grafana and Prometheus Networking
Grafana ran in a separate Docker container while Prometheus ran inside Kubernetes.

Prometheus was exposed locally using Kubernetes port-forwarding.

Grafana connected to Prometheus using:
```text
http://host.docker.internal:9090
```

## OpenRouter Credit Limitation
Aurora successfully received the Kubernetes-backed Grafana incident, but final AI summary generation failed during the external LLM request.

The Aurora worker logs showed an HTTP 402 response because the requested maximum token budget exceeded the remaining available OpenRouter credit.

The Kubernetes → Prometheus → Grafana → Aurora monitoring and alert-ingestion pipeline itself completed successfully.

No additional credit was purchased for this portfolio lab.
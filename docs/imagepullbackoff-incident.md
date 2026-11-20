## Incident 1: ImagePullBackOff
### Scenario
A controlled Kubernetes failure was introduced by configuring the `demo-app` Deployment with an invalid container image:

nginx:does-not-exist

## Observed Failure
The replacement Pod failed to start and entered:
- ErrImagePull
- ImagePullBackOff
Kubernetes events showed that the configured container image could not be retrieved.

## Investigation
The incident investigation included checking:

- Deployment state
- ReplicaSets
- Pod status
- Container image configuration
- Kubernetes events
- Node health
The invalid container image was identified as the cause of the failed rollout.

## Recovery
The Deployment was rolled back using:
```bash
kubectl rollout undo deployment/demo-app -n aurora-lab
```
The rollout was verified using:
```bash
kubectl rollout status deployment/demo-app -n aurora-lab
```
The workload returned to the working nginx:latest image and reached a healthy 1/1 Running state.

## Result
This incident validated a complete Kubernetes troubleshooting workflow:
```text
Deployment change
      ↓
Invalid image
      ↓
ImagePullBackOff
      ↓
Investigation
      ↓
Root cause identified
      ↓
Rollback
      ↓
Healthy workload
```
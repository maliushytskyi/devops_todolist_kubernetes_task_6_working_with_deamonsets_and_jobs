# Deploying the DaemonSet and CronJob

## Prerequisites
- A running cluster and `kubectl` configured for it
- The `mateapp` namespace, with the todoapp Deployment and ClusterIP Service already deployed in it

```bash
kubectl get ns mateapp || kubectl create namespace mateapp
kubectl get svc -n mateapp
```

## Deploy
```bash
kubectl apply -f daemonset.yml
kubectl apply -f cronjob.yml
```

## Validate

### DaemonSet
```bash
kubectl get daemonset -n mateapp
kubectl get pods -n mateapp -l app=todoapp-pinger -o wide
kubectl logs -n mateapp -l app=todoapp-pinger --tail=20
```
Expected: one pod per node, and a new `200` line roughly every 5 seconds.

### CronJob
```bash
kubectl get cronjob -n mateapp
kubectl get jobs -n mateapp
kubectl logs -n mateapp job/<job-name>
```
To trigger a run without waiting 4 minutes:
```bash
kubectl create job --from=cronjob/todoapp-healthcheck manual-test -n mateapp
kubectl logs -n mateapp job/manual-test
```
Expected: the `/api/health` response from todoapp, and Jobs showing `COMPLETIONS 1/1`.

## Cleanup
```bash
kubectl delete -f daemonset.yml -f cronjob.yml
```
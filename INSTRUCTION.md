# Deploying the DaemonSet and CronJob

Both manifests live in `.infrastructure/` and belong to the `mateapp` namespace.

## Prerequisites

- A running Kubernetes cluster (e.g. minikube) and `kubectl` configured for it
- The `mateapp` namespace with the todoapp Deployment and its ClusterIP Service (`todoapp-service`) already deployed

```bash
kubectl get ns mateapp || kubectl create namespace mateapp
kubectl get svc -n mateapp
```


## Deploy

```bash
kubectl apply -f .infrastructure/daemonset.yml
kubectl apply -f .infrastructure/cronjob.yml
```

## Validate

### DaemonSet

```bash
kubectl get daemonset -n mateapp
kubectl get pods -n mateapp -l app=todoapp-daemonset -o wide
kubectl logs -n mateapp -l app=todoapp-daemonset --tail=20
```

Expected: `READY` equals `DESIRED`, one pod per node, and a new `/api/health` response in the logs roughly every 5 seconds.

### CronJob

```bash
kubectl get cronjob -n mateapp
kubectl get jobs -n mateapp -l app=todoapp-cronjob
```

To avoid waiting 4 minutes, trigger a run manually and read its logs:

```bash
kubectl create job --from=cronjob/todoapp-cronjob manual-test -n mateapp
kubectl logs -n mateapp job/manual-test
```

Expected: the health response from todoapp, and Jobs with `COMPLETIONS 1/1`.

The CronJob keeps 10 successful and 5 failed Job runs in history:

```bash
kubectl get cronjob todoapp-cronjob -n mateapp \
  -o jsonpath='{.spec.successfulJobsHistoryLimit} {.spec.failedJobsHistoryLimit} {.spec.concurrencyPolicy}{"\n"}'
```

Expected output: `10 5 Allow`

## Cleanup

```bash
kubectl delete -f .infrastructure/daemonset.yml -f .infrastructure/cronjob.yml
kubectl delete job manual-test -n mateapp
```
# GCP Discovery GKE Secrets

Premises
```
gcloud components update
```

1. Listing containers

```
gcloud container clusters list
```

2. Getting credentials

```
gcloud container clusters get-credentials CLUSTER_NAME --region REGION
```

3. Getting namespaces and workloads

```
kubectl get ns
```

4. Getting workloads

```
kubectl get deploy,statefulset,daemonset -A
```

5. Getting all Service Accounts

```
kubectl get sa -A > ServiceAccounts.txt
```

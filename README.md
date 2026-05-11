# GCP Discovery GKE Secrets

Premises
```
gcloud components update
```

1. Create Kubernetes map - Discovering :
- clusters
- namespaces
- workloads
- service accounts
- ingressos
- pipelines
- ferramentas instaladas

`Listing containters`
```
gcloud container clusters list > 1.1-all_containers.txt
```

`Getting credentials`
```
gcloud container clusters get-credentials CLUSTER_NAME --region REGION
```

2. Discovering namespaces and workloads

`Getting namespaces and workloads`
```
kubectl get ns  > 2.1-all_namespaces.txt
```

`Getting workloads`
```
kubectl get deploy,statefulset,daemonset -A  > 2.2-all_workloads.txt
```

`Getting all Service Accounts`
```
kubectl get sa -A > ServiceAccounts.txt  > 2.3-all_service_accounts.txt
```

`Getting all Pods`
```
kubectl get pods -A > Pods.txt  > 2.4-all_pods.txt
```

3. Kubernetes secrets discovering

- are there Kubernetes Secrets?
- only used Secret Manager?
- is there a mix?
- is there hardcoded secret?

```
kubectl get secrets -A > 3.1_all_secrets.txt
```

`Classifying as:`
- opaque
- docker-registry
- tls
- helm
- service-account-token

`Looking for workloads references`

4. Discovering GCP Secret Manager usage

`Getting all secrets`
```
gcloud secrets list > 4.1_secrets_list.txt
```

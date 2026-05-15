# 1. List existing Kubernetes namespaces

```bash
kubectl get ns
```

---

# 2. Create the test namespace

```bash
kubectl create namespace ns-test-secret
```

Validate:

```bash
kubectl get ns
```

---

# 3. Create the 3 secrets inside the namespace

## Secret 1

```bash
kubectl create secret generic secret-test-1 \
  -n ns-test-secret \
  --from-literal=key1=value1 \
  --from-literal=key2=value2 \
  --from-literal=key3=value3
```

---

## Secret 2

```bash
kubectl create secret generic secret-test-2 \
  -n ns-test-secret \
  --from-literal=key1=value1 \
  --from-literal=key2=value2 \
  --from-literal=key3=value3
```

---

## Secret 3

```bash
kubectl create secret generic secret-test-3 \
  -n ns-test-secret \
  --from-literal=key1=value1 \
  --from-literal=key2=value2 \
  --from-literal=key3=value3
```

---

# 4. List the created secrets

```bash
kubectl get secrets -n ns-test-secret
```

You should see:

* secret-test-1
* secret-test-2
* secret-test-3

---

# 5. Validate the secret values

## Secret 1

```bash
kubectl get secret secret-test-1 \
  -n ns-test-secret \
  -o json
```

---

## View decoded values

### key1

```bash
kubectl get secret secret-test-1 \
  -n ns-test-secret \
  -o jsonpath='{.data.key1}' | base64 -d
echo
```

### key2

```bash
kubectl get secret secret-test-1 \
  -n ns-test-secret \
  -o jsonpath='{.data.key2}' | base64 -d
echo
```

### key3

```bash
kubectl get secret secret-test-1 \
  -n ns-test-secret \
  -o jsonpath='{.data.key3}' | base64 -d
echo
```

---

# 6. List only the secret names

This will probably be used in your script:

```bash
kubectl get secrets \
  -n ns-test-secret \
  --no-headers \
  -o custom-columns=":metadata.name"
```

Or:

```bash
kubectl get secrets \
  -n ns-test-secret \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

---

# 7. Run your script

Example:

```bash
./main.sh
```

or

```bash
bash main.sh
```

---

# 8. Expected script behavior

The expected workflow is:

## For each Kubernetes secret:

Example:

* secret-test-1
* secret-test-2
* secret-test-3

The script should:

### STEP A — Read values from the Kubernetes secret

Example:

```json
{
  "key1": "value1",
  "key2": "value2",
  "key3": "value3"
}
```

---

### STEP B — Check if the secret already exists in AWS Secrets Manager

Example:

```bash
aws secretsmanager describe-secret \
  --secret-id secret-test-1
```

---

## Scenario 1 — Secret EXISTS

The script should:

1. Read the current content:

```bash
aws secretsmanager get-secret-value \
  --secret-id secret-test-1
```

2. Iterate through the keys
3. Update/add the values
4. Execute:

```bash
aws secretsmanager update-secret \
  --secret-id secret-test-1 \
  --secret-string '<json>'
```

---

## Scenario 2 — Secret DOES NOT EXIST

The script should:

1. Create the secret:

```bash
aws secretsmanager create-secret \
  --name secret-test-1 \
  --secret-string '{}'
```

2. Iterate through the Kubernetes values
3. Update the JSON
4. Execute:

```bash
aws secretsmanager update-secret \
  --secret-id secret-test-1 \
  --secret-string '<final_json>'
```

---

# 9. Validate secrets in AWS Secrets Manager

List all secrets:

```bash
aws secretsmanager list-secrets
```

---

## View one secret content

```bash
aws secretsmanager get-secret-value \
  --secret-id secret-test-1
```

---

# 10. Remove Kubernetes secrets

## Secret 1

```bash
kubectl delete secret secret-test-1 -n ns-test-secret
```

## Secret 2

```bash
kubectl delete secret secret-test-2 -n ns-test-secret
```

## Secret 3

```bash
kubectl delete secret secret-test-3 -n ns-test-secret
```

Or delete all at once:

```bash
kubectl delete secrets --all -n ns-test-secret
```

---

# 11. Validate removal

```bash
kubectl get secrets -n ns-test-secret
```

---

# 12. Remove the namespace

```bash
kubectl delete namespace ns-test-secret
```

---

# 13. Final validation

```bash
kubectl get ns
```

The namespace `ns-test-secret` should no longer exist.

---

# Important tip for your script

You will probably want to ignore Kubernetes default secrets such as:

```text
default-token-xxxxx
```

or

```text
builder-token-xxxxx
```

So filter only:

```bash
kubectl get secrets \
  -n ns-test-secret \
  --field-selector type=Opaque
```

This way you only retrieve manually created secrets.

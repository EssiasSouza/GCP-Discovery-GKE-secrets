Your bash logic is probably fine now.

This is still IAM permission propagation or policy mismatch.

The key clue is:

```text id="w7x3d1"
because no identity-based policy allows the secretsmanager:GetSecretValue action
```

AWS is explicitly saying:

> "I evaluated the policies attached to this IAM user and did not find GetSecretValue allowed for this resource."

So focus on IAM, not bash.

---

Most likely causes now:

# 1. Policy not yet attached/applied

AWS IAM changes can take some seconds/minutes to propagate.

Test:

```bash id="x2m8f9"
aws sts get-caller-identity
```

to confirm the user.

Then wait 1–2 minutes and retry.

---

# 2. Wrong resource ARN pattern

Your secret resource is:

```text id="p4k1z7"
gke-gringo-staging/default/teste-essias
```

But Secrets Manager internally stores ARNs like:

```text id="n8v2q5"
arn:aws:secretsmanager:sa-east-1:498638359097:secret:gke-gringo-staging/default/teste-essias-ABC123
```

So your policy should be:

```json id="t7c9r4"
"Resource": [
  "arn:aws:secretsmanager:sa-east-1:498638359097:secret:gke-gringo-staging/*"
]
```

OR even temporarily:

```json id="h5s1x8"
"Resource": "*"
```

to validate the issue.

---

# 3. Easiest debug approach

Temporarily use this policy:

```json id="j3w6n2"
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "secretsmanager:*",
      "Resource": "*"
    }
  ]
}
```

If everything suddenly works:

* your script is fine
* your AWS CLI is fine
* your resource ARN restriction is wrong

Then you can tighten permissions afterward.

This is the fastest way to isolate the problem.

---

# 4. Another important thing

Your command should be quoted:

Current:

```bash id="k1y4v6"
--secret-id $SECRET
```

Better:

```bash id="u9r7m3"
--secret-id "$SECRET"
```

and:

```bash id="f6n2p8"
echo "$json"
echo "$existing_values"
```

---

# 5. Your loop is still dangerous

This is still problematic:

```bash id="r2q5w1"
for i in $new_values; do
```

because secrets containing spaces/newlines will break.

Better:

```bash id="m8c4t7"
while IFS= read -r i; do
```

---

# 6. Major bug: invalid JSON handling

This line:

```bash id="d4v9s2"
json=$(echo $json | jq ...)
```

will corrupt JSON if it contains spaces/newlines.

Use:

```bash id="y7k3n6"
json=$(echo "$json" | jq ...)
```

---

So right now:

* your bash code has some improvements needed
* but the CURRENT error is still IAM policy related, not shell related

The fastest way forward is:

1. temporarily allow:

   ```json
   "Action": "secretsmanager:*",
   "Resource": "*"
   ```

2. confirm script works

3. then reduce permissions carefully until least privilege is achieved.

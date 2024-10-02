```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      serviceAccountName: my-service-account  # Specify your service account here
      containers:
      - name: my-container
        image: my-image:tag
        command: ["/bin/sh"]
        args:
        - -c
        - |
          TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
          export SERVICE_ACCOUNT_TOKEN=$TOKEN
          exec your_actual_command
        volumeMounts:
        - name: service-account-token
          mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          readOnly: true
      volumes:
      - name: service-account-token
        projected:
          sources:
          - serviceAccountToken:
              expirationSeconds: 3600
              audience: api  # This is optional and can be customized


```


You're correct. When you manually create a `Secret` of type `kubernetes.io/service-account-token` and name it explicitly (e.g., `my-service-account-token`), the token name won't be automatically assigned by Kubernetes as in the previous case where the token secret was automatically generated and associated with the service account.

In this case, you need to refer to the explicit name you've given the token secret (`my-service-account-token`) instead of dynamically fetching it. Here's the updated workflow:


To regenerate the secret of type `kubernetes.io/service-account-token` with the correct association to your service account (`my-service-account`), here is the YAML manifest for you:

### Regenerated Secret YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-service-account-token
  annotations:
    kubernetes.io/service-account.name: "my-service-account"
type: kubernetes.io/service-account-token
```

### Notes:
- The `name: my-service-account-token` in the metadata section specifies the name of the secret.
- The `annotations` section includes the `kubernetes.io/service-account.name` annotation, which links the secret to the service account `my-service-account`.
- The `type: kubernetes.io/service-account-token` ensures that Kubernetes will automatically populate this secret with the token associated with the `my-service-account` service account.

Once you apply this YAML, Kubernetes will generate and fill in the token data for the service account.

You can apply this YAML using `kubectl`:

```bash
kubectl apply -f secret.yaml
```

Let me know if you need further customization or assistance with this process!

### Updated Kubernetes Job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: token-extractor-job
spec:
  template:
    spec:
      containers:
      - name: token-extractor
        image: bitnami/kubectl:latest
        command: ["/bin/sh", "-c"]
        args:
        - |
          TOKEN=$(kubectl get secret my-service-account-token -o jsonpath='{.data.token}' | base64 --decode)
          kubectl create secret generic app-secret --from-literal=SERVICE_ACCOUNT_TOKEN=$TOKEN --dry-run=client -o yaml | kubectl apply -f -
      restartPolicy: OnFailure
```

### Key Changes:
- **Fixed Secret Name:**
  - Instead of dynamically fetching the token name, we directly reference the secret by the name you explicitly gave it: `my-service-account-token`.

- **Token Extraction:**
  - We use the same `kubectl get secret` command, but now with the fixed secret name (`my-service-account-token`) to extract the token data.

- **Base64 Decode:**
  - The token is stored as base64-encoded, so the `base64 --decode` is used to decode it before applying it to the new secret.

This approach should work since the secret is being manually created with a known name. Now, you can extract the token from the specific `my-service-account-token` secret and apply it as part of your Kubernetes Job.

Does this meet your requirements, or would you like any further adjustments?

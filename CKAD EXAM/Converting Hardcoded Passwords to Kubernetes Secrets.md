
## Problem: Deployment with hardcoded passwords (INSECURE)

Before we fix this, here's what the insecure deployment looks like:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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
      containers:
      - name: app
        image: my-app:latest
        env:
        - name: DB_HOST
          value: "postgres.example.com"
        - name: DB_USER
          value: "admin"
        - name: DB_PASSWORD
          value: "hardcoded-password-123"  # ❌ INSECURE! Visible in git, kubectl describe, etc.
        - name: API_KEY
          value: "secret-api-key-456"      # ❌ INSECURE!
```

## Solution: Use Kubernetes Secrets

### Step 1: Create the Secret

This stores sensitive data separately from the deployment config:

```bash
kubectl create secret generic my-app-secrets \
  --from-literal=db-password='hardcoded-password-123' \
  --from-literal=api-key='secret-api-key-456'
```

Verify the secret was created:

```bash
kubectl get secrets
kubectl describe secret my-app-secrets
```

### Step 2: Generate deployment YAML

```bash
kubectl create deployment my-app \
  --image=my-app:latest \
  --dry-run=client -o yaml > deployment.yaml
```

This generates:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-app
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-app
    spec:
      containers:
      - image: my-app:latest
        name: my-app
        resources: {}
status: {}
```

### Step 3: Edit the deployment

```bash
vim deployment.yaml
```

Update the file to look like this (the SECURE version):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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
      containers:
      - name: my-app
        image: my-app:latest
        env:
        - name: DB_HOST
          value: "postgres.example.com"
        - name: DB_USER
          value: "admin"
        - name: DB_PASSWORD              # ✅ SECURE: Loaded from Secret
          valueFrom:
            secretKeyRef:
              name: my-app-secrets
              key: db-password
        - name: API_KEY                  # ✅ SECURE: Loaded from Secret
          valueFrom:
            secretKeyRef:
              name: my-app-secrets
              key: api-key
```

### Step 4: Apply the deployment

```bash
kubectl apply -f deployment.yaml
```

## Verification Commands

Check if deployment is running:

```bash
kubectl get deployments
kubectl get pods
```

Verify environment variables are loaded:

```bash
kubectl exec -it deployment/my-app -- env | grep -E "DB_PASSWORD|API_KEY"
```

View pod environment variables without execing:

```bash
kubectl describe pod <pod-name> | grep -A 10 "Environment:"
```

## Useful Debugging & Management

Decode a secret value (for debugging):

```bash
kubectl get secret my-app-secrets -o jsonpath='{.data.db-password}' | base64 -d
```

Edit secret if needed:

```bash
kubectl edit secret my-app-secrets
```

Delete and recreate secret (for rotation):

```bash
kubectl delete secret my-app-secrets
kubectl create secret generic my-app-secrets \
  --from-literal=db-password='new-password' \
  --from-literal=api-key='new-api-key'
```

Restart deployment to pick up new secret values:

```bash
kubectl rollout restart deployment/my-app
```

## Why Use Secrets?

1. **Separation of concerns**: Config is separate from sensitive data
2. **Access control**: Use RBAC to control who can view secrets
3. **Easier rotation**: Update secrets without changing deployment YAML
4. **Better security**: Secrets can be encrypted at rest (with proper cluster config)
5. **Not in version control**: Secrets aren't committed to git with your manifests
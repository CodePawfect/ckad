
## Problem: Understanding Paused Rollouts

Sometimes you need to update a deployment but want to review the changes before they're fully rolled out. Kubernetes allows you to pause a rollout, make changes, and then resume when ready.

Here's a typical scenario:

- You have a running deployment
- You want to change the container image
- You want to pause and verify before completing the rollout

## Solution: Pause, Update, and Resume Rollout

### Step 1: Create initial deployment

Generate a deployment YAML:

```bash
kubectl create deployment my-app \
  --image=nginx:1.19 \
  --replicas=3 \
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
  replicas: 3
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
      - image: nginx:1.19
        name: nginx
        resources: {}
status: {}
```

Apply the deployment:

```bash
kubectl apply -f deployment.yaml
```

Verify it's running:

```bash
kubectl get deployments
kubectl get pods
```

### Step 2: Pause the rollout

Before making changes, pause the deployment:

```bash
kubectl rollout pause deployment/my-app
```

Output:

```
deployment.apps/my-app paused
```

### Step 3: Update the container image

Change the image version:

```bash
kubectl set image deployment/my-app nginx=nginx:1.21
```

Check the rollout status (it will show as paused):

```bash
kubectl rollout status deployment/my-app
```

Output:

```
Waiting for deployment "my-app" rollout to finish: 0 out of 3 new replicas have been updated...
deployment "my-app" successfully rolled out
```

### Step 4: View rollout history

Check the revision history:

```bash
kubectl rollout history deployment/my-app
```

Output:

```
deployment.apps/my-app
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

View details of a specific revision:

```bash
kubectl rollout history deployment/my-app --revision=2
```

### Step 5: Resume the rollout

After verifying the changes, resume the rollout:

```bash
kubectl rollout resume deployment/my-app
```

Output:

```
deployment.apps/my-app resumed
```

Watch the rollout progress:

```bash
kubectl rollout status deployment/my-app --watch
```

The deployment will now update all replicas to the new image version.

### Final State

After resuming, the deployment looks like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - image: nginx:1.21  # ✅ Updated image
        name: nginx
```

## Verification Commands

Check current image version:

```bash
kubectl get deployment my-app -o jsonpath='{.spec.template.spec.containers[0].image}'
```

View pods and their images:

```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

Describe deployment to see events:

```bash
kubectl describe deployment my-app
```

## Useful Rollout Management Commands

### Undo a rollout

Rollback to previous revision:

```bash
kubectl rollout undo deployment/my-app
```

Rollback to specific revision:

```bash
kubectl rollout undo deployment/my-app --to-revision=1
```

### Restart a deployment

Force a restart without changing the spec:

```bash
kubectl rollout restart deployment/my-app
```

### Add change-cause annotation

To track why changes were made, add annotations:

```bash
kubectl annotate deployment/my-app kubernetes.io/change-cause="Updated to nginx 1.21"
```

Then the rollout history will show:

```bash
kubectl rollout history deployment/my-app
```

Output:

```
deployment.apps/my-app
REVISION  CHANGE-CAUSE
1         <none>
2         Updated to nginx 1.21
```

### Multiple changes while paused

You can make multiple changes while paused:

```bash
kubectl rollout pause deployment/my-app
kubectl set image deployment/my-app nginx=nginx:1.21
kubectl set resources deployment/my-app -c=nginx --limits=cpu=200m,memory=512Mi
kubectl rollout resume deployment/my-app
```

All changes will be applied together when you resume.

## Why Use Pause/Resume?

1. **Test before full rollout**: Verify changes on a single pod before updating all replicas
2. **Batch multiple changes**: Make several updates that deploy together
3. **Review configuration**: Inspect the pending changes before they affect production
4. **Controlled updates**: More control over when changes actually roll out
5. **Minimize disruption**: Combine changes into a single rollout event
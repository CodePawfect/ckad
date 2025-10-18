## Problem: Running Containers Securely

By default, containers often run as root (UID 0), which is a security risk. You need to:

- Run containers as non-root users
- Drop unnecessary Linux capabilities
- Add specific capabilities when needed
- Apply principle of least privilege

**Example scenario**: You have a web app that doesn't need root privileges but the image defaults to root user.

## Solution: Use SecurityContext

SecurityContext allows you to configure security settings at the pod level or container level.

### Step 1: Create basic deployment

Generate a deployment YAML:

```bash
kubectl create deployment webapp \
  --image=nginx:alpine \
  --dry-run=client -o yaml > deployment.yaml
```

This generates:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        resources: {}
status: {}
```

### Step 2: Add SecurityContext with run-as-user

```bash
vim deployment.yaml
```

Add securityContext to run as non-root user:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      # Pod-level securityContext (applies to all containers)
      securityContext:
        runAsUser: 1000          # ✅ Run as UID 1000
        runAsGroup: 3000         # ✅ Run as GID 3000
        fsGroup: 2000            # ✅ Files created will be owned by GID 2000
        runAsNonRoot: true       # ✅ Enforce non-root
      containers:
      - name: nginx
        image: nginx:alpine
```

### Step 3: Add capabilities at container level

```bash
vim deployment.yaml
```

Configure capabilities (drop all, add only needed ones):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
        runAsNonRoot: true
      containers:
      - name: nginx
        image: nginx:alpine
        # Container-level securityContext (overrides pod-level)
        securityContext:
          allowPrivilegeEscalation: false  # ✅ Prevent privilege escalation
          capabilities:
            drop:
            - ALL                          # ✅ Drop all capabilities
            add:
            - NET_BIND_SERVICE             # ✅ Add only what's needed (bind to ports < 1024)
          readOnlyRootFilesystem: true     # ✅ Make root filesystem read-only
```

### Step 4: Apply the deployment

```bash
kubectl apply -f deployment.yaml
```

Output:

```
deployment.apps/webapp created
```

## Why Use SecurityContext?

1. **Defense in depth**: Limit blast radius if container is compromised
2. **Compliance**: Meet security standards and audit requirements
3. **Principle of least privilege**: Run with minimum necessary permissions
4. **Prevent privilege escalation**: Stop attackers from gaining root
5. **Multi-tenancy**: Isolate workloads from each other
6. **Production readiness**: Essential for secure production deployments
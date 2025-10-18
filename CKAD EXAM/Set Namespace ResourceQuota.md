## Problem: Limiting Resource Usage per Namespace

Prevent a single namespace from consuming all cluster resources. ResourceQuota limits:

- CPU and memory usage
- Number of objects (pods, services, etc.)
- Storage requests

## Solution: Create ResourceQuota

### Step 1: Create namespace

```bash
kubectl create namespace dev-team
```

### Step 2: Create ResourceQuota

```bash
kubectl create quota dev-quota \
  --namespace=dev-team \
  --hard=requests.cpu=4,requests.memory=8Gi,limits.cpu=8,limits.memory=16Gi,pods=10 \
  --dry-run=client -o yaml > resourcequota.yaml
```

This generates:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: dev-quota
  namespace: dev-team
spec:
  hard:
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
status: {}
```

### Step 3: Edit for additional limits (optional)

```bash
vim resourcequota.yaml
```

Add more resource types:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev-team
spec:
  hard:
    # Compute resources
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    
    # Object counts
    pods: "10"
    services: "5"
    configmaps: "10"
    persistentvolumeclaims: "5"
    secrets: "10"
    
    # Storage
    requests.storage: "50Gi"
```

### Step 4: Apply the ResourceQuota

```bash
kubectl apply -f resourcequota.yaml
```

### Step 5: Verify ResourceQuota

```bash
kubectl get resourcequota -n dev-team
```

Output:

```
NAME        AGE   REQUEST
dev-quota   10s   pods: 0/10, requests.cpu: 0/4, requests.memory: 0/8Gi, ...
```
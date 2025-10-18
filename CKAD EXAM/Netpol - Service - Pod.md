# Kubernetes NetworkPolicy Troubleshooting - Secure Pod Connectivity

## Task Description

We have already deployed:
* A pod named `secure-pod`
* A service named `secure-service` that targets this pod

Currently, both incoming and outgoing network connections to/from `secure-pod` are failing.

**Goal:** Fix the issue so that incoming connections from the pod `webapp-color` to `secure-pod` are successful.

**Requirements:**
1. Do not delete or recreate existing Kubernetes objects unless specifically asked
2. All resources are in the `default` namespace
3. The fix must be persistent

---

## Problem Analysis

### Initial Investigation
```bash
# Check existing resources
k get pods
k get svc
k get ep

# Verify service configuration
k describe svc secure-service
# ✓ Service exists with correct selector (run=secure-pod)
# ✓ Endpoints are populated (172.17.0.9:80)
# ✓ Port mapping is correct (80/TCP)

# Verify pod labels match service selector
k get pod secure-pod --show-labels
# ✓ Labels: run=secure-pod (matches service selector)

# Check for Network Policies (KEY FINDING!)
k get netpol
# NAME           POD-SELECTOR   AGE
# default-deny   <none>         4m22s

k describe netpol default-deny
```

### Root Cause Identified

**The `default-deny` NetworkPolicy is blocking all ingress traffic:**
- `PodSelector: <none>` → Applies to ALL pods in the namespace
- `Allowing ingress traffic: <none>` → No ingress rules = blocks ALL incoming connections
- `Not affecting egress traffic` → Outgoing traffic is not controlled

**Why connectivity fails:**
- The default-deny policy blocks all ingress by design
- No NetworkPolicy exists to allow traffic from `webapp-color` to `secure-pod`
- Service and endpoints are fine - the issue is purely network policy enforcement

---

## Solution Strategy

### Why NOT just edit/delete default-deny?

❌ **Don't delete default-deny** - It's a security best practice (deny-by-default)
❌ **Don't edit default-deny** - It should remain simple and block everything

✅ **Create a NEW NetworkPolicy** that allows specific traffic

### How NetworkPolicies Work (Critical Understanding)

**NetworkPolicies use OR logic:**
- If ANY policy allows traffic → ALLOWED ✅
- If NO policy allows traffic → DENIED ❌

This means our new "allow" policy will create an **exception** to the default-deny rule!

### Implementation Plan

1. Check labels on both pods (for selectors)
2. Create a new NetworkPolicy that:
   - Applies to `secure-pod` (podSelector)
   - Allows ingress from `webapp-color` (from.podSelector)
   - Allows traffic on port 80 (ports)
3. Apply and verify

---

## Solution Implementation

### Step 1: Gather Information
```bash
# Get labels for NetworkPolicy selectors
k get pod secure-pod --show-labels
# run=secure-pod

k get pod webapp-color --show-labels
# run=webapp-color (or similar)
```

### Step 2: Create NetworkPolicy YAML
```bash
vim allow-webapp-to-secure.yaml
```

In vim, write:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-webapp-to-secure
  namespace: default
spec:
  # Apply this policy to pods with label run=secure-pod
  podSelector:
    matchLabels:
      run: secure-pod
  
  # We're defining ingress rules
  policyTypes:
  - Ingress
  
  ingress:
  # Allow traffic from pods with label run=webapp-color
  - from:
    - podSelector:
        matchLabels:
          run: webapp-color
    # Only allow traffic on port 80 (the application port)
    ports:
    - protocol: TCP
      port: 80
```

**Key YAML Explanations:**
- `podSelector.matchLabels.run=secure-pod` - This policy applies ONLY to secure-pod
- `ingress.from.podSelector.matchLabels.run=webapp-color` - Only allow traffic FROM webapp-color
- `ports: 80/TCP` - Only allow on the application port
- This creates a precise whitelist: webapp-color → secure-pod:80

### Step 3: Apply and Verify
```bash
# Apply the NetworkPolicy
k apply -f allow-webapp-to-secure.yaml

# Verify both policies now exist
k get netpol
# NAME                      POD-SELECTOR      AGE
# default-deny              <none>            10m
# allow-webapp-to-secure    run=secure-pod    5s

# Describe the new policy to confirm configuration
k describe netpol allow-webapp-to-secure

# Test connectivity from webapp-color to secure-pod
k exec -it webapp-color -- curl http://secure-service
# Should return successful response! ✓

# Optional: Verify security - test from unauthorized pod
k run test-pod --image=nginx --rm -it -- curl http://secure-service --max-time 5
# Should timeout/fail - proves our policy is working correctly ✓
```

---
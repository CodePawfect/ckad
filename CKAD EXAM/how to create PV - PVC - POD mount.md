# CKAD Lightning Lab - Part 1 - Question 1

**Task:**
Create a PersistentVolume named `log-volume` with the following specifications:
- StorageClassName: `manual` (already created for you — do not create it again)
- AccessModes: ReadWriteMany (RWX)
- Capacity: 1Gi
- hostPath: `/opt/volume/nginx`

Next, create a PersistentVolumeClaim named `log-claim` that:
- Requests at least 200Mi of storage
- Binds to the `log-volume` created above

Finally, create a pod named `logger` that:
- Uses the image `nginx:alpine`
- Mounts the `log-claim` at the path `/var/www/nginx` inside the container

---

## Solution

### Step 1: Create PersistentVolume
No kubectl command exists to generate PV YAML, so create manually:
```bash
vim pv.yaml
```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /opt/volume/nginx
```
```bash
kubectl apply -f pv.yaml
```

### Step 2: Create PersistentVolumeClaim
No kubectl command exists to generate PVC YAML, so create manually:
```bash
vim pvc.yaml
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```
```bash
kubectl apply -f pvc.yaml
```

### Step 3: Create Pod with Volume Mount
```bash
kubectl run logger --image=nginx:alpine --dry-run=client -o yaml > pod.yaml
```
```bash
vi pod.yaml
```

Edit to add volume and volumeMount:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  containers:
  - name: logger
    image: nginx:alpine
    volumeMounts:
    - name: log-storage
      mountPath: /var/www/nginx
  volumes:
  - name: log-storage
    persistentVolumeClaim:
      claimName: log-claim
```
```bash
kubectl apply -f pod.yaml
```

### Verification
```bash
# Check PV
kubectl get pv log-volume

# Check PVC (should show STATUS: Bound)
kubectl get pvc log-claim

# Check Pod (should show STATUS: Running)
kubectl get pod logger

# Verify mount inside the pod
kubectl exec logger -- df -h | grep nginx
```

---

## Key Points
- No kubectl generators exist for PV and PVC - must write YAML manually
- Pod can be generated with `kubectl run` but requires manual editing for volumes
- PVC automatically binds to PV when storageClassName and accessModes match
- PVC requests 200Mi, PV provides 1Gi - binding works because PVC ≤ PV

#flashcards/ckad 
create a minimal ymal to create a PersistentVolume with hostPath
?
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /opt/volume/nginx
+++
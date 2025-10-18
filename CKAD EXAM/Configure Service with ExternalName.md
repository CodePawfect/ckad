## Problem: Accessing External Services from Pods

You have an external service (database, API, etc.) hosted outside your Kubernetes cluster, and you want:

- Pods to access it using a consistent internal DNS name
- Ability to switch to internal service later without changing app code
- Clean abstraction between service location and service name

**Example scenario**: Your database is at `prod-db.company.com:5432` but you want pods to connect to `database:5432`.

## Solution: Use ExternalName Service

An ExternalName service creates a DNS CNAME record that redirects to an external hostname.

### Step 1: Create ExternalName service using imperative command

Unfortunately, there's no direct imperative command for ExternalName services, so we use `--dry-run`:

```bash
kubectl create service externalname database \
  --external-name prod-db.company.com \
  --dry-run=client -o yaml > externalname-service.yaml
```

This generates:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: database
  name: database
spec:
  externalName: prod-db.company.com
  type: ExternalName
status:
  loadBalancer: {}
```

### Step 2: Edit the service (if needed)

```bash
vim externalname-service.yaml
```

Clean version (remove unnecessary fields):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  type: ExternalName
  externalName: prod-db.company.com  # External DNS name
```

**Important notes:**

- No `selector` field (doesn't route to pods)
- No `ports` field (DNS redirect only, port handled by external service)
- No `clusterIP` (service type ExternalName doesn't get one)

### Step 3: Apply the service

```bash
kubectl apply -f externalname-service.yaml
```

Output:

```
service/database created
```

### Step 4: Verify the service

Check the service was created:

```bash
kubectl get service database
```

Output:

```
NAME       TYPE           CLUSTER-IP   EXTERNAL-IP           PORT(S)   AGE
database   ExternalName   <none>       prod-db.company.com   <none>    10s
```

Describe the service:

```bash
kubectl describe service database
```

Output:

```
Name:              database
Namespace:         default
Labels:            app=database
Annotations:       <none>
Selector:          <none>
Type:              ExternalName
IP:                
External Name:     prod-db.company.com
Session Affinity:  None
Events:            <none>
```

### Step 5: Test DNS resolution from a pod

Create a test pod:

```bash
kubectl run test-pod --image=busybox:1.28 --rm -it --restart=Never -- sh
```

Inside the pod, test DNS resolution:

```bash
nslookup database
```

Output:

```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      database
Address 1: prod-db.company.com
```

The service returns a CNAME pointing to `prod-db.company.com`!

Test with full DNS name:

```bash
nslookup database.default.svc.cluster.local
```

### Complete Example: Application Using ExternalName Service

Create a deployment that uses the service:

```bash
kubectl create deployment my-app \
  --image=postgres:alpine \
  --dry-run=client -o yaml > app-deployment.yaml
```

```bash
vim app-deployment.yaml
```

Update to connect to the database service:

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
          value: "database"  # ✅ Uses ExternalName service
        - name: DB_PORT
          value: "5432"
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
```

Apply:

```bash
kubectl apply -f app-deployment.yaml
```

Now the app connects to `database:5432` which resolves to `prod-db.company.com:5432`.

## Migration Example: External to Internal

Later, when you move the database into the cluster, simply change the service type:

```bash
kubectl delete service database
```

Create a normal ClusterIP service:

```bash
kubectl expose deployment postgres-db \
  --name=database \
  --port=5432 \
  --target-port=5432
```

Your app code **doesn't change** - still connects to `database:5432`!

## Verification Commands

Check service type:

```bash
kubectl get service database -o jsonpath='{.spec.type}'
```

Check external name:

```bash
kubectl get service database -o jsonpath='{.spec.externalName}'
```

Test connectivity from a pod:

```bash
kubectl run -it --rm debug --image=alpine --restart=Never -- sh
apk add curl
curl -v telnet://database:5432
```

## Common Use Cases

### Use Case 1: External Database

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  type: ExternalName
  externalName: db.prod.company.com
```

### Use Case 2: External API

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-api
spec:
  type: ExternalName
  externalName: api.stripe.com
```

### Use Case 3: Service in Another Cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: shared-service
spec:
  type: ExternalName
  externalName: service.other-cluster.example.com
```

### Use Case 4: Development vs Production

In dev environment:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  type: ExternalName
  externalName: dev-db.company.local
```

In prod environment:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  type: ClusterIP  # Internal service
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

## Important Limitations

1. **No port mapping**: ExternalName only does DNS redirect, can't remap ports
2. **No load balancing**: Just a DNS CNAME, external service handles load balancing
3. **No health checks**: Kubernetes doesn't monitor the external endpoint
4. **Requires DNS**: The external name must be resolvable by cluster DNS
5. **No IP addresses**: Can only use hostnames, not IP addresses (use Endpoints for IPs)

## Alternative: Using Endpoints for External IPs

If you need to use an IP address instead of hostname:

```bash
# Create service without selector
kubectl create service clusterip database \
  --tcp=5432:5432 \
  --dry-run=client -o yaml > database-service.yaml
```

Remove the selector, apply:

```bash
kubectl apply -f database-service.yaml
```

Manually create endpoints:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Endpoints
metadata:
  name: database  # Must match service name
subsets:
- addresses:
  - ip: 10.0.1.50  # External IP
  ports:
  - port: 5432
EOF
```

## Why Use ExternalName Services?

1. **Abstraction**: Decouple service location from service name
2. **Easy migration**: Move services in/out of cluster without app changes
3. **Consistency**: Uniform DNS naming across environments
4. **Flexibility**: Switch between external and internal services seamlessly
5. **Clean code**: Apps use simple names like `database` instead of full hostnames
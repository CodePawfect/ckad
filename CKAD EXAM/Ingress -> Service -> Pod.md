# Ingress → Service → Pod Flow

## Port Mapping Rule
**CRITICAL:** Ingress port = Service `port` (NOT `targetPort`)

## The Flow
```
External Request
    ↓
Ingress (host + path)
    ↓
Service:port          ← Ingress references THIS
    ↓
Pod:targetPort        ← Service forwards to THIS
```

## Example
```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  ports:
  - port: 8080        # ← Ingress uses THIS
    targetPort: 3000  # ← Pod container listens here
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-svc
            port:
              number: 8080  # ← Match Service port
```

## Quick Check

| Resource | Port Field | Value |
|----------|------------|-------|
| Service | `port:` | 8080 |
| Service | `targetPort:` | 3000 |
| **Ingress** | `port.number:` | **8080** ✅ |
| ~~Ingress~~ | ~~port.number:~~ | ~~3000~~ ❌ |

## Exam Tip
1. Check Service: `kubectl get svc my-svc -o yaml | grep port`
2. Use the `port:` value (not `targetPort:`) in Ingress

---
**Remember:** Ingress talks to Service, Service talks to Pod
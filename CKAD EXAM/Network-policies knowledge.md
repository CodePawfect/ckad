# Ingress
![[Pasted image 20251018163428.png]]

![[Pasted image 20251018163530.png]]

## Allow incoming traffic from dedicated pods
![[Pasted image 20251018163833.png]]
matchLabels is AND.
For OR we need instead:
``` yaml
podSelector: 
  matchExpressions:n 
  - key: app 
    operator: In 
    values: 
    - foo 
    - bar # app=foo OR app=bar
```

If we add ports to the ingress rule, we can allow traffic e.g. only from port 3306
![[Pasted image 20251018164136.png]]

![[Pasted image 20251018171928.png]]

## Allow incoming traffic from other Namespaces
How to allow traffic from other Namespaces?
![[Pasted image 20251018172027.png]]

![[Pasted image 20251018172143.png]]

## Allow incoming traffic from IP blocks
How to allow traffic from IP blocks?
![[Pasted image 20251018172313.png]]

# Egress
![[Pasted image 20251018172426.png]]

![[Pasted image 20251018173759.png]]
### blocks DNS too
![[Pasted image 20251018173837.png]]

we add another rule to allow egress to DNS
![[Pasted image 20251018173956.png]]

## allow outgoing traffic to other pods in any namespace but not outside the cluster
![[Pasted image 20251018174319.png]]


![[Pasted image 20251018163428.png]]

![[Pasted image 20251018163530.png]]

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


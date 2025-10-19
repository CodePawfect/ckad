### Get 'app=v2' and not 'tier=frontend' pods
```shell
kubectl get po -l app=v2,tier!=frontend
# or
kubectl get po -l 'app in (v2), tier notin (frontend)'
# or
kubectl get po --selector=app=v2,tier!=frontend
```

### Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels
```shell
kubectl label po -l "app in(v1,v2)" tier=web
```
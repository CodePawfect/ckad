Get 'app=v2' and not 'tier=frontend' pods
```shell
kubectl get po -l app=v2,tier!=frontend
# or
kubectl get po -l 'app in (v2), tier notin (frontend)'
# or
kubectl get po --selector=app=v2,tier!=frontend
```

Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels
```shell
kubectl label po -l "app in(v1,v2)" tier=web
```

Add a new annotation owner: marketing where label is app=v2
```bash
kubectl annotate po -l "app=v2" owner=marketing
```

remove all app labels:
```bash
codepawfect@pop-os:~/ckad$ k get pods --show-labels
NAME     READY   STATUS    RESTARTS   AGE   LABELS
nginx    1/1     Running   0          20m   run=nginx
nginx1   1/1     Running   0          14m   app=v1,hero=beast,tier=web
nginx2   1/1     Running   0          14m   app=v2,tier=web
nginx3   1/1     Running   0          14m   app=v1,hero=beast,tier=web

codepawfect@pop-os:~/ckad$ kubectl label po -l app app-
pod/nginx1 unlabeled
pod/nginx2 unlabeled
pod/nginx3 unlabeled

codepawfect@pop-os:~/ckad$ k get pods --show-labels
NAME     READY   STATUS    RESTARTS   AGE   LABELS
nginx    1/1     Running   0          20m   run=nginx
nginx1   1/1     Running   0          14m   hero=beast,tier=web
nginx2   1/1     Running   0          14m   tier=web
nginx3   1/1     Running   0          14m   hero=beast,tier=web
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      command:
        - ls
        - "-l"
        - "/etc/secret-volume"
      volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
```

#flashcards/ckad
which fields are needed to create a volume mount in a pod from a secret?
?
#### Volume definition:
spec.volumes[].name
spec.volumes[].secret.secretName

### Container volume mount:
spec.containers[].volumeMounts[].name
spec.containers[].volumeMounts[].mountPath
<!--SR:!2025-10-20,3,250-->
+++
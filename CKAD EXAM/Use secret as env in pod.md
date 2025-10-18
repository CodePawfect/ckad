#flashcards/ckad 

## Single Secret Key as Env Variable

Use one key from a Secret as an environment variable:

```yaml
spec:
  containers:
  - name: my-app
    image: nginx
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

**Use case:** Individual sensitive values like passwords, API keys, tokens.

## All Secret Keys as Env Variables

Import all keys from a Secret as environment variables:

```yaml
spec:
  containers:
  - name: my-app
    image: nginx
    envFrom:
    - secretRef:
        name: db-secret
```

**Result:** Each key in `db-secret` becomes an env var with the same name.

**Use case:** When you want all secret values available as environment variables.

## Multiple Secrets

Combine multiple secrets and individual values:

```yaml
spec:
  containers:
  - name: my-app
    image: nginx
    envFrom:
    - secretRef:
        name: db-secret
    - secretRef:
        name: api-secret
    env:
    - name: CUSTOM_VAR
      value: "some-value"
    - name: ANOTHER_SECRET
      valueFrom:
        secretKeyRef:
          name: other-secret
          key: token
```

## Creating Secrets

```bash
# From literal values
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# From file
kubectl create secret generic my-secret \
  --from-file=ssh-key=/path/to/id_rsa

# From env file
kubectl create secret generic my-secret \
  --from-env-file=secret.env
```

---

## Flashcards

What are the required fields to use a single secret key as an environment variable in a pod? 
?

```yaml
env:
- name: ENV_VAR_NAME
  valueFrom:
    secretKeyRef:
      name: secret-name
      key: secret-key
```

+++

What is the field to import all keys from a secret as environment variables? 
?

```yaml
envFrom:
- secretRef:
    name: secret-name
```

+++
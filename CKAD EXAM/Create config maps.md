#flashcards/ckad
# Kubernetes ConfigMap Creation

Quick reference for creating ConfigMaps using different sources.

## From Literal Values

Create a ConfigMap with key-value pairs directly in the command:

```bash
kubectl create configmap my-config \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
```

**Use case:** Quick configs with few values, secrets management testing, or simple environment variables.

## From Environment File

Create a ConfigMap from a `.env` file where each line is `KEY=VALUE`:

```bash
kubectl create configmap my-config \
  --from-env-file=config.env
```

**Example `config.env`:**

```
DATABASE_HOST=localhost
DATABASE_PORT=5432
LOG_LEVEL=info
```

**Use case:** Environment-specific configuration, application settings.

## From File(s)

Create a ConfigMap from one or more files (the filename becomes the key):

```bash
# Single file
kubectl create configmap my-config \
  --from-file=app.properties

# Multiple files
kubectl create configmap my-config \
  --from-file=app.properties \
  --from-file=logging.conf

# Entire directory
kubectl create configmap my-config \
  --from-file=./config-dir/

# Custom key name
kubectl create configmap my-config \
  --from-file=custom-key=app.properties
```

**Use case:** Configuration files (JSON, YAML, XML), application properties, certificates.

## Viewing ConfigMaps

```bash
# List all ConfigMaps
kubectl get configmaps

# View ConfigMap details
kubectl describe configmap my-config

# View ConfigMap as YAML
kubectl get configmap my-config -o yaml
```

## Mixed Sources

You can combine multiple sources in one command:

```bash
kubectl create configmap my-config \
  --from-literal=version=1.0 \
  --from-env-file=app.env \
  --from-file=config.json
```

how to create a config map from the Key "app" and the value "backend"?
?
kubectl create cm configmapname --from-literal=app=backend
<!--SR:!2025-10-19,2,248-->
+++

how to create a config map from a file with multiple key/value pairs?
?
kubectl create cm configmapname --from-env-file=/path/to/file
<!--SR:!2025-10-19,2,248-->
+++

how to create a config map from a file where the whole file content is the value?
?
kubectl create cm configmapname --from-file=keyname=/path/to/file
<!--SR:!2025-10-18,1,230-->
+++
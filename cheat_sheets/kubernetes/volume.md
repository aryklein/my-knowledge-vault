---
tags:
  - kuberentes
  - wip
last_updated: 20240423
---

## Volume mounts and the "subPath" property

In Kubernetes, when you mount a volume, especially one sourced from a
**ConfigMap** or **Secret**, the default behavior is to mount all the content of
the **ConfigMap** or **Secret** into the specified directory. Here's how it
works step-by-step:

### Without `subPath`:
When you specify a `mountPath` for a volume from a `ConfigMap` or `Secret` but
do not use `subPath`, Kubernetes mounts the entire `ConfigMap` or `Secret` as a
directory at the `mountPath`.
Each key in the `ConfigMap` or `Secret` becomes a file within that directory.
This means if your `ConfigMap` contains multiple keys (entries), each one will
appear as a file within the directory specified by `mountPath`.

### With `subPath`:
Using `subPath` allows you to specify a single file (key from the `ConfigMap` or
`Secret`) to mount at the `mountPath`.
This is crucial when you want to place a single specific file at a specific
path, rather than mounting all `ConfigMap` or `Secret` data into a directory.
The `subPath` tells Kubernetes to isolate the mount to just the specified file,
preventing the overlay of the entire directory with all `ConfigMap` or `Secret`
data

For example:

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: pg-env
data:
  .env: |
    PGPORT=5432
    PGHOST=/var/run/postgresql
    PGUSER=postgres
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: pg-backup-verifier
spec:
  schedule: "0 0 * * *"  # This cron job runs every day
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: pg-backup-verifier
              image: my.registry.org/pg-backup-verifier:latest
              volumeMounts:
                - name: env
                  mountPath: /var/lib/postgresql/.env
                  subPath: .env
                  configmap
            - name: env
              configMap:
                name: pg-backup-verifier-config
                items:
                  - key: .env
                    path: .env
```

if you don't use `subPath: .env` it will create the directory
`/var/lib/postgresql/.env/` with all the files in there.
---
tags: postgresql, kubernetes
---

# CloudNativePG operator

CloudNativePG is the Kubernetes operator that covers the full life-cycle of a
highly available PostgreSQL database cluster with a primary/standby
architecture, using native streaming replication.

For additional information, please visit the official [website](https://cloudnative-pg.io)

## Installing the operator on Kubernetes with Helm

To install the operator on Kubernetes using Helm, you can follow these steps:

```bash
# Add the Helm repository
helm repo add cnpg https://cloudnative-pg.github.io/charts
# Update the Helm repositories
helm repo update
# install the CloudNativePG operator
helm upgrade --install cnpg \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg
```

If the release `cnpg` does not exist, the `upgrade` command in Helm will
install it.

To check if the CloudNativePG operator is running, you can use the following
command:

```bash
kubectl get pods -n cnpg-system
NAME                                READY   STATUS    RESTARTS   AGE
cnpg-cloudnative-pg-5cfb886-c6ds6   1/1     Running   0          5m41s
```

This command will list all the pods in the cnpg-system namespace. If the
CloudNativePG operator is running, you should see one or more pods related to
it in the output.

## PostgreSQL cluster deployment

To deploy a simple PostgreSQL cluster, you can apply a manifest file that
defines your desired configuration. Here's an example of how the manifest file
might look:

```yaml
# Example of PostgreSQL cluster
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: postgres-test
spec:
  instances: 2

  # Example of rolling update strategy:
  # - unsupervised: automated update of the primary once all
  #                 replicas have been upgraded (default)
  # - supervised: requires manual supervision to perform
  #               the switchover of the primary
  primaryUpdateStrategy: supervised

  # Require 1Gi of space
  storage:
    size: 1Gi
```

The following is a bit more complete example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  namespace: postgres-test
  name: postgres-test-app-user
type: kubernetes.io/basic-auth
data:
  password: VHhWZVE0bk44MlNTaVlIb3N3cU9VUlp2UURhTDRLcE5FbHNDRUVlOWJ3RHhNZDczS2NrSWVYelM1Y1U2TGlDMg==
  username: YXBw
---
apiVersion: v1
kind: Secret
metadata:
  namespace: postgres-test
  name: postgres-test-superuser
type: kubernetes.io/basic-auth
data:
  password: dU4zaTFIaDBiWWJDYzRUeVZBYWNCaG1TemdxdHpxeG1PVmpBbjBRSUNoc0pyU211OVBZMmZ3MnE4RUtLTHBaOQ==
  username: cG9zdGdyZXM=
---
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  namespace: postgres-test
  name: postgres-test
spec:
  description: "Test Postgres cluster"
  imageName: ghcr.io/cloudnative-pg/postgresql:15.2
  # imagePullSecret is only required if the images are located in a private registry
  # imagePullSecrets:
  #   - name: private_registry_access
  instances: 3
  startDelay: 300
  stopDelay: 300
  primaryUpdateStrategy: unsupervised

  postgresql:
    parameters:
      shared_buffers: 256MB
      pg_stat_statements.max: '10000'
      pg_stat_statements.track: all
      auto_explain.log_min_duration: '10s'
    pg_hba:
      - host all all 10.0.0.0/8 md5
  bootstrap:
    initdb:
      database: app
      owner: app
      secret:
        name: postgres-test-app-user
  superuserSecret:
    name: postgres-test-superuser
  storage:
    storageClass: standard
    size: 10Gi
  resources:
    requests:
      memory: "512Mi"
      cpu: "1"
    limits:
      memory: "1Gi"
      cpu: "2"
  affinity:
    enablePodAntiAffinity: true
    topologyKey: failure-domain.beta.kubernetes.io/zone
  nodeMaintenanceWindow:
    inProgress: false
    reusePVC: false
```

To apply the manifest file and deploy the PostgreSQL cluster, use the following
command:

```bash
kubectl apply -f postgres-test.yaml
```

To check the status of the deployed PostgreSQL cluster, you can execute the
following command:

```bash
kubectl get cluster
```

Documentation on how to use the operator can be found [here](https://cloudnative-pg.io/docs)

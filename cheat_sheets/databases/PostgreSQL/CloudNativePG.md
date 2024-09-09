---
tags: postgresql, kubernetes
---
# CloudNativePG operator

CloudNativePG is the Kubernetes operator that covers the full life-cycle of a
highly available PostgreSQL database cluster with a primary/standby
architecture, using native streaming replication.

For additional information, please visit the official
[website](https://cloudnative-pg.io)

## Installation

### Directly using the operator manifest

The operator can be installed like any other resource in Kubernetes, through a
YAML manifest applied via `kubectl`:

```bash
kubectl apply --server-side -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/main/releases/cnpg-1.24.0.yaml
```

### Installing the operator on Kubernetes with Helm

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

## Upgrades

Upgrading CloudNativePG operator is a two-step process:

1. upgrade the controller and the related Kubernetes resources
2. upgrade the instance manager running in every PostgreSQL pod

The first step is normally done by applying the manifest of the newer version
for plain Kubernetes installations, or using Helm.

The second step is automatically executed after having updated the controller,
by default triggering a rolling update of every deployed PostgreSQL instance to
use the new instance manager. The rolling update procedure culminates with a
switchover, which is controlled by the `primaryUpdateStrategy` option, by
default set to `unsupervised`. When set to `supervised`, users need to complete
the rolling update by manually promoting a new instance through the `cnpg`
plugin for `kubectl`.

## Operator configuration

The behavior of the operator can be customized through a `ConfigMap`/`Secret`
that is located in the same namespace of the operator deployment and with
`cnpg-controller-manager-config` as the name.

Available options can be found
[here](https://cloudnative-pg.io/documentation/1.24/operator_conf)

The ones that I found the most important are:

- `INHERITED_ANNOTATIONS`:list of annotation names that, when defined in a
  Cluster metadata, will be inherited by all the generated resources, including
  pods
- `INHERITED_LABELS`: st of label names that, when defined in a Cluster
  metadata, will be inherited by all the generated resources, including pods
- `CERTIFICATE_DURATION`: Determines the lifetime of the generated certificates
  in days. Default is 90

For example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cnpg-controller-manager-config
  namespace: cnpg-system
data:
  INHERITED_ANNOTATIONS: categories
  INHERITED_LABELS: environment, workload, app
  CERTIFICATE_DURATION: "1826"
```

### Restarting the operator to reload configs

For the change to be effective, you need to recreate the operator pods to reload
the config map:

```bash
kubectl rollout restart deployment \
    -n cnpg-system \
    cnpg-controller-manager
```

## Deploying a simple PostgreSQL cluster

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
  instances: 3

  # Require 10Gi of space
  storage:
    size: 10Gi
```


The following example is more comprehensive. For further details, please refer
to the official documentation

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pg-cluster-test-01
  namespace: cloudnativepg
spec:
  # number of nodes
  instances: 3

  # postgresql configuration
  postgresql:
    pg_hba:
      - host replication streaming_replica_cluster all md5
    parameters:
      shared_buffers: "256MB"

  # environments
  env:
    - name: FOO
      value: bar
    - name: BAR
      value: foo

  # Image catalog imageCatalogRef:
  # kubectl apply -f
  https://raw.githubusercontent.com/cloudnative-pg/postgres-containers/main/Debian/ClusterImageCatalog-bullseye.yaml
    apiGroup: postgresql.cnpg.io
    kind: ClusterImageCatalog
    name: postgresql
    major: 14

  # Affinity rules. Avoid nodes in the same k8s worker
  affinity:
    enablePodAntiAffinity: true
    topologyKey: kubernetes.io/hostname
    podAntiAffinityType: preferred

  # storage
  storage:
    size: 50Gi

  # wal file storage
  walStorage:
    size: 10Gi

  # for backups and wal archive
  backup:
    barmanObjectStore:
      destinationPath: "s3://barman/pg-cluster-test-01/"
      endpointURL: "https://us-central-1.examplestorage.com"
      s3Credentials:
        accessKeyId:
          name: obj-storage-creds
          key: ACCESS_KEY_ID
        secretAccessKey:
          name: obj-storage-creds
          key: ACCESS_SECRET_KEY
      wal:
        compression: gzip
    retentionPolicy: 7d

  # certificates configuration
  certificates:
    serverAltDNSNames:
      - pg-cluster-test-01.example.org

  # managed roles
  managed:
    roles:
      - name: ary
        ensure: present
        comment: "Ary User"
        login: true
        superuser: false
        passwordSecret:
          name: pg-user-ary
      - name: streaming_replica_cluster
        ensure: present
        comment: "Streaming Replica"
        login: true
        replication: true
        superuser: false
        passwordSecret:
          name: streaming-replica-cluster

  # node resources
  resources:
    requests:
      memory: "512Mi"
      cpu: "1"
    limits:
      memory: "1Gi"
      cpu: "2"
```

After applying this manifest, you can check with:

```bash
kubectl get clusters
```

Alternatively, you can use the `cnfg` plugin with `kubectl`.

```bash
kubectl cnpg status pg-cluster-test-01
```


## Secrets

### Managed roles

Managed role passwords are stored as Kubernetes secrets and should appear as
follows:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: pg-user-ary
  labels:
    cnpg.io/reload: "true"
type: kubernetes.io/basic-auth
data:
  username: YXJ5
  password: YXJ5
---
apiVersion: v1
kind: Secret
metadata:
  name: streaming-replica-cluster
type: kubernetes.io/basic-auth
data:
  username: c3RyZWFtaW5nX3JlcGxpY2FfY2x1c3Rlcg==
  password: YXJ5
```

### The object storage secret

The following is another example of the object storage secret."

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: obj-storage-creds
type: Opaque
data:
  ACCESS_KEY_ID: fooooobarrr==
  ACCESS_SECRET_KEY: baaaaarrrfoooo==
```

## Replica cluster

A replica cluster is a CloudNativePG Cluster resource designed to replicate data
from another PostgreSQL instance, ideally also managed by CloudNativePG.

Typically, a replica cluster is deployed in a different Kubernetes cluster
located in another region. These clusters can be configured to perform cascading
replication and may utilize object storage for replicating data from the primary
source

[[postgres_replica_cluster_01.excalidraw]]

### Bootstrapping a replica cluster

The first step is to bootstrap the replica cluster. I have tested streaming
replication via `pg_basebackup` and recovery using a Barman Cloud backup stored
in an object store.

The cluster definition, as outlined in the Excalidraw diagram shared earlier, is
as follows:

**Cluster 1 -  pg-cluster-test-01 (primary cluster)**:

```yaml
---
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pg-cluster-test-01
spec:
  postgresql:
    pg_hba:
      - host replication streaming_replica_cluster all md5
...
...
...
  backup:
    barmanObjectStore:
      destinationPath: "s3://barman/pg-cluster-test-01/"
      endpointURL: "https://us-central-1.telnyxcloudstorage.com"
      s3Credentials:
        accessKeyId:
          name: obk-storage-creds
          key: ACCESS_KEY_ID
        secretAccessKey:
          name: obj-storage-creds
          key: ACCESS_SECRET_KEY
      wal:
        compression: gzip
    retentionPolicy: 7d
  replica:
    primary: pg-cluster-test-01
    source: pg-cluster-test-02
  externalClusters:
    - name: pg-cluster-test-02
      connectionParameters:
        host: pg-cluster-test-02.example.org
        user: streaming_replica_cluster
        sslmode: disable
      password:
        name: streaming-replica-cluster
        key: password
      barmanObjectStore:
        destinationPath: "s3://barman/pg-cluster-test-02/"
        endpointURL: "https://us-central-1.examplestorage.com"
        s3Credentials:
          accessKeyId:
            name: obj-storage-creds
            key: ACCESS_KEY_ID
          secretAccessKey:
            name: obj-storage-creds
            key: ACCESS_SECRET_KEY
    - name: pg-cluster-test-01
      connectionParameters:
        host: pg-cluster-test-01.example.org
        user: streaming_replica_cluster
        sslmode: disable
      password:
        name: streaming-replica-cluster
        key: password
      barmanObjectStore:
        destinationPath: "s3://barman/pg-cluster-test-01/"
        endpointURL: "https://us-central-1.examplestorage.com"
        s3Credentials:
          accessKeyId:
            name: obj-storage-creds
            key: ACCESS_KEY_ID
          secretAccessKey:
            name: obj-storage-creds
            key: ACCESS_SECRET_KEY
```

**Cluster 2 -  pg-cluster-test-02 (replica cluster)**:

```yaml
---
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pg-cluster-test-02
spec:
  postgresql:
    pg_hba:
      - host replication streaming_replica_cluster all md5
...
...
...
  backup:
    barmanObjectStore:
      destinationPath: "s3://barman/pg-cluster-test-02/"
      endpointURL: "https://us-central-1.telnyxcloudstorage.com"
      s3Credentials:
        accessKeyId:
          name: telnyx-storage-creds
          key: ACCESS_KEY_ID
        secretAccessKey:
          name: telnyx-storage-creds
          key: ACCESS_SECRET_KEY
      wal:
        compression: gzip
    retentionPolicy: 7d
  certificates:
    serverAltDNSNames:
      - pg-cluster-test-02.query.dev.telnyx.io
  replica:
    primary: pg-cluster-test-01
    source: pg-cluster-test-01
  externalClusters:
    - name: pg-cluster-test-01
      connectionParameters:
        host: pg-cluster-test-01.example.org
        user: streaming_replica_cluster
        sslmode: disable
        dbname: postgres
      password:
        name: streaming-replica-cluster
        key: password
      barmanObjectStore:
        destinationPath: "s3://barman/pg-cluster-test-01/"
        endpointURL: "https://us-central-1.examplestorage.com"
        s3Credentials:
          accessKeyId:
            name: obj-storage-creds
            key: ACCESS_KEY_ID
          secretAccessKey:
            name: obj-storage-creds
            key: ACCESS_SECRET_KEY
    - name: pg-cluster-test-02
      connectionParameters:
        host: pg-cluster-test-02.example.org
        user: streaming_replica_cluster
        sslmode: disable
        dbname: postgres
      password:
        name: streaming-replica-cluster
        key: password
      barmanObjectStore:
        destinationPath: "s3://barman/pg-cluster-test-02/"
        endpointURL: "https://us-central-1.examplestorage.com"
        s3Credentials:
          accessKeyId:
            name: obj-storage-creds
            key: ACCESS_KEY_ID
          secretAccessKey:
            name: obj-storage-creds
            key: ACCESS_SECRET_KEY
```

Each cluster should point to it's own bucket for wal archive and the part that
manage the cluster role is `replica` statement.

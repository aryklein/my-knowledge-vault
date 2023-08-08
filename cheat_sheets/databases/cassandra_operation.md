---
tags: database, cheatsheet
---

# Cassandra basic operation cheat sheet

## Nodetool basics

Check Cassandra Status:

```sh
nodetool status
```

## CQL (Cassandra Query Language)

- **Start CQL shell**:

```sh
cqlsh
```

- **Create keyspace**

To create a keyspace with `SimpleStrategy` and replication factor 3:

```cql
CREATE KEYSPACE IF NOT EXISTS keyspace_name
  WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };
```

To create a keyspace using the `NetworkTopologyStrategy`, you'll need to specify
your data centers and the number of replicas you want in each. Here's an
example:

```cql
CREATE KEYSPACE IF NOT EXISTS keyspace_name
WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3,
  'datacenter2' : 2 };
```

__Note__: be aware that the names of the data centers ('datacenter1' and
'datacenter2' in this example) should exactly match the names of your actual
data centers as they are defined in your Cassandra configuration

- **Selecting a keyspace to use**

```cql
USE keyspace_name;
```

- **Creating a table**

```cql
CREATE TABLE table_name (
  column1 datatype PRIMARY KEY,
  column2 datatype,
  ...
);
```

- **Inserting data**

```cql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

-  **Selecting data**

```cql
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name WHERE column1=value;
```

-   **Updating data**

```cql
UPDATE table_name SET column1 = value1 WHERE column2 = value2;`
```

-   **Deleteing data**

```cql
DELETE FROM table_name WHERE column1 = value1;
```

-   **Dropping a table**

```cql
DROP TABLE table_name;
```

-   **Dropping a keyspace**

```cql
DROP KEYSPACE keyspace_name;
```

- **Creating a user**

```cql
CREATE USER IF NOT EXISTS username WITH PASSWORD 'password' NOSUPERUSER;
```

In this command, `NOSUPERUSER` specifies that the new user does not have
superuser privileges. If you want the new user to have superuser privileges,
replace `NOSUPERUSER` with `SUPERUSER`.

## Cluster HA replication setup

In a fresh Cassandra deployment, the following system keyspaces are created:

1. `system`: Contains definitions of users, credentials, permissions, and
several other internal metadata. This keyspace is only stored locally and is not
replicated.
2. `system_auth`: Contains role-based access control (RBAC) data. This keyspace
should be replicated across all nodes for high availability.
3. `system_distributed`: Stores distributed data, like hints and batch logs.
This keyspace should be replicated for high availability.
4. `system_traces`: Stores trace data when query tracing is enabled. It is not
critical for operation, but for trace data availability across the cluster, it
should be replicated.
5. `system_schema`: Contains schema definitions. This keyspace is stored locally
and is not replicated.

So, the keyspaces that you should typically consider for replication
configuration in a multi-node or multi-datacenter setup are:

```
system_auth
system_distributed
system_traces
```

Here's how to alter these keyspaces:

```cql
ALTER KEYSPACE system_auth WITH REPLICATION = {
  'class' : 'NetworkTopologyStrategy',
  'datacenter1' : 3,
  'datacenter2' : 3
};

ALTER KEYSPACE system_distributed
WITH REPLICATION = {
  'class' : 'NetworkTopologyStrategy',
  'datacenter1' : 3,  'datacenter2' : 3
};

ALTER KEYSPACE system_traces
WITH REPLICATION = {
  'class' : 'NetworkTopologyStrategy',
  'datacenter1' : 3, 'datacenter2' : 3
};
```

Remember to replace `datacenter1` and `datacenter2` with your actual data center
names, and the numbers with your actual desired replication factors.

After altering the replication factor of a keyspace, run a `repair` on the
keyspace using to ensure the changes propagate correctly.

```sh
nodetool repair <keyspace_name>
```

## Decommissioning a Cassandra Datacenter

Decommissioning a datacenter in Cassandra involves removing all nodes in the
datacenter, one by one. This process needs to be performed carefully to maintain
data consistency and avoid data loss. Here's the high-level process:

1. **Prepare the cluster**: Prior to starting the decommissioning process,
ensure all nodes in your cluster are up and running. You can use the `nodetool
status` command to check the status of the nodes.
2. **Decommission nodes**: For each node in the datacenter you want to
decommission:
   -  Use the `nodetool drain` command to flush all memtables from the node to
      SSTables on disk. This command will also stop listening for new
      connections:
   - Run `nodetool decommission`. to stream all data from the node to the
     remaining nodes in the cluster
   -  Stop Cassandra service. If it's running on containers, stop the container.
3. **Verify**: After decommissioning all nodes in the datacenter, verify the
health and status of the cluster using the `nodetool status` command. It should
show the decommissioned datacenter as being removed.
4. **Remove from keyspace definitions**: update the replication factor of all
keyspaces that referenced the decommissioned datacenter. For each keyspace, you
will need to execute:

```cql
ALTER KEYSPACE keyspace_name WITH REPLICATION = {
  'class' : 'NetworkTopologyStrategy', 'remaining_datacenter' : 3 };
```
adjust the replication factor (3 in this example) as needed.

5. **Cleanup**: after removing the datacenter from the keyspace definitions, you
should run a cleanup on **all** nodes in the remaining datacenter(s) to remove
any data that belongs to the decommissioned datacenter. Run nodetool cleanup on
each node.

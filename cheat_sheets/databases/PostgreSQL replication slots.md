---
tags:
  - postgresql
  - database
---
# PostgreSQL Replication Slots

A replication slot is a feature in PostgreSQL that ensures that the primary
server will retain the WAL logs that are needed by the replicas even when they
are disconnected from the primary.

When streaming replication is utilized between a primary and some hot or
archiving standbys, a replication slot is needed to keep the WAL files alive
even when the replica is offline or disconnected. If the standby goes down, the
primary can keep track of how much the standby lags and preserve the WAL files
it requires until the standby reconnects. The WAL files are then decoded and
played back on the duplicate.

## Monitor PostgreSQL Replication Slots

The below command displays all the replication slots that exist on the database
cluster.

```SQL
postgres=# select * from pg_replication_slots;
```

These are the columns that you will see in the pg_replication_slots view:

- **`slot_name`**: This is a unique identifier of the replication slot which can
  contain lower-case letters, underscore characters, and numbers.
- **`plugin`**: For physical slots, it will be null.
- **`slot_type`**: is a text indicating whether the slot is physical or logical.
- **`datoid`**: Physical slots have no associated databases and hence is null.
  For logical slots, it will be the OID of the database this slot is associated
  with.
- **`active`**: This is a Boolean value. It is True if the slot is currently
  active and is False if it is inactive.
- **`xmin`**: This value in a replication slot indicates the oldest unvacuumed
  transaction in the database. In the context of replication, this means that no
  data older than this `xmin` value is needed by the replica. This helps in
  managing the amount of retained WAL and avoids the bloat of unnecessary data
  on the primary server
- **`catalog_xmin`**: is the lowest (oldest) XID (transaction ID) that is still
  visible in the system catalogs (tables, indexes, etc.). It is significant
  because any data row with an XID older than `catalog_xmin` is known to be dead
  to all current and future transactions and can be marked for vacuuming.
- **`restart_lsn`** specifically refers to the LSN (Log Sequence Number) where
  the replication slot will restart reading the WAL. In replication, this is
  used to keep track of the location in the log from where changes need to be
  read and applied to keep the replica database in sync with the primary
  database.
## How to Drop PostgreSQL Replication Slots

This is the command used to delete a replication slot:

```SQL
postgres=# select pg_drop_replication_slot('foobar');
```

**Orphaned Replication Slot:** The WAL files are retained by the primary when
the replica disconnects. This also means that the pg_wal directory may run out
of space

## How to Create PostgreSQL Replication Slots

The function `pg_create_physical_replication_slot` is used to create a physical
replication slot. This command has to be run in the primary node.

```SQL
postgres=# select pg_create_physical_replication_slot('foobar');
```

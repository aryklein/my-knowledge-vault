---
tags:
  - postgresql
---

# PITR (Point In Time Recovery)

## Basics Concepts

- **WAL (Write-Ahead Logging):** PostgreSQL uses a mechanism called Write-Ahead
  Logging to ensure data integrity. Before any changes (like data updates or
  inserts) are made to the actual database tables, those changes are first
  recorded in WAL files. This way, if the database crashes or there's a power
  failure, PostgreSQL can use these WAL files to recover the changes that hadn't
  yet been written (or "flushed") to the disk.

- **Checkpoint**: A checkpoint is a specific point in time where PostgreSQL has
  safely written all the changes recorded in the WAL files up to that moment to
  the actual database files on disk. Checkpoints help reduce the amount of time
  needed for recovery in case of a crash because the system only needs to replay
  WAL records after the last checkpoint.

For example, by running pg_controldata you can see the following information:

```bash
/usr/lib/postgresql/14/bin/pg_controldata -D /var/lib/postgresql/data/

...
...
Latest checkpoint location:           2/B7000060
Latest checkpoint's REDO location:    2/B7000028
Latest checkpoint's REDO WAL file:    0000000100000002000000B7
...
...
```

- `Latest Checkpoint Location (2/B7000060)`:  This represents the exact point in
  the WAL sequence where the latest checkpoint was made. The format `2/B7000060`
  is a WAL file location identifier. Here, `2` represents the **timeline** ID
  (which is important in replication and recovery scenarios, indicating a
  version of the database history), and `B7000060` is the byte offset within the
  WAL sequence where the checkpoint occurred. It tells PostgreSQL where to start
  the recovery process from the WAL files.

- `Latest Checkpoint's REDO Location (2/B7000028)`:  The **REDO** location
  specifies the earliest point in the WAL files that must be processed to
  recover the database to a consistent state. In this case, `2/B7000028`
  indicates that to ensure the database is consistent, PostgreSQL needs to start
  replaying changes from this location forward. **It's slightly before the
  checkpoint location because some changes need to be re-applied to ensure the
  database is fully consistent.**

-  `Latest Checkpoint's REDO WAL file (0000000100000002000000B7)`: This is the
   name of the WAL file containing the REDO location. WAL filenames are
   constructed in a specific format that encodes the timeline ID, the major WAL
   segment number, and the minor WAL segment number. In
   `0000000100000002000000B7`, the file pertains to the specified REDO location.
   This file and subsequent ones contain all the necessary information to
   recover the database to a consistent and up-to-date state

## PostgreSQL settings embedded within WAL files

PostgreSQL knows about critical settings from the primary because this
information is embedded within the Write-Ahead Logging (WAL) data. When
PostgreSQL performs operations, it records changes in WAL files to ensure data
durability and consistency. For certain settings critical to replication and
recovery operations, PostgreSQL includes the value of these settings in the WAL
files. This is done to ensure that when a recovery or replication is performed,
the recovering or replica server can validate its environment against the
requirements of the primary server's workload that generated the WALs.

When you attempt to recover or start a replica, PostgreSQL processes the WAL
files. If it encounters a setting that is critical for the operation and finds
that the current server's configuration does not meet the minimum requirements
as recorded in the WAL, it will halt recovery and issue an error message like
the one you've seen. This mechanism helps prevent scenarios where the replica or
recovering server might not be able to handle the workload or process the WAL
files correctly due to insufficient resources.

If you've lost your `postgresql.conf` file and need to recover PostgreSQL server
parameters, especially when dealing with a situation like WAL file recovery
where specific settings are expected, you can use `pg_controldata`

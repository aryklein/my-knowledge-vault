---
tags:
  - postgresql
  - cheatsheet
---
# Influencing WAL Rotation

In PostgreSQL, Write-Ahead Logging (WAL) files are essential for data durability
and recovery. PostgreSQL continuously writes to WAL files, but the timing of
when a WAL file is closed and a new one is started isn't directly controlled by
a time-based setting. Instead, WAL segments are switched based on the amount of
data written or specific database events. However, you can indirectly influence
the timing of WAL segment switches through certain configuration settings and
commands

- **WAL Segment Size** the `wal_segment_size` (previously` wal_segment_bytes` in
  versions before PostgreSQL 11) is a fixed size, typically **16MB** by default.
  Once PostgreSQL writes this amount of data, it starts a new WAL segment. You
  cannot change this size without recompiling PostgreSQL from source.

- **WAL Writer Delay** (`wal_writer_delay`): This parameter specifies how often
  the WAL writer flushes WAL data to disk. It does not directly close a WAL file
  but can influence the frequency of disk writes. This parameter is set in
  milliseconds.

- **Manual WAL Rotation**: You can force PostgreSQL to switch to a new WAL
  segment with the `SELECT pg_switch_wal()` function (or `SELECT
  pg_switch_xlog()` in versions before PostgreSQL 10). This could be scheduled
  to run via `pg_cron` or an external scheduler to approximate a time-based
  rotation strategy.

- **Archive Timeout** (`archive_timeout`): This setting forces PostgreSQL to
  switch WAL files if a certain amount of time has passed since the last switch,
  even if the current WAL file isn't full. It's specified in seconds, and
  setting it to a value greater than 0 enables the feature

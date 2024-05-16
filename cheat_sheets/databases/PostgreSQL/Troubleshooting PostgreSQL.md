# Troubleshooting PostgreSQL

## Finding long running queries on PostgreSQL

Long running queries may interfere on the overall database performance and
probably they are stuck on some background process. In order to find them you
can use the following query:

```SQL
SELECT
  pid,
  now() - pg_stat_activity.query_start AS duration,
  usename,
  application_name,
  state,
  query
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';
```

You can change the interval time. For example `10 minutes`.

The first returned column is the process id, the second is duration, following
the query and state of this activity. If the state is idle you don’t need to
worry about it, but active queries may be the reason behind low performances on
the database.

## Queries that causes high CPU usage

To check the query causing high CPU usage, first identify the process ID (PID).
Then, run the following SQL sentence:

```sql
SELECT query FROM pg_stat_activity WHERE pid=<pid>;
```

To identify which processes are causing high CPU usage, you can run the
following command:

```bash
ps -eo %cpu,%mem,pid,user,args | sort -k1 -r -n | head -10
```

You can also check these queries with:

```bash
for i in `ps -eo %cpu,%mem,pid,user,args | sort -k1 -r -n | head -10 | grep
postgres: | awk '{print $3}'`; do psql -P pager=off -c "SELECT query FROM
pg_stat_activity WHERE pid=$i"; done
```

This command will display the top 10 PostgreSQL queries consuming the most CPU.

## Killing long running queries

In order to cancel these long running queries you should execute:

```SQL
SELECT pg_cancel_backend(pid);
```
The `pid` parameter is the value returned in the previous step.
`pg_cancel_backend` can take a few seconds to stop the query. If it doesn't
work, you can use:

```sql
SELECT pg_terminate_backend(pid);
```

Be careful with `pg_terminate_backend`. It is the `kill -9` in PostgreSQL. It
will terminate the entire process which can lead to a full database restart in
order to recover consistency.

## Timing a query

You can time a query by running this command before executing the query:

```sql
postgres=# \timing
Timing is on.
postgres=# SELECT ...
Time: xx.xx ms
postgres=# \timing
Timing is off.
```

## Kill session/connection per database

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity WHERE -- don't kill my own connection!
    pid <> pg_backend_pid()
    -- don't kill the connections to other databases
    AND datname = 'database_name';
```

---
tags:
  - postgresql
---
# pg_cron

`pg_cron` is a PostgreSQL extension that allows database jobs to be scheduled in
the same way as cron jobs on Linux systems. Here’s a general guide on how to do
it:

## 1. Install pg_cron
The installation steps for `pg_cron` can vary depending on your operating system
and PostgreSQL version. Here's how to do it for Debian/Ubuntu:

```bash
sudo apt-get install -y postgresql-XX-cron
```

where XX is the PostgreSQL version. For example for PostgreSQL 16,
``postgresql-16-cron

## 2. Configure pg_cron
After installation, you need to enable and configure `pg_cron`:

1)  Modify your `postgresql.conf` file to include `pg_cron` in the
`shared_preload_libraries`.

```
shared_preload_libraries = 'foo, bar, pg_cron'
```
2) Restart PostgreSQL to apply the changes

3)  Connect to your PostgreSQL database and create the extension:

```SQL
\c my_database
CREATE EXTENSION pg_cron;
```

4) Schedule a Job with pg_cron:

To schedule a daily update to a row in a table, use the `cron.schedule`
function. The syntax is:
```SQL
SELECT cron.schedule('ScheduleName', 'CronSyntax', $$UPDATE your_table SET
your_column = 'new'  WHERE condition$$
```

Replace `ScheduleName` with a descriptive name for your schedule, `CronSyntax`
with the appropriate cron syntax for your schedule (e.g., `'0 0 * * *'` for
daily at midnight), `your_table` with your table name, `your_column` with the
column you want to update, `new_value` with the new value, and `condition` with
the condition that identifies the row(s) to update.

For example, to update a row in `users` table setting `last_checked` to the
current date for the user with `id 1` every day at midnight:

```SQL
SELECT cron.schedule('daily_user_update', '0 0 * * *', $$UPDATE users SET
last_checked = CURRENT_DATE WHERE id = 1$$);
```
## 3. Monitor and Manage Scheduled Jobs

You can list all scheduled jobs and check their status with:
```SQL
\c my_database
SELECT * FROM cron.job;
```

To delete a job:
```SQL
\c my_database
SELECT cron.unschedule('ScheduleName');
```

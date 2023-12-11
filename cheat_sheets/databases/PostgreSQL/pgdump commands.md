---
tags:
  - postgresql
  - database
---
# Dump and Restore

The subsequent guide details the steps for exporting a database from one
PostgreSQL node and subsequently importing it into another.

_Important_: ensure that both nodes are network-accessible to each other.

1) In the source PostgreSQL, edit `/data/postgres/pg_hba.conf` to allow
`pg_dump` remotely from the destination PostgreSQL.

2) Reload PostgreSQL

```bash
psql -U postgres -d postgres
postgres=# SELECT pg_reload_conf();
```

3) Try to connect from the new Postgres. (it's just to check the pg_hba.conf
configuration file)

```bash
psql -h <src_postgres> -p 5432 -U <user> -d <db_name>
```

4) In the destination PostgreSQL, create the role (user) and the databases:

```SQL
CREATE ROLE <role> WITH LOGIN PASSWORD '<s3cr3t>';
CREATE DATABASE <db_to_migrate>;
```

5) In the destination PostgreSQL run:

```bash
sudo su - postgres
pg_dump -v -h <src_postgres> -p 5432 -U postgres <db_to_migrate> | psql -U
postgres <db_to_migrate>
```

6) The database is migrated. Now change the ownership of the database and tables
(in the new PostgreSQL) You can use this bash lines:

```bash
SCHEMA=public && OWNER=<role> && DB=<db_to_migrate>
for table in `psql -U postgres -d ${DB} -tc "select tablename from pg_tables
where schemaname = '${SCHEMA}';"`; \
do psql -d ${DB} -c "alter table ${SCHEMA}.${table} owner to ${OWNER};"; done
psql -U postgres -d ${DB} -c "ALTER DATABASE ${DB} OWNER TO ${OWNER};"
```

## Tip

To selectively exclude data from certain tables during a dump, use the following
flags:
```
--exclude-table-data table1 --exclude-table-data table2
```


If you wish to dump only the data from specific tables, you can achieve this
with:
```
--data-only -t table1 -t table2
```

For example:
```bash
pg_dump -v -h <src_postgres> -p 5432 --exclude-table-data table1
--exclude-table-data table2 -U postgres <db_to_migrate> | psql -U postgres
<db_to_migrate>
```

This command will export the schema of all tables but will exclude the data from
`table1` and `table2`. This approach is particularly useful during migrations
when you need to replicate the table structure (schema) without transferring the
actual data.

Finally the data can be migrated with:
```bash
pg_dump -v -h <src_postgres> -p 5432 --data-only -t table1 -t table2 -U postgres
<db_to_migrate> | psql -U postgres <db_to_migrate>
```

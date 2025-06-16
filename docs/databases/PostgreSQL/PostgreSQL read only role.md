# PostgreSQL Read-Only Role Setup

This guide shows how to create a read-only PostgreSQL role with proper permissions.

## Prerequisites
- Superuser access (e.g., `postgres` user)
- Database owner privileges for the target database

## Step-by-Step Process

### 1. Create the Read-Only Role
Connect as a superuser and create the role:
```sql
CREATE ROLE <ro_user> WITH LOGIN ENCRYPTED PASSWORD 'strong_password';
```

### 2. Configure Database Access
Connect as the database owner and execute the following commands:

#### Revoke Default Permissions
```sql
REVOKE ALL ON DATABASE <database> FROM <ro_user>;
REVOKE ALL ON SCHEMA public FROM <ro_user>;
```

#### Grant Necessary Permissions
```sql
-- Allow connection to database
GRANT CONNECT ON DATABASE <database> TO <ro_user>;

-- Grant schema usage
GRANT USAGE ON SCHEMA public TO <ro_user>;

-- Grant SELECT on existing tables and sequences
GRANT SELECT ON ALL TABLES IN SCHEMA public TO <ro_user>;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO <ro_user>;
```

#### Set Default Privileges for Future Objects
```sql
-- Ensure future tables are readable
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO <ro_user>;

-- Ensure future sequences are readable
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON SEQUENCES TO <ro_user>;
```

## Verification
Test the role by connecting and attempting read operations:
```sql
-- Connect as the read-only user
\c <database> <ro_user>

-- Test read access
SELECT * FROM some_table LIMIT 1;

-- Verify write operations fail
INSERT INTO some_table VALUES (...); -- Should fail
```

## Notes
- Replace `<ro_user>` with your desired username
- Replace `<database>` with your target database name
- Use strong passwords for production environments
- Consider using multiple schemas if your database structure requires it
# Yugabyte Administration Guide

## Overview

YugabyteDB is a distributed SQL database that provides high availability 
and horizontal scalability. This guide covers essential administrative 
tasks for managing YugabyteDB clusters.

## Cluster Monitoring

### List All Nodes

Query all nodes in the cluster to check their status:

```sql
SELECT * FROM yb_servers();
```

### Web Interface

Access the cluster dashboard via the master node web UI:

```
http://<master_node_ip>:7000/
```

**Default Ports:**
- Master nodes: 7000 (HTTP), 7100 (RPC)
- Tablet servers: 9000 (HTTP), 9100 (RPC)

## Master Node Management

### Add a New Master Node

1. **Add the server to the master configuration:**
   ```bash
   yb-admin --master_addresses <m1:7100,m2:7100,m3:7100> \
     change_master_config ADD_SERVER <new_master_ip:7100>
   ```

2. **Verify the new master joined successfully:**
   ```bash
   yb-admin --master_addresses <m1:7100,m2:7100,m3:7100> \
     list_all_masters
   ```

### Remove a Master Node

**⚠️ Warning:** Ensure you maintain an odd number of masters 
(typically 3 or 5) for proper quorum.

```bash
yb-admin --master_addresses <m1:7100,m2:7100,m3:7100> \
  change_master_config REMOVE_SERVER <master_to_remove_ip:7100>
```

## Best Practices

### Master Node Configuration

- Always maintain an odd number of masters (3, 5, or 7)
- Distribute masters across different availability zones
- Monitor master node health regularly

### Command Syntax Notes

- Replace `<m1:7100,m2:7100,m3:7100>` with actual comma-separated 
  master node addresses (m1, m2, m3 represent the 3 master nodes)
- Include port 7100 for RPC communication
- Use `--master_addresses` flag (with double dashes) for proper 
  command syntax

### Master Address Requirements

You need to pass **at least a majority of the living masters** in the 
`--master_addresses` list:

- **3-master cluster:** Minimum 2 masters (recommended: all 3)
- **5-master cluster:** Minimum 3 masters (recommended: all 5)

**Why:** YugabyteDB uses the provided list to discover and connect to 
the master quorum. Listing all masters ensures commands succeed even if 
some masters are temporarily unavailable.

## Troubleshooting

### Common Issues

- **Connection failures:** Verify master addresses and network 
  connectivity
- **Quorum loss:** Ensure majority of masters are healthy
- **Port conflicts:** Check that ports 7000, 7100, 9000, 9100 are 
  available

### Verification Commands

```bash
# Check cluster status
yb-admin --master_addresses <m1:7100,m2:7100,m3:7100> list_all_masters

# View tablet server status
yb-admin --master_addresses <m1:7100,m2:7100,m3:7100> \
  list_all_tablet_servers
```

## Database and Tablespace Example

### Create Database

```sql
CREATE DATABASE geodb;
\c geodb
```

### Create Regional Tablespaces

```sql
-- EU tablespace
CREATE TABLESPACE ts_eu WITH (replica_placement='{
  "num_replicas": 3,
  "placement_blocks": [
    {"cloud":"gce","region":"eu-central","zone":"fr5",
     "min_num_replicas":3, "leader_preference":1}
  ]
}');

-- US tablespace
CREATE TABLESPACE ts_us WITH (replica_placement='{
  "num_replicas": 3,
  "placement_blocks": [
    {"cloud":"telnyx","region":"us-east","zone":"fl1",
     "min_num_replicas":3, "leader_preference":1}
  ]
}');

-- SY tablespace
CREATE TABLESPACE ts_sy WITH (replica_placement='{
  "num_replicas": 3,
  "placement_blocks": [
    {"cloud":"aws","region":"ap-southeast","zone":"sy1",
     "min_num_replicas":3, "leader_preference":1}
  ]
}');
```

### Create Geo-Partitioned Table

```sql
CREATE TABLE transactions (
    user_id   INTEGER NOT NULL,
    account_id INTEGER NOT NULL,
    geo_partition VARCHAR,
    account_type VARCHAR NOT NULL,
    amount NUMERIC NOT NULL,
    txn_type VARCHAR NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
) PARTITION BY LIST (geo_partition);

CREATE TABLE transactions_us
    PARTITION OF transactions
      (user_id, account_id, geo_partition, account_type,
       amount, txn_type, created_at,
       PRIMARY KEY (user_id HASH, account_id, geo_partition))
FOR VALUES IN ('US') TABLESPACE ts_us;

CREATE TABLE transactions_sy
    PARTITION OF transactions
      (user_id, account_id, geo_partition, account_type,
       amount, txn_type, created_at,
       PRIMARY KEY (user_id HASH, account_id, geo_partition))
    FOR VALUES IN ('SY') TABLESPACE ts_sy;

CREATE TABLE transactions_eu
    PARTITION OF transactions
      (user_id, account_id, geo_partition, account_type,
       amount, txn_type, created_at,
       PRIMARY KEY (user_id HASH, account_id, geo_partition))
    FOR VALUES IN ('EU') TABLESPACE ts_eu;
```

### Insert Sample Data

```sql
INSERT INTO transactions
    VALUES (100, 10001, 'EU', 'checking', 120.50, 'debit');

INSERT INTO transactions
    VALUES (101, 10002, 'US', 'checking', 121.70, 'debit');

INSERT INTO transactions
    VALUES (102, 10003, 'SY', 'checking', 125.10, 'debit');
```
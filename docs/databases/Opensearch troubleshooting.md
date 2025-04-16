# Opensearch Troubleshooting Guide

This document provides essential troubleshooting commands for Opensearch clusters.

## Cluster Health and Status

### Check Cluster Health
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_cluster/health?pretty' | jq
```

The health status will be one of:
- `green`: All primary and replica shards are allocated
- `yellow`: All primary shards are allocated, but some replica shards are not
- `red`: At least one primary shard is not allocated

### Check Cluster Settings
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_cluster/settings?pretty' | jq
```

### Check Nodes Status
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_cat/nodes?v&h=id,name,heap.percent,ram.percent,cpu,load_1m,disk.total,disk.used,disk.avail,disk.percent' | sort -k1
```

## Shard Management

### Identify Unassigned Shards
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_cat/shards?format=json&h=index,shard,prirep,state,unassigned.reason,node' | jq '[.[] | select(.state == "UNASSIGNED")]'
```

### Understanding Allocation Failures

Check why certain shards cannot be allocated:
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_cluster/allocation/explain?pretty'
```

For a specific shard:
```bash
curl -X GET "https://user:password@example-opensearch.example.org:9200/_cluster/allocation/explain" -H 'Content-Type: application/json' -d '{
  "index": "logs-2025.03.29",
  "shard": 0,
  "primary": true
}'
```

Common reasons for unassigned shards:

- `ALLOCATION_FAILED`: Could not allocate due to disk space, node filtering, etc.
- `NODE_LEFT`: A node containing the shard left the cluster
- `CLUSTER_RECOVERED`: Shards recovering after cluster restart
- `INDEX_CREATED`: New index shards being allocated
- `THROTTLE:` Shard allocation is temporarily paused to avoid overloading nodes. This usually happens when a node reaches the limit of concurrent shard recoveries (either incoming or outgoing). This is managed by the following cluster settings: 
	- `cluster.routing.allocation.node_concurrent_outgoing_recoveries`
	- `node_concurrent_incoming_recoveries`, or `node_concurrent_recoveries`

## Resolving Allocation Issues

### Basic Reallocation (Automatic Attempt)

If the cluster is not under pressure and allocation was temporarily blocked (e.g., a node went down and came back), OpenSearch should retry on its own.

To manually trigger the reroute process:
```bash
curl -X POST "https://user:password@example-opensearch.example.org:9200/_cluster/reroute?retry_failed=true"
```

### Force Allocation of Specific Shard
```bash
curl -X POST "https://user:password@example-opensearch.example.org:9200/_cluster/reroute" -H 'Content-Type: application/json' -d '{
  "commands": [
    {
      "allocate_stale_primary": {
        "index": "logs-2025.03.29",
        "shard": 0,
        "node": "node-name",
        "accept_data_loss": true
      }
    }
  ]
}'
```

⚠️ Use `accept_data_loss: true` with extreme caution as it can result in data loss!

## Index Management

### List All Indices with Basic Info
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_cat/indices?v&h=health,status,index,pri,rep,docs.count,store.size' | sort
```

### Check Index Settings
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/index-name/_settings?pretty'
```

### Close and Open an Index
```bash
# Close index (makes it read-only and unallocated)
curl -X POST "https://user:password@example-opensearch.example.org:9200/index-name/_close"

# Open index
curl -X POST "https://user:password@example-opensearch.example.org:9200/index-name/_open"
```

## Performance Troubleshooting

### Check Thread Pool Stats
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_nodes/stats/thread_pool?pretty'
```

### Hot Threads Analysis
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_nodes/hot_threads?pretty'
```

### Memory Pressure
```bash
curl -X GET -k 'https://user:password@example-opensearch.example.org:9200/_nodes/stats/jvm?pretty' | jq '.nodes[] | .jvm.mem'
```

## Security Notes

- The examples use `-k` to ignore SSL certificate validation - only use in test environments
- Replace `user:password@example-opensearch.example.org:9200` with your actual credentials and endpoint
- Consider using environment variables or secure credential management for production environments
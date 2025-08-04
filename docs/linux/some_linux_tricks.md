---
tags: linux, troubleshooting, system-administration
---
# Linux Tricks and Workarounds

A collection of useful Linux tricks, workarounds, and troubleshooting
techniques.

## Memory Management

### How to Consume Memory for Testing

```bash
dd if=/dev/urandom of=/dev/shm/bloque1 bs=1M count=14000
```

This command fills system memory with 14 GiB of random data by writing to
`/dev/shm` (shared memory filesystem).

**⚠️ Warning**: This will consume significant system memory and may cause system
instability. Use with caution and monitor system resources.

**Cleanup**: Remove the file when done:
```bash
rm /dev/shm/bloque1
```

## File System Operations

### Quick File Operations

**Create large sparse files**:
```bash
# Create a 1GB sparse file (doesn't actually use disk space until written to)
truncate -s 1G largefile.txt

# Create a file with actual data
dd if=/dev/zero of=largefile.txt bs=1M count=1024
```

*Last updated: [[daily-notes/2025-08-04]]*

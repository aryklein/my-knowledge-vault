---
tags:
  - linux
  - partition
  - filesystem
  - disk
---
# Disk and filesystem management on Linux

## Convert a disk to GPT

Non-interactive (with `parted`)

```bash
parted /dev/sdx mklabel gpt
```

With `fdisk`

```bash
fdisk /dev/sdx
```

```
Welcome to fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): g
Command (m for help): w
```

With `fdisk` (non-interactive):

```bash
fdisk /dev/nvme1n1 <<EOF
g
w
EOF
```

## Wiping a filesystem signature

To wipe a filesystem signature from a disk or partition, you can use the
`wipefs` command. This tool can erase filesystem, RAID, and partition-table
signatures to make the space available for a new filesystem.

List all available signatures:

```bash
sudo wipefs -n /dev/sdx
```

Wipe all signatures on the **device**:

```bash
sudo wipefs -a /dev/sdx
```

Wipe all signatures on the **partition**:

```bash
sudo wipefs -a /dev/sdx1
```

## How to extend the last partition and filesystem

`growpart` is one of the utility to extend the **last** partition of the disk
to fill the available free space on the disk. It changes the sector position to
the end sector of the disk.

I usually use this method to add more disk space in a cloud instance (VM)

**_NOTE:_**
-  It only extends the last partition, without creating or deleting any existing
partition.
-  It can be executed online

```bash
growpart /dev/sdb 1
```

First argument is block device and the second argument is partition number. In
this example `growpart` will extend the partition `1` for the block device
`/dev/sdb`.

Now it's time to extend the filesystem. The command will change depeding on the
filesystem type:

```bash
# ext4
resize2fs /dev/sdb1
# xfs
xfs_growfs -d /mount-point
```

The option `-d` in `xfs_growfs` is to increase the size of the file system to
the maximum size that the underlying device supports. Otherwise you should use
`-D <size>`
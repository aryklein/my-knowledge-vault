---
tags: linux
---

# How to extend the last partition and filesystem

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
the maximum size that the underlying device supports. Othwerwise you should use
`-D <size>`

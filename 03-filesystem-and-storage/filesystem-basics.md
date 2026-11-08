# Filesystem basics

A filesystem organizes data on a storage device so Linux can store and retrieve files. On a server, understanding the relationship between disks, partitions, filesystems, and mount points makes storage changes much easier to troubleshoot.

## Overview

These terms describe different layers:

| Term | Meaning |
|---|---|
| Block device | Storage device exposed by the kernel, such as `/dev/sda` or `/dev/nvme0n1`. |
| Partition | A region of a disk described by its partition table, such as `/dev/sda1`. |
| Physical volume (PV) | A disk or partition prepared for LVM. |
| Volume group (VG) | A pool of storage made from one or more LVM physical volumes. |
| Logical volume (LV) | A virtual block device allocated from a volume group. |
| Filesystem | The structure used to store files and directories, such as ext4 or XFS. |
| Mount point | A directory where a filesystem is attached to the Linux directory tree. |
| UUID | A filesystem identifier commonly used to refer to a filesystem independently of device naming. |

A common non-LVM layout is:

```text
Disk (/dev/sdb)
└── Partition (/dev/sdb1)
    └── Filesystem (ext4)
        └── Mount point (/srv/data)
```

An LVM layout adds a storage-management layer:

```text
Disk or partition
└── Physical Volume (PV)
    └── Volume Group (VG)
        └── Logical Volume (LV)
            └── Filesystem
                └── Mount point
```

## Useful commands

### `lsblk`

List block devices and their relationships.

```bash
lsblk
lsblk -f
lsblk -o NAME,SIZE,TYPE,FSTYPE,UUID,MOUNTPOINTS
```

| Option | Meaning |
|---|---|
| `-f` | Show filesystem information, labels, UUIDs, and mount points. |
| `-o` | Select the output columns. |
| `-p` | Show full device paths. |
| `-a` | Include empty devices and RAM disks. |

### `findmnt`

Show mounted filesystems and their mount points.

```bash
findmnt
findmnt /
findmnt --target /var
```

| Option | Meaning |
|---|---|
| `--target <path>` | Show the filesystem containing a path. |
| `--source <device>` | Search by device or filesystem source. |
| `--fstab` | Show entries from `/etc/fstab`, rather than only the current mount table. |

### `blkid`

Display filesystem type, UUID, and label information.

```bash
sudo blkid
sudo blkid /dev/sdb1
```

Use `lsblk -f` for a convenient overview. `blkid` is useful when you need the exact UUID or filesystem metadata.

## Important distinctions

- A disk can contain multiple partitions, and a filesystem can also be created directly on a whole disk.
- A partition does not automatically contain a filesystem.
- LVM logical volumes are block devices; they still need a filesystem before they can hold ordinary files.
- A filesystem is not available at a directory until it is mounted.
- Device names such as `/dev/sdb` can change between boots or when hardware changes. UUIDs are generally more stable for `/etc/fstab` entries.

# Filesystems

A filesystem defines how files and directories are stored on a block device. Choose a filesystem based on workload, operational needs, and recovery requirements.

## Common filesystem tools

| Tool | Purpose |
|---|---|
| `mkfs.ext4` | Create an ext4 filesystem. |
| `e2fsck` | Check and repair ext2/ext3/ext4 filesystems. |
| `resize2fs` | Grow or shrink ext2/ext3/ext4 filesystems. |
| `tune2fs` | Display or change ext filesystem parameters. |
| `mkfs.xfs` | Create an XFS filesystem (from `xfsprogs`). |
| `xfs_repair` | Repair an unmounted XFS filesystem (from `xfsprogs`). |
| `xfs_growfs` | Grow a mounted XFS filesystem (from `xfsprogs`). |

## Create an ext4 filesystem

**Only use this on a new or intentionally erased partition or logical volume.** Creating a filesystem replaces existing filesystem metadata.

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkfs -t <TYPE> /dev/sdb1
sudo mkfs.ext4 -L data /dev/sdb1
```

| Option | Meaning |
|---|---|
| `-L <label>` | Set a filesystem label. |
| `-m <percent>` | Set the percentage of reserved blocks (ext filesystems). |

Before running `mkfs`, confirm the target with `lsblk -f`, `blkid`, and `findmnt`. Do not run it against a mounted device or a device containing data you need.

## Inspect an ext filesystem

```bash
sudo tune2fs -l /dev/sdb1
sudo dumpe2fs -h /dev/sdb1
```

`-l` displays filesystem parameters. `dumpe2fs -h` displays the superblock summary.

## Check an ext filesystem

`e2fsck` checks ext2/ext3/ext4 filesystems. It is generally unsafe to run a repair check on a mounted filesystem.

```bash
sudo umount /dev/sdb1
sudo e2fsck -f /dev/sdb1
```

| Option | Meaning |
|---|---|
| `-f` | Force a check even when the filesystem appears clean. |
| `-p` | Automatically repair problems that can be safely fixed without operator intervention. |
| `-n` | Open read-only and answer no to repair prompts. |

For a filesystem used by the running system, use an appropriate rescue environment or maintenance procedure rather than trying to unmount the root filesystem.

## Resize ext4

After the underlying block device or logical volume has been enlarged, grow the filesystem to use the available space:

```bash
sudo resize2fs /dev/vg_data/lv_data
```

Without a size argument, `resize2fs` grows the filesystem to fill the available device space. The filesystem and block device must be compatible with the requested size.

Shrinking is more sensitive: the filesystem must be shrunk before the underlying logical volume or partition. Back up first and follow the filesystem-specific procedure. Never reduce a block device first and hope the filesystem will fit afterward.

## XFS notes

XFS is designed to grow while mounted; it cannot be shrunk with the standard XFS tools.

```bash
sudo xfs_info /srv/data
sudo xfs_growfs /srv/data
```

For repair, the XFS filesystem must be unmounted:

```bash
sudo umount /dev/sdb1
sudo xfs_repair /dev/sdb1
```

Install XFS tools when needed:

```bash
sudo apt install xfsprogs
```


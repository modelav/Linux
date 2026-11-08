# Mounting and unmounting filesystems

Mounting attaches a filesystem to a directory in the Linux filesystem tree. Unmounting detaches it cleanly.

## Inspect current mounts

```bash
findmnt
findmnt /srv/data
mount
```

`findmnt` is usually the clearest way to inspect the relationship between a device, filesystem, and mount point.

## Mount a filesystem temporarily

Create the mount point, then mount the device:

```bash
sudo mkdir -p /srv/data
sudo mount /dev/sdb1 /srv/data
findmnt /srv/data
```

Specify the filesystem type or mount options when needed:

```bash
sudo mount -t ext4 /dev/sdb1 /srv/data
sudo mount -o ro /dev/sdb1 /mnt/inspection
```

| Option | Meaning |
|---|---|
| `-t <type>` | Specify the filesystem type. |
| `-o <options>` | Set comma-separated mount options. |
| `-r` | Mount read-only (equivalent to `-o ro`). |
| `-v` | Show more detail about the mount operation. |

A manual mount normally lasts until it is unmounted or the system is rebooted. Use `/etc/fstab` for persistent mounts.

## Unmount a filesystem

```bash
sudo umount /srv/data
```

You can use the mount point or device name. Prefer the mount point when it is unambiguous.

If a filesystem is busy, identify processes using it:

```bash
sudo fuser -vm /srv/data
sudo lsof +f -- /srv/data
```

Stop or move the processes using the filesystem, leave the directory in shells, and retry `umount`.

`umount -l` (lazy unmount) detaches the filesystem from the directory tree immediately and completes cleanup when it is no longer busy. It is not a substitute for finding and stopping processes that are still using the filesystem.

## Bind mounts

A bind mount makes an existing directory available at another path:

```bash
sudo mkdir -p /srv/shared
sudo mount --bind /var/lib/example/shared /srv/shared
findmnt /srv/shared
```

A bind mount does not copy data; both paths refer to the same underlying files.

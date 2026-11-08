# Linux filesystem hierarchy

Linux organizes files and directories in one tree rooted at `/`. Disks, partitions, network filesystems, and virtual filesystems are attached to that tree at mount points.

## Important directories

| Directory | Purpose |
|---|---|
| `/` | Root of the filesystem tree. |
| `/boot` | Kernel images, initramfs files and bootloader data on many installations. |
| `/dev` | Device nodes and device-related interfaces. |
| `/etc` | Host-specific system and service configuration. |
| `/home` | Regular users' home directories. |
| `/media` | Common location for removable-media mounts. |
| `/mnt` | Conventional location for temporary/manual mounts. |
| `/opt` | Add-on application software. |
| `/proc` | Virtual filesystem exposing process and kernel information. |
| `/root` | Root user's home directory. |
| `/run` | Volatile runtime data created since boot, such as sockets and PID files. |
| `/srv` | Data served by services on the system. |
| `/sys` | Virtual filesystem exposing kernel and device information. |
| `/tmp` | Temporary files; cleanup policy varies. |
| `/usr` | Most distribution-managed programs, libraries and shared data. |
| `/var` | Variable data such as logs, caches, queues and application state. |

The table describes common purposes. It does not mean each directory is a separate filesystem or that every server uses every path.

## Useful subdirectories

### `/etc`

| Path | Purpose |
|---|---|
| `/etc/hostname` | System hostname. |
| `/etc/hosts` | Static hostname-to-address mappings. |
| `/etc/fstab` | Filesystem mount definitions. |
| `/etc/passwd` | User account records. |
| `/etc/shadow` | Password hashes and password-aging data. |
| `/etc/ssh/` | OpenSSH client and server configuration. |
| `/etc/systemd/` | Local systemd configuration and unit overrides. |
| `/etc/apt/` | APT configuration and repository sources. |

### `/var`

| Path | Purpose |
|---|---|
| `/var/log` | System and application logs. |
| `/var/lib` | Persistent application and package state. |
| `/var/cache` | Re-creatable cached data. |
| `/var/spool` | Queued work, such as print or mail queues. |
| `/var/tmp` | Temporary files that commonly persist longer than files in `/tmp`. |

Do not delete files from `/var/lib` or `/var/spool` simply to free disk space. They may contain application state or queued work.

### `/usr`

| Path | Purpose |
|---|---|
| `/usr/bin` | General user commands and programs. |
| `/usr/sbin` | System administration programs and other system binaries. |
| `/usr/lib` | Libraries and package-specific program files. |
| `/usr/share` | Architecture-independent data and documentation. |
| `/usr/local/bin` | Locally installed commands. |
| `/usr/local/sbin` | Locally installed administration commands. |

On Debian's merged-`/usr` layout, `/bin`, `/sbin` and `/lib` may be symbolic links into `/usr`.

## Virtual and runtime filesystems

`/proc` and `/sys` are kernel-provided interfaces, not ordinary directories of persistent files. `/run` is temporary runtime storage and is normally cleared at boot. `/dev` contains device nodes managed by the system.

```bash
mount | grep -E ' on /(proc|sys|run|dev) '
findmnt /proc /sys /run /dev
```

## File types

The first character in `ls -l` indicates the file type:

| Character | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `p` | FIFO (named pipe) |
| `s` | Unix-domain socket |

```bash
ls -l /etc/hostname /bin/sh /dev/null
file /etc/hostname
stat /etc/hostname
```

## Mount points

A mount point is a directory where another filesystem is attached to the tree. `/home` may be on a separate filesystem even though applications see it as a normal directory.

```bash
findmnt
findmnt /
findmnt /var
```

Use `findmnt` to see which filesystem contains a path. For disk, partition and mount operations, see the storage guides.

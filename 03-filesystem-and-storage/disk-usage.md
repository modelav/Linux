# Disk usage

Use these commands to find out how much space is available, identify large directories, and investigate filesystems that appear full.

## Check filesystem space with `df`

`df` reports used and available space on mounted filesystems.

```bash
df -h
df -hT
df -ih
```

| Option | Meaning |
|---|---|
| `-h` | Human-readable sizes. |
| `-T` | Include filesystem type. |
| `-i` | Show inode usage instead of block usage. |
| `-x <type>` | Exclude a filesystem type. |

A filesystem can run out of inodes even when it still has free space in bytes. Use `df -i` to check inode availability.

## Find large directories with `du`

`du` estimates space used by files and directories.

```bash
du -sh /var/log
du -h -d 1 /var
du -xhd1 /var
```

| Option | Meaning |
|---|---|
| `-s` | Show only a total for each argument. |
| `-h` | Human-readable sizes. |
| `-d <n>` | Limit directory traversal to depth `<n>`. |
| `-x` | Stay on the same filesystem; do not cross mount points. |

Sort directory sizes:

```bash
sudo du -xhd1 /var | sort -h
```

## Find large files

```bash
sudo find /var -xdev -type f -size +1G -print
```

`-xdev` keeps the search on the same filesystem, and `-size +1G` matches files larger than 1 GiB. Review results before deleting or moving anything.

## Check for deleted files still using space

A process can keep a deleted file open. The file no longer appears in its directory, but its disk space is not released until the process closes it.

If `lsof` is installed:

```bash
sudo lsof +L1
```

Identify the owning process and restart or signal it using the application's normal operational procedure.

## Interactive usage viewer

`ncdu` provides an interactive directory-usage browser.

```bash
sudo apt install ncdu
sudo ncdu -x /var
```

Use the `-x` option to stay on one filesystem. Review entries carefully; interactive deletion features can remove data.

## `df` and `du` show different things

- `df` reports filesystem-level space usage.
- `du` walks directory entries and estimates space used by files it can see.
- A large difference can be caused by deleted-but-open files, reserved filesystem blocks, snapshots, hidden data under mount points, or permissions that prevent `du` from reading some directories.

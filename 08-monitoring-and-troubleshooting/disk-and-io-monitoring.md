# Disk and I/O monitoring

Use these commands to distinguish a full filesystem from slow storage, high I/O activity, or a device-health problem.

## Filesystem space and inodes

### `df`

```bash
df -hT
df -i
```

| Option | Meaning |
|---|---|
| `-h` | Human-readable sizes. |
| `-T` | Show filesystem type. |
| `-i` | Show inode usage instead of block usage. |

A filesystem can run out of space or inodes. Check both reports.

### `du`

Show which directories use the most space:

```bash
sudo du -xhd1 /var | sort -h
sudo du -xhd1 /home | sort -h
```

| Option | Meaning |
|---|---|
| `-x` | Stay on one filesystem. |
| `-h` | Human-readable sizes. |
| `-d 1` | Show totals one directory level below the starting path. |

Find individual files larger than 1 GiB:

```bash
sudo find /var -xdev -type f -size +1G -printf '%s %p\n' | sort -nr | head
```


## Disk I/O activity

### `iostat`

Provided by the `sysstat` package:

```bash
iostat -xzy 1 5
```

| Option | Meaning |
|---|---|
| `-x` | Show extended device statistics. |
| `-z` | Omit devices with no activity during the sample period. |
| `-d` | Show device utilization report. |
| `-y` | Omit the first report, which otherwise includes averages since boot. |

Useful fields include read/write rates, `await` (average I/O request wait time), `aqu-sz` (average queue length), and `%util` (time the device had I/O in progress). Interpret these in context: `%util` alone is not a reliable saturation measure for every SSD, NVMe, RAID, or multi-queue device.

### `iotop`

Show processes performing disk I/O:

```bash
sudo iotop
```

Availability depends on kernel accounting support. Some systems require elevated privileges to display full process I/O information.

## Deleted files still using disk space

A process can keep a deleted file open. The directory entry is gone, but the space is not released until the process closes the file descriptor.

With `lsof` installed:

```bash
sudo lsof +L1
```

Identify the process and its service before restarting anything.

## Disk health

### `smartctl`

Provided by `smartmontools`:

```bash
sudo smartctl -H /dev/sdX
sudo smartctl -a /dev/sdX
```

| Option | Meaning |
|---|---|
| `-H` | Show the device's SMART overall-health self-assessment. |
| `-a` | Show available SMART and device information. |
| `-t short` | Start a short self-test on supported devices. |
| `-l selftest` | Show the self-test log. |

Replace `/dev/sdX` with the correct device. For NVMe or hardware RAID, the device path and required options may differ. A passing SMART result does not guarantee that a disk will not fail.

## TRIM for SSD storage

Check whether periodic TRIM is enabled:

```bash
systemctl status fstrim.timer
```

`fstrim` informs supported storage devices about blocks that are no longer in use. On systems where the timer is configured, it normally handles periodic trimming. Do not run frequent manual TRIM jobs without a reason.

## Common checks when a disk is full

```bash
df -hT
df -i
sudo du -xhd1 /var | sort -h
sudo lsof +L1
```

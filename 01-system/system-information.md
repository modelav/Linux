# System information and identification

## 1. Identify the OS release

### `/etc/os-release`

This file contains operating-system identification fields in a shell-compatible format. It is provided by the operating system and is the preferred general-purpose source for scripts that need distribution identification.

```bash
cat /etc/os-release
```

Important fields:

| Field | Meaning |
|---|---|
| `PRETTY_NAME` | Human-readable operating-system name. |
| `NAME` | Distribution name. |
| `VERSION` | Human-readable release version. |
| `VERSION_ID` | Machine-readable release identifier. |
| `VERSION_CODENAME` |  release codename when provided. |
| `ID` | Distribution identifier. |


### `hostnamectl`

**Purpose:** Display or manage the system hostname and related system identity information.  

```bash
hostnamectl
hostnamectl status
```

Useful fields commonly include the static hostname, machine ID, boot ID, operating-system information, kernel, and architecture. Availability depends on systemd and its host-named service.

## 2. Kernel and machine architecture

### `uname`

**Purpose:** Print kernel and machine information.  

| Option | Meaning |
|---|---|
| `-a` | Print all available information. |
| `-r` | Print the kernel release. |
| `-v` | Print the kernel version/build string. |
| `-s` | Print the kernel name. |
| `-m` | Print the machine hardware name. |
| `-p` | Print the processor type, if known; may return `unknown`. |
| `-n` | Print the network node hostname. |


### Architecture and CPU details

```bash
getconf LONG_BIT
lscpu
nproc
```

| Command / option | Meaning |
|---|---|
| `getconf LONG_BIT` | Reports the userspace `long` integer width (commonly 64 or 32). |
| `lscpu` | Displays CPU architecture, topology, cores, threads, and related details. Supplied by the `util-linux` package on Debian. |
| `nproc` | Prints the processing units available to the current process; this can differ from the machine's total CPU count. |
| `lscpu -e` | Displays an extended per-CPU table. |

## 3. Uptime, time, and load

### `uptime`

**Purpose:** Show current time, uptime, logged-in users, and 1-, 5-, and 15-minute load averages.  

```bash
uptime
uptime -p
uptime -s
```

| Option | Meaning |
|---|---|
| `-p` | Show uptime in a human-readable format. |
| `-s` | Show the date and time the system started. |

Load average is not CPU percentage. It reflects the average number of tasks runnable or waiting in uninterruptible kernel sleep over the stated periods. Interpret it with CPU count, workload, I/O latency, and historical baseline.

### `timedatectl`

**Purpose:** Inspect system clock, time zone, and time-synchronization status.  

```bash
timedatectl status
timedatectl list-timezones
timedatectl set-timezone <COUNTRY>/<CITY>
```

### `date`

```bash
date
date -u
date --iso-8601=seconds
```

| Option | Meaning |
|---|---|
| `-u` | Display time in UTC. |
| `--iso-8601=seconds` | Use an ISO 8601-style timestamp with seconds. |

## 4. Memory, CPU, and block-device overview

These commands provide a first look. Dedicated monitoring and storage guides will cover interpretation in more depth.

```bash
free -h
lscpu
lsblk
```

### `free`

**Purpose:** Report physical memory and swap usage.  

| Option | Meaning |
|---|---|
| `-h` | Human-readable units. |
| `-m` | Display values in mebibytes. |
| `-g` | Display values in gibibytes. |
| `-w` | Wide output with separate buffers and cache columns. |

```bash
free -h
free -wh
```

The `available` column is generally more useful than the `free` column for estimating memory that can be made available to new applications without swapping. Linux uses otherwise-idle memory for caches.

### `lsblk`

```bash
lsblk
lsblk -f
```

`lsblk` lists block devices and their relationships. `-f` adds filesystem information such as filesystem type, label, UUID, and mount point. See the storage section before changing partitions or filesystems.

## 5. General system summary checklist

Run this compact set when first connecting to an unfamiliar host:

```bash
hostnamectl
cat /etc/os-release
uname -r
uptime
lscpu
free -h
lsblk -f
findmnt /
timedatectl status
```



# Performance tools

A small set of Linux tools can show whether a system is limited by CPU, memory, storage, or network activity. Most of the commands below are provided by `procps`; the historical and extended statistics come from `sysstat`.

## Install sysstat

```bash
sudo apt update
sudo apt install sysstat
```

The package provides `iostat`, `mpstat`, `pidstat`, and `sar`.

## CPU utilization with `mpstat`

```bash
mpstat 1 5
mpstat -P ALL 1 5
```

| Option | Meaning |
|---|---|
| `-P ALL` | Show statistics for each CPU. |
| `1 5` | Sample every second, five reports. |

## Disk activity with `iostat`

```bash
iostat -xzy 1 5
```

Extended device statistics help identify high wait times and busy storage devices. Look at read/write throughput, `await`, queue length, and utilization together.

## Per-process activity with `pidstat`

```bash
pidstat 1 5
pidstat -u -r -d 1 5
pidstat -p <PID> -u -r -d 1 5
```

| Option | Meaning |
|---|---|
| `-u` | CPU utilization. |
| `-r` | Memory and page-fault statistics. |
| `-d` | I/O statistics. |
| `-p <PID>` | Monitor a specific process. |
| `1 5` | Sample every second, five times. |

## Virtual memory and system activity with `vmstat`

`vmstat` is supplied by `procps`:

```bash
vmstat -w -y 1 5
vmstat -s
```

Use the repeated report to inspect runnable tasks, blocked tasks, swap activity, CPU idle time, and I/O wait. The `-s` option displays summary counters.

## Historical reports with `sar`

`sar` can report system activity collected over time:

```bash
sar -u 1 5
sar -r 1 5
sar -n DEV 1 5
```

| Option | Meaning |
|---|---|
| `-u` | CPU utilization. |
| `-r` | Memory utilization. |
| `-n DEV` | Network interface statistics. |
| `-f <file>` | Read a previously collected activity file. |

For example, after collection is configured:

```bash
sar -u -f /var/log/sysstat/sa08
```

Historical reports are not available immediately after installing the package. On Debian, enable data collection through the package's configuration (commonly `/etc/default/sysstat`) and check the installed package's systemd units or scheduled jobs. Confirm the actual data-file location with `ls -l /var/log/sysstat/` and the package documentation.

## A simple investigation sequence

1. `uptime` — check load averages.
2. `top` — identify busy processes.
3. `free -h` and `vmstat 1 5` — inspect memory and swap activity.
4. `iostat -xzy 1 5` — inspect storage latency and throughput.
5. `mpstat -P ALL 1 5` — check per-CPU utilization.
6. `pidstat -u -r -d 1 5` — identify per-process CPU, memory, and I/O use.


# CPU and memory

This guide helps you check CPU load, memory pressure, swap activity, and common signs of resource exhaustion.

## CPU load and utilization

### `uptime`

Show uptime and the 1-, 5-, and 15-minute load averages:

```bash
uptime
```

Load average is not CPU percentage. It estimates the average number of tasks that are runnable or waiting in uninterruptible sleep. Compare it with the number of CPUs and the workload; a load of 4 has different meaning on a 2-CPU host and a 32-CPU host.

```bash
nproc
lscpu
```

### `top`

Display a live summary of CPU, memory, and processes:

```bash
top
```

Useful keys inside `top`:

| Key | Meaning |
|---|---|
| `P` | Sort by CPU usage. |
| `M` | Sort by memory usage. |
| `1` | Toggle per-CPU statistics. |
| `H` | Toggle thread display. |
| `q` | Quit. |

Batch output for a quick sample:

```bash
top -b -n 1
```

### CPU details with `mpstat`

`mpstat` is provided by `sysstat`:

```bash
mpstat -P ALL 1 5
```

This samples CPU utilization every second, five times. It is useful for spotting one saturated CPU when the overall average looks normal.

## Memory usage

### `free`

```bash
free -h
free -h -w
```

| Option | Meaning |
|---|---|
| `-h` | Human-readable units. |
| `-w` | Show buffers and cache in separate columns. |
| `-s <seconds>` | Repeat the report at the specified interval. |

The `available` column is generally the best quick estimate of memory available to new applications. Linux uses otherwise-idle memory for filesystem caches, so a low `free` value alone does not mean the host is running out of RAM.

### `/proc/meminfo`

Show detailed kernel memory counters:

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree|Dirty|Writeback):' /proc/meminfo
```

### `vmstat`

```bash
vmstat -w 1 5
```

The first report normally contains averages since boot. Use `-y` to omit that first report and focus on the sampling interval:

```bash
vmstat -w -y 1 5
```

Important columns:

| Column | Meaning |
|---|---|
| `r` | Runnable tasks. |
| `b` | Tasks blocked waiting for I/O. |
| `si` / `so` | Swap-in / swap-out activity. |
| `us` | CPU time in user space. |
| `sy` | CPU time in kernel space. |
| `id` | CPU idle time. |
| `wa` | CPU time waiting for I/O. |
| `st` | CPU time stolen by the hypervisor. |

Sustained swap activity, high I/O wait, or a long runnable queue can indicate pressure, but interpret the values with the workload and other metrics.

## Memory pressure and OOM events

Linux Pressure Stall Information (PSI) exposes time tasks spend stalled because of CPU, memory, or I/O pressure on supported kernels:

```bash
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

Check for out-of-memory events in the current boot's kernel log:

```bash
journalctl -k -b | grep -iE 'out of memory|oom-kill|killed process'
```

A process being killed by the OOM killer usually indicates memory pressure, a memory limit, or a workload spike. Check the service's memory use and any container or cgroup limits before increasing RAM or changing kernel settings.

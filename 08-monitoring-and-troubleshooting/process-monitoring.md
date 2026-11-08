# Process monitoring

Use process tools to find which program is consuming CPU, memory, file descriptors, or disk I/O.
## Find resource-heavy processes

### Sort by CPU or memory with `ps`

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,rss,vsz,etime,cmd --sort=-%cpu | head
ps -eo pid,ppid,user,stat,%cpu,%mem,rss,vsz,etime,cmd --sort=-rss | head
```

| Field | Meaning |
|---|---|
| `PID` / `PPID` | Process and parent process IDs. |
| `%CPU` | CPU utilization reported by `ps`. |
| `%MEM` | Resident memory as a percentage of physical memory. |
| `RSS` | Resident memory in KiB. |
| `VSZ` | Virtual memory size in KiB. |
| `ETIME` | Elapsed time since the process started. |
| `CMD` | Command and arguments. |

RSS is physical memory currently resident for the process; VSZ includes address space that may not be resident. Do not interpret VSZ as actual RAM consumption.

## Inspect a specific process

```bash
ps -p <PID> -o pid,ppid,user,stat,%cpu,%mem,rss,vsz,etime,args
cat /proc/<PID>/status
readlink -f /proc/<PID>/exe
readlink -f /proc/<PID>/cwd
```

Useful `/proc/<PID>/` entries:

| Path | Purpose |
|---|---|
| `status` | Process state, IDs, memory fields, and thread count. |
| `cmdline` | Command-line arguments, separated by NUL bytes. |
| `exe` | Symbolic link to the executable. |
| `cwd` | Symbolic link to the current working directory. |
| `fd/` | Open file descriptors. |
| `smaps_rollup` | Summarized memory-map accounting when available. |

Some process details are restricted by permissions or kernel security settings.

## Monitor threads

A process may have many threads. Inspect them with:

```bash
top -H -p <PID>
ps -T -p <PID>
```


## Inspect open files

With `lsof` installed:

```bash
sudo lsof -p <PID>
sudo lsof -p <PID> | head
sudo ls -l /proc/<PID>/fd
```

## Attribute activity to a process

With `sysstat` installed:

```bash
pidstat -p <PID> -u -r -d 1 5
```

This reports CPU, memory/page-fault, and I/O activity for the selected process.

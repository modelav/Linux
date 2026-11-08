# Network monitoring

These tools help inspect interface counters, connections, traffic rates, and network errors.

## Interface counters

### `ip -s link`

```bash
ip -s link
ip -s link show dev <interface>
```

The output includes received and transmitted bytes, packets, errors, and drops. Increasing error or drop counters may indicate link, driver, queue, or congestion problems; check whether the counters continue increasing during the incident.

## Connection summary

### `ss`

```bash
ss -s
ss -tunap
ss -tan state established
```

| Option | Meaning |
|---|---|
| `-t` | TCP sockets. |
| `-u` | UDP sockets. |
| `-n` | Numeric addresses and ports; do not resolve names. |
| `-a` | Include listening and non-listening sockets. |
| `-p` | Show the process using the socket where permissions allow. |
| `-l` | Show listening sockets. |

Use `sudo` when you need process details for sockets owned by other users.

## Network throughput with `sar`

Provided by `sysstat`:

```bash
sar -n DEV 1 5
sar -n TCP,ETCP 1 5
```

`DEV` reports interface traffic and packet rates. `TCP,ETCP` reports TCP activity and errors. This is useful for observing traffic over time rather than only checking a single counter.

## Live traffic tools

Optional tools provide a simpler view of traffic rates:

```bash
sudo iftop -i <interface>
nload <interface>
vnstat --live --iface <interface>
```

- `iftop` shows traffic flows and approximate rates by connection.
- `nload` shows incoming and outgoing interface rates.
- `vnstat` records traffic totals over time; `--live` shows current rates.


## Check interface counters over time

```bash
watch -n 2 'ip -s link show dev <interface>'
```

`watch` repeats the command. Use the same interface and compare counter changes over time; counters are cumulative since boot or interface reset.


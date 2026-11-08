# Ports and sockets

Ports identify application endpoints on TCP and UDP. Sockets are the kernel interfaces applications use to communicate over networks or locally.

## Show listening sockets with `ss`

`ss` from the `iproute2` package is the recommended tool for inspecting sockets on current Debian systems.

```bash
ss -lnt
ss -lntp
ss -lnup
ss -s
```

| Option | Meaning |
|---|---|
| `-l` | Show listening sockets. |
| `-n` | Show numeric addresses and port numbers; do not resolve names. |
| `-t` | Show TCP sockets. |
| `-u` | Show UDP sockets. |
| `-p` | Show associated process information where permissions allow. |
| `-a` | Show listening and non-listening sockets. |
| `-s` | Show socket summary statistics. |
| `-4` / `-6` | Show IPv4 / IPv6 sockets only. |

Use `sudo` when process details are hidden for sockets owned by other users:

```bash
sudo ss -lntp
sudo ss -lnup
```

## Find a listener on a port

```bash
sudo ss -lntp 'sport = :22'
sudo ss -lntp 'sport = :443'
sudo ss -lunp 'sport = :53'
```

The filter expression in quotes is interpreted by `ss`. `sport` means source port in the socket filter; for listening sockets this identifies the local listening port.

## Show established connections

```bash
ss -tn state established
ss -tn dst <remote-ip>
ss -tan
```

TCP state filters are useful when investigating active connections, connection buildup, or unexpected remote peers.

## Local Unix sockets

Unix-domain sockets provide local inter-process communication and do not use IP addresses or TCP/UDP ports.

```bash
ss -lx
ss -xap
```

| Option | Meaning |
|---|---|
| `-x` | Show Unix-domain sockets. |
| `-l` | Show listening sockets. |
| `-a` | Show all sockets. |
| `-p` | Show process information where permitted. |

## Check whether a TCP port is reachable

From another host, use `nc` (netcat) if installed:

```bash
nc -vz server.example.com 22
nc -vz server.example.com 443
```

`-v` enables verbose output and `-z` checks without sending application data. A successful TCP connection proves that the TCP handshake completed; it does not prove the application protocol is healthy.

## Understand a port conflict

If a service cannot bind to a port:

1. Check for an existing listener with `sudo ss -lntup`.
2. Identify the process and service.
3. Check whether the application is configured to use the intended address and port.
4. Check IPv4 and IPv6 binding behavior; a wildcard listener may occupy more than one address family depending on socket settings.


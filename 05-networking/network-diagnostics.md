# Network diagnostics

This guide collects common tools for testing connectivity at different layers: IP reachability, DNS, TCP/UDP ports, HTTP(S), and packet paths.

## `ping` — test IP reachability

`ping` is provided by Debian's `iputils-ping` package.

```bash
ping -c 4 192.0.2.1
ping -4 -c 4 debian.org
ping -6 -c 4 debian.org
```

| Option | Meaning |
|---|---|
| `-c <count>` | Stop after sending the specified number of packets. |
| `-4` / `-6` | Use IPv4 / IPv6 only. |
| `-I <interface>` | Select the source interface or address. |
| `-W <seconds>` | Wait this long for a response to an individual probe. |
| `-w <seconds>` | Set an overall deadline. |

Some hosts and networks block ICMP. A failed ping is not proof that a TCP service is unavailable.

## `traceroute` — inspect the path

Install the `traceroute` package if the command is missing.

```bash
traceroute debian.org
traceroute -n debian.org
traceroute -I debian.org
```

| Option | Meaning |
|---|---|
| `-n` | Do not resolve hop addresses to hostnames. |
| `-I` | Use ICMP probes. |
| `-T` | Use TCP probes (where supported by the installed implementation). |
| `-4` / `-6` | Use IPv4 / IPv6. |

Firewalls and routers may filter or rate-limit probes, so missing hops do not necessarily indicate a broken route.

## `mtr` — continuous path diagnostics

`mtr` combines traceroute-style path discovery with repeated measurements. Install `mtr-tiny` if it is not present.

```bash
mtr debian.org
mtr -rwzc 20 debian.org
```

| Option | Meaning |
|---|---|
| `-r` | Report mode; print results and exit. |
| `-w` | Wide report format. |
| `-z` | Show ASN information when available. |
| `-c <count>` | Number of probes per hop in report mode. |

Packet loss at an intermediate hop that does not continue to the destination may indicate ICMP rate limiting rather than forwarding loss.

## `curl` — test application connectivity

```bash
curl -I https://debian.org/
curl -v https://example.com/
curl -4 -I https://example.com/
curl -6 -I https://example.com/
```

| Option | Meaning |
|---|---|
| `-I` | Fetch response headers only (HTTP HEAD). |
| `-v` | Show connection and protocol details. |
| `-4` / `-6` | Use IPv4 / IPv6. |
| `-L` | Follow HTTP redirects. |
| `--connect-timeout <seconds>` | Limit time spent establishing the connection. |
| `--max-time <seconds>` | Limit total operation time. |

Do not use `-k`/`--insecure` as a routine TLS fix; it disables certificate verification and can hide certificate or interception problems.

## `nc` — test a TCP port

```bash
nc -vz -w 3 server.example.com 443
```

| Option | Meaning |
|---|---|
| `-v` | Verbose output. |
| `-z` | Scan/check without sending application data. |
| `-w <seconds>` | Set connection timeout. |

Different netcat implementations support slightly different options. Check `man nc` on the installed package.

## Packet capture with `tcpdump`

`tcpdump` captures packets for troubleshooting.

```bash
sudo tcpdump -ni enp1s0
sudo tcpdump -ni enp1s0 host 192.0.2.10
sudo tcpdump -ni enp1s0 'tcp port 443'
sudo tcpdump -ni any 'port 53'
```

| Option | Meaning |
|---|---|
| `-i <interface>` | Capture on the selected interface; `any` selects all supported interfaces. |
| `-n` | Do not resolve addresses or ports to names. |
| `-c <count>` | Stop after capturing the specified number of packets. |
| `-w <file>` | Write packets to a pcap file instead of printing decoded output. |
| `-r <file>` | Read packets from a saved capture file. |

Packet captures can contain credentials, session data, personal information, and internal network details.

## A practical troubleshooting order

1. `ip -br link` — is the interface present and up?
2. `ip -br addr` — does it have the expected address?
3. `ip route` and `ip route get <destination>` — is there a valid route?
4. `ping <gateway>` — can the local next hop be reached, if ICMP is allowed?
5. `getent hosts <name>` and `dig <name>` — does name resolution work?
6. `nc -vz <host> <port>` — can the required TCP port be reached?
7. `curl -v <url>` — can the application protocol complete?
8. `tcpdump` — inspect packets when the previous checks are inconclusive.

# Networking basics

This guide introduces the basic pieces of a Linux network configuration and the commands used to inspect them.

## Network layers in practice

A server normally needs several things to communicate:

1. A network interface connected to a link.
2. An IP address and prefix on that interface.
3. A route to the destination (often a default gateway).
4. DNS configuration when communicating by hostname.
5. Firewall rules that permit the required traffic.



## Network interface names

Modern Linux systems commonly use predictable interface names such as `ens18`, `enp1s0`, or `eno1` instead of always using `eth0`. Names depend on hardware, firmware, virtualization, and naming policy.

Useful commands:

```bash
ip -br link
ip -br addr
ip -s link
```

| Command / option | Meaning |
|---|---|
| `ip link` | Show or manage network interfaces. |
| `-br` | Brief, one-line-per-object output. |
| `-s` | Include interface statistics such as RX/TX packets and errors. |
| `ip addr` | Show IP addresses assigned to interfaces. |

Example interface details:

```bash
ip link show dev enp1s0
ip addr show dev enp1s0
```

Important interface attributes:

| Attribute | Meaning |
|---|---|
| State | Whether the interface is administratively up/down and its operational state. |
| MAC address | Layer-2 hardware or virtual interface address. |
| MTU | Maximum Layer-3 packet size accepted by the interface. Ethernet commonly uses 1500 bytes, but tunnels and other networks may differ. |
| RX/TX counters | Received and transmitted packets, bytes, errors and drops. |

## Loopback interface

The `lo` interface is used for local communication within the host. It normally has the address `127.0.0.1/8` for IPv4 and `::1/128` for IPv6.

```bash
ip addr show dev lo
ping -c 2 127.0.0.1
ping -6 -c 2 ::1
```

local services commonly depend on loopback.

## Common network configuration files

| Path | Purpose |
|---|---|
| `/etc/hostname` | System hostname. |
| `/etc/hosts` | Static local hostname mappings. |
| `/etc/resolv.conf` | Resolver configuration or a symlink managed by a network/DNS service. |
| `/etc/network/interfaces` | `ifupdown` interface configuration, when used. |
| `/etc/systemd/network/` | `systemd-networkd` `.network`, `.netdev`, and `.link` files. |
| `/etc/NetworkManager/` | NetworkManager configuration and drop-ins. |
| `/etc/nftables.conf` | Common persistent nftables ruleset file on Debian when the nftables service is used. |

## Quick connectivity checks

```bash
ip -br addr
ip route
ping -c 3 <default-gateway>
ping -c 3 1.1.1.1
getent ahosts debian.org
curl -I https://debian.org/
```

These tests answer different questions:

- `ip -br addr`: Does the interface have an address?
- `ip route`: Is there a route to the destination?
- `ping` to the gateway: Can the host reach its local next hop using ICMP?
- `ping` to a public IP: Is there IP connectivity beyond the local network, if ICMP is permitted?
- `getent`: Can the system's configured name-service path resolve a hostname?
- `curl`: Can an application establish an HTTP(S) connection?

A failed ping alone does not prove that a host or service is down; ICMP may be filtered.


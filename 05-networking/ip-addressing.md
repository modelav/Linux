# IP addressing

Use the `ip` command from the `iproute2` package to inspect and manage IPv4 and IPv6 addresses and network interfaces.

## Show addresses

```bash
ip address
ip -br address
ip -4 address
ip -6 address
ip address show dev enp1s0
```

| Option / command | Meaning |
|---|---|
| `-br` | Brief output. |
| `-4` | Show IPv4 information only. |
| `-6` | Show IPv6 information only. |
| `show dev <interface>` | Show addresses on one interface. |
| `-s` | Include statistics when used with link inspection. |

## Add or remove a runtime address

These commands change the live interface configuration. The change is normally temporary and may disappear when the interface is reconfigured or the system reboots.

```bash
sudo ip address add 192.0.2.10/24 dev enp1s0
sudo ip address del 192.0.2.10/24 dev enp1s0
```

IPv6 example:

```bash
sudo ip -6 address add 2001:db8:10::10/64 dev enp1s0
```

## Bring an interface up or down

```bash
ip link show
sudo ip link set dev enp1s0 up
sudo ip link set dev enp1s0 down
```

`up` and `down` change the administrative state. Taking down the interface used by SSH disconnects remote sessions. Persistent interface state should be configured through the host's network manager, not by relying on `ip link set` at boot.

## Replace an address

```bash
sudo ip address replace 192.0.2.20/24 dev enp1s0
```

`replace` adds the address if it is absent or replaces the matching address configuration if it exists. Confirm the interface and address before making changes.

## Inspect neighbors

The neighbor table (often called the ARP table for IPv4) maps local-network IP addresses to link-layer addresses.

```bash
ip neigh
ip -4 neigh
ip -6 neigh
ip neigh show dev enp1s0
```

| State | Meaning |
|---|---|
| `REACHABLE` | Recently confirmed reachable. |
| `STALE` | Mapping exists but has not recently been confirmed. |
| `DELAY` / `PROBE` | Reachability is being checked. |
| `FAILED` | Neighbor resolution failed. |

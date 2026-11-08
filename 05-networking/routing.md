# Routing

Routing determines where packets are sent next. Use `ip route` for IPv4 and `ip -6 route` for IPv6. See the [Debian `ip-route` manual](https://manpages.debian.org/trixie/iproute2/ip-route.8.en.html) for route syntax and policy details.

## View routing tables

```bash
ip route
ip -4 route
ip -6 route
ip route show table main
ip route get 1.1.1.1
```

| Command / option | Meaning |
|---|---|
| `ip route` | Show IPv4 routes in the main table by default. |
| `ip -6 route` | Show IPv6 routes. |
| `show table main` | Show routes in the main routing table. |
| `get <destination>` | Ask the kernel which route it would use to reach a destination. |

A default route (`default via ...`) is used when no more-specific route matches the destination.

## Add and remove a route

These changes affect the live routing table and may be lost when networking is restarted or the host reboots.

```bash
sudo ip route add 192.0.2.0/24 via 192.0.2.1 dev enp1s0
sudo ip route del 192.0.2.0/24 via 192.0.2.1 dev enp1s0
```

Add a default gateway:

```bash
sudo ip route add default via 192.0.2.1 dev enp1s0
```

Use `replace` to add a route or update the matching route:

```bash
sudo ip route replace default via 192.0.2.254 dev enp1s0
```

For IPv6, use `ip -6 route` and an IPv6 next-hop address:

```bash
sudo ip -6 route add default via 2001:db8:10::1 dev enp1s0
```

Use the actual gateway and interface for your network. A gateway normally needs to be reachable through the selected interface unless special routing configuration is in use.

## Route metrics

When multiple routes can reach the same destination, metrics can influence route selection. Lower metric values are generally preferred among otherwise comparable routes.

```bash
ip route
sudo ip route add default via 192.0.2.1 dev enp1s0 metric 100
```

The `metric` example creates a runtime route. Configure persistent route metrics through the active network manager.

## Policy routing

Linux supports multiple routing tables and rules that select a table based on properties such as source address, destination, or packet mark.

```bash
ip rule show
ip route show table all
```

Use `ip rule` and named routing tables only when you understand the routing policy. Complex multi-WAN, VPN, container, and source-based routing setups can behave unexpectedly if rules or routes are changed in isolation.

## Troubleshooting workflow

```bash
ip -br address
ip route
ip route get <destination-ip>
ip neigh
ping -c 3 <gateway-ip>
```

Check the source address, outgoing interface, gateway, route metric, and neighbor state. If the destination is remote, verify the return route as well; a correct outbound route does not guarantee a working two-way connection.

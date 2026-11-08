# Firewall security practices

A firewall should expose only the network services the server needs. This guide focuses on security policy and operational checks.

## Start with an allow-list

For a typical server, define:

- Which services must be reachable from the internet.
- Which services should be reachable only from trusted management networks.
- Which ports are internal-only or should listen on loopback.
- Whether IPv4 and IPv6 are both in use.
- Whether outbound access should be unrestricted or limited for the server's role.

A default-deny inbound policy is a common starting point, followed by explicit allowances for required services such as SSH, HTTP, and HTTPS. Do not copy an allow-list without checking the actual service ports and management source addresses.

## Review listening sockets

```bash
sudo ss -tulpn
sudo ss -lntup
```

Check each listening socket against the server's intended role. A service listening on `0.0.0.0` or `::` may accept connections on all relevant interfaces; a service bound to `127.0.0.1` or `::1` is loopback-only.

## Restrict management access

- Allow SSH only from trusted management networks where possible.
- Avoid exposing database, cache, metrics, and administration ports directly to the public internet.
- Use VPN or private-network access for administrative interfaces when appropriate.
- Keep an out-of-band console or recovery path for firewall changes.


## Remember IPv6

A firewall policy that protects IPv4 but leaves IPv6 unfiltered can still expose services. Check both address families and include IPv6 rules where the host has IPv6 connectivity.

```bash
ip -br address
ip -6 route
sudo ss -lntup
```

## Avoid competing firewall managers

Do not assume nftables, UFW, Docker, container platforms, VPN software, and cloud firewalls manage independent rule sets. They may all change packet filtering behavior or add rules dynamically.

Before applying a ruleset, identify the current firewall manager and inspect the full ruleset:

```bash
sudo nft list ruleset
systemctl status nftables
```

A command such as `nft flush ruleset` removes the entire nftables ruleset and can break connectivity, containers, VPNs, or other software. Do not use it on a shared host unless you fully own and intend to replace all rules.

## Verify a firewall change

1. Save the current ruleset and configuration.
2. Confirm the intended access is allowed, including SSH and any required IPv6 traffic.
3. Validate the ruleset syntax before loading it.
4. Apply changes during a maintenance window where possible.
5. Test from a second client or external network.
6. Keep the current remote session and console access until the new policy is confirmed.

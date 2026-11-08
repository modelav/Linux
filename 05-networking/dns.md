# DNS

DNS translates domain names into resource records such as IP addresses. On Debian, name resolution may involve `/etc/hosts`, the Name Service Switch (NSS), `/etc/resolv.conf`, and optionally `systemd-resolved` or another local resolver.

## Important files

| Path | Purpose |
|---|---|
| `/etc/resolv.conf` | Nameserver addresses and search-domain settings used by traditional resolver clients. It may be a symlink managed by another service. |
| `/etc/hosts` | Static hostname-to-address entries. |
| `/etc/nsswitch.conf` | Controls the order of sources used by NSS, including the `hosts:` line. |
| `/etc/systemd/resolved.conf` | Optional systemd-resolved global configuration. |

Before editing `/etc/resolv.conf`, inspect it and determine which service manages it:

```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
cat /etc/nsswitch.conf | grep '^hosts:'
```

Do not overwrite a symlink-managed resolver file; the network manager or resolver service may replace it.

## Test name resolution

### `getent`

`getent` uses the system's configured NSS lookup path, which makes it useful for testing what normal applications see.

```bash
getent hosts debian.org
getent ahosts debian.org
getent hosts example.internal
```

### `dig`

Install `dig` from the `bind9-dnsutils` package if it is not available:

```bash
sudo apt install bind9-dnsutils
```

```bash
dig debian.org
dig A debian.org
dig AAAA debian.org
dig MX debian.org
dig @192.0.2.53 debian.org
dig +short debian.org
dig -x 192.0.2.10
```

| Option / argument | Meaning |
|---|---|
| `@<server>` | Query a specific DNS server. |
| `A` / `AAAA` | Query IPv4 / IPv6 address records. |
| `MX` | Query mail-exchange records. |
| `TXT` | Query text records. |
| `+short` | Show a compact answer. |
| `-x <address>` | Perform a reverse lookup. |

`dig` queries DNS directly; it does not follow the complete NSS path used by all applications. Use `getent` to test system name resolution.

### `host`

```bash
host debian.org
host -t MX debian.org
host 192.0.2.10
```

`host` is another DNS lookup utility supplied by `bind9-dnsutils`.

## systemd-resolved

`systemd-resolved` is a separate package and is not installed by default on every Debian server. To install it when you deliberately choose this resolver:

```bash
sudo apt install systemd-resolved
```

When it is installed and active, use `resolvectl` to inspect and query its resolver state. See the [Debian `systemd-resolved` manual](https://manpages.debian.org/trixie/systemd-resolved/systemd-resolved.service.8.en.html).

```bash
systemctl status systemd-resolved
resolvectl status
resolvectl query debian.org
resolvectl dns
resolvectl domain
```

| Command | Meaning |
|---|---|
| `status` | Show per-link DNS settings and global resolver state. |
| `query <name>` | Resolve a hostname or address through systemd-resolved. |
| `dns` | Show configured DNS servers. |
| `domain` | Show configured search and routing domains. |
| `flush-caches` | Flush the resolver cache. |

Only use `resolvectl` commands when `systemd-resolved` is installed and running. The official [Debian `resolvectl` manual](https://manpages.debian.org/trixie/systemd-resolved/resolvectl.1.en.html) describes its available commands.

## Configure DNS

The correct place to configure DNS depends on the service managing the interface:

- `ifupdown`: configuration in `/etc/network/interfaces`; DNS integration may require additional resolver tooling.
- `systemd-networkd`: DNS can be supplied in `.network` files and passed to systemd-resolved when it is used.
- NetworkManager: DNS is configured per connection profile, commonly using `nmcli`.
- Static `/etc/resolv.conf`: only when no service is expected to manage it.

Do not configure DNS independently in several places; the active manager may overwrite manual edits.

## Troubleshooting

1. Check IP connectivity to a known address.
2. Check `/etc/resolv.conf` and whether it is managed by a service.
3. Test with `getent hosts <name>`.
4. Query a known DNS server directly with `dig @<server> <name>`.
5. If using systemd-resolved, inspect `resolvectl status` and relevant logs with `journalctl -u systemd-resolved`.
6. Check the `hosts:` line in `/etc/nsswitch.conf` and any local `/etc/hosts` entry.

A successful direct `dig` query combined with a failed `getent` lookup often points to local resolver/NSS configuration rather than basic network reachability.

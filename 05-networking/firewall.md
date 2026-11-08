# Firewall with nftables

`nftables` is the Linux packet-filtering framework used by current Debian systems. The `nft` command manages tables, chains, sets, and rules. The Debian `nftables` package provides the command and a systemd service for loading a persistent ruleset. See the [Debian nftables manual](https://manpages.debian.org/trixie/nftables/nft.8.en.html) and the [nftables server ruleset example](https://wiki.nftables.org/wiki-nftables/index.php/Simple_ruleset_for_a_server).

## Inspect the current ruleset

```bash
sudo nft list ruleset
sudo nft list tables
sudo systemctl status nftables
```

| Command | Meaning |
|---|---|
| `nft list ruleset` | Display all currently loaded nftables rules. |
| `nft list tables` | List tables. |
| `nft list table inet filter` | Show one table. |
| `nft -a list ruleset` | Include rule handles, useful when deleting or replacing a specific rule. |

## nftables concepts

| Object | Purpose |
|---|---|
| Table | Container for chains, sets, maps, and rules. Common families include `ip`, `ip6`, and `inet`. |
| Chain | Holds rules. Base chains attach to a packet-processing hook; regular chains are used as jump targets. |
| Rule | Matches packets and performs an action such as `accept`, `drop`, `reject`, or `jump`. |
| Set | Stores multiple values that rules can match efficiently. |
| Hook | Defines where a base chain participates in packet processing, such as `input`, `output`, or `forward`. |
| Policy | Default verdict for packets that reach the end of a base chain. |

The `inet` family can handle both IPv4 and IPv6 in the same table.

## Example server ruleset

The following is a starting example for a simple server that accepts inbound SSH, HTTP, and HTTPS, allows established connections, and drops other unsolicited inbound traffic. Adjust it to the actual services and management source addresses on the host.

```nft
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;

        # Local traffic
        iifname "lo" accept

        # Invalid packets
        ct state invalid drop

        # Replies to connections initiated by this host
        ct state established,related accept

        # Permit essential ICMP and ICMPv6 traffic
        ip protocol icmp accept
        meta l4proto ipv6-icmp accept

        # SSH, HTTP and HTTPS (IPv4 and IPv6)
        tcp dport { 22, 80, 443 } accept
    }

    chain forward {
        type filter hook forward priority filter; policy drop;
    }

    chain output {
        type filter hook output priority filter; policy accept;
    }
}
```

Important notes:

- This sample assumes SSH listens on TCP port 22. Change it if the server uses a different port.
- For a public or sensitive server, restrict SSH to trusted management IP addresses or networks where practical.
- The `inet` table covers IPv4 and IPv6. Test both address families.
- ICMPv6 is important for IPv6 neighbor discovery and path-MTU behavior. Do not block it indiscriminately.
- If the host routes traffic, the `forward` chain needs an intentional forwarding policy rather than the sample's drop policy.
- `flush ruleset` removes all nftables tables and rules, including rules potentially managed by other software. Do not use it on a host where containers, VPN software, virtualization, or another firewall manager owns part of the ruleset without understanding the consequences.

## Load and verify a ruleset

If the `nft` command is missing, install the `nftables` package first:

```bash
sudo apt install nftables
```

Save the ruleset in `/etc/nftables.conf`, then check its syntax before applying it:

```bash
sudo nft -c -f /etc/nftables.conf
```

`-c` checks the ruleset without applying it. If validation succeeds, load it:

```bash
sudo nft -f /etc/nftables.conf
sudo nft list ruleset
```

**Remote access warning:** A firewall mistake can immediately disconnect SSH or block future access. Before applying a default-drop policy, verify that the correct SSH port and management source are allowed. Use a provider console or other out-of-band recovery path, and consider having a tested rollback plan. Do not rely on your current SSH session remaining connected as proof that new connections will work.

## Make nftables persistent

On Debian, the `nftables` package provides `/etc/nftables.conf` and the `nftables.service` unit for loading rules at boot. After validating the file:

```bash
sudo systemctl enable --now nftables
systemctl status nftables
```

Check the service and resulting ruleset after a reboot during a controlled maintenance window.

## Add or delete a rule interactively

Rules can be added with `nft` commands, but rules added only at runtime are not automatically persistent unless also saved in the configuration loaded at boot.

```bash
sudo nft add rule inet filter input tcp dport 8443 accept
sudo nft -a list chain inet filter input
```

Delete a specific rule by its handle:

```bash
sudo nft delete rule inet filter input handle <handle>
```

Use `-a` to identify the handle. Rule handles can change after ruleset updates; inspect the chain immediately before deleting a rule.

## Counters and logs

Counters help determine whether rules are matching traffic. Add `counter` to a rule when configuring it, then inspect counters with:

```bash
sudo nft list ruleset
```

Logging dropped traffic can generate large logs. Use rate limits and a clear prefix if adding log rules; avoid logging every packet on a busy server.

## Troubleshooting

```bash
sudo nft list ruleset
sudo systemctl status nftables
sudo journalctl -u nftables -b
sudo ss -lntup
```

Check the packet's direction, address family, interface, protocol, port, and connection state. Confirm that the application is actually listening before changing firewall rules.

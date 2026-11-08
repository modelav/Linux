# Network configuration

Debian can configure network interfaces through different network managers. The correct configuration method depends on the system's current setup. Do not configure the same interface independently with multiple managers.

## Identify the active network manager

```bash
systemctl is-active networking
systemctl is-active systemd-networkd
systemctl is-active NetworkManager
ps -ef | grep -E '[i]fup|[s]ystemd-networkd|[N]etworkManager'
```

Inspect common configuration locations:

```bash
cat /etc/network/interfaces
find /etc/systemd/network -maxdepth 1 -type f -print 2>/dev/null
find /etc/NetworkManager/system-connections -maxdepth 1 -type f -print 2>/dev/null
```

These commands provide clues; an installed or active service does not necessarily manage every interface. `NetworkManager` may leave interfaces configured in `/etc/network/interfaces` unmanaged by default.

## ifupdown

The [`interfaces(5)` manual](https://manpages.debian.org/trixie/ifupdown/interfaces.5.en.html) documents `/etc/network/interfaces` syntax.

`ifupdown` uses `/etc/network/interfaces` and files included from it. This is common on traditional server installations.

### DHCP example

```ini
# /etc/network/interfaces
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto enp1s0
iface enp1s0 inet dhcp
```

### Static IPv4 example

```ini
auto enp1s0
iface enp1s0 inet static
    address 192.0.2.10/24
    gateway 192.0.2.1
```

Useful commands:

```bash
ifquery enp1s0
sudo ifup enp1s0
sudo ifdown enp1s0
```

Do not run `ifdown` on the interface carrying your SSH session unless you have console or out-of-band access.

## systemd-networkd

The [Debian `systemd.network` manual](https://manpages.debian.org/trixie/systemd/systemd.network.5.en.html) documents `.network` file settings.

`systemd-networkd` uses `.network` files under `/etc/systemd/network/`. A `.network` file matches an interface and defines its network configuration.

### DHCP example

```ini
# /etc/systemd/network/20-wired.network
[Match]
Name=enp1s0

[Network]
DHCP=yes
```

### Static IPv4 example

```ini
# /etc/systemd/network/20-wired.network
[Match]
Name=enp1s0

[Network]
Address=192.0.2.10/24
Gateway=192.0.2.1
DNS=192.0.2.53
```

DNS settings are integrated with `systemd-resolved` when that resolver is used. Do not assume that writing `DNS=` alone configures `/etc/resolv.conf` on a host without the resolver integration.

Useful commands:

```bash
networkctl status
networkctl status enp1s0
networkctl list
```

After changing a networkd configuration, reload and reconfigure only when you understand the impact:

```bash
sudo networkctl reload
sudo networkctl reconfigure enp1s0
```

## NetworkManager

The [Debian `nmcli` manual](https://manpages.debian.org/trixie/network-manager/nmcli.1.en.html) documents connection and device commands.

NetworkManager can manage headless servers as well as desktops. Use `nmcli` for connection profiles and device state.

```bash
nmcli general status
nmcli device status
nmcli connection show
nmcli connection show --active
```

Create a DHCP connection profile:

```bash
sudo nmcli connection add type ethernet ifname enp1s0 con-name wired-dhcp ipv4.method auto
```

Create a static IPv4 profile:

```bash
sudo nmcli connection add type ethernet ifname enp1s0 con-name wired-static \
  ipv4.method manual ipv4.addresses 192.0.2.10/24 \
  ipv4.gateway 192.0.2.1 ipv4.dns 192.0.2.53
```

Activate a profile:

```bash
sudo nmcli connection up wired-static
```

Useful `nmcli` commands:

| Command | Meaning |
|---|---|
| `nmcli device status` | Show device state. |
| `nmcli connection show` | List saved connection profiles. |
| `nmcli connection show --active` | List active profiles. |
| `nmcli connection up <name>` | Activate a profile. |
| `nmcli connection down <name>` | Deactivate a profile. |
| `nmcli connection modify <name> <setting> <value>` | Change profile settings. |
| `nmcli connection reload` | Reload saved profiles from disk. |

NetworkManager supports checkpoints for some disruptive remote changes. A checkpoint can restore the previous configuration if you do not confirm the change before the timeout:

```bash
sudo nmcli device checkpoint --timeout 60 enp1s0 -- nmcli connection up wired-static
```

Confirm the change when prompted only after verifying that the new connection works.

## Remote server warning

Changing the IP address, gateway, VLAN, DNS, or network manager can immediately disconnect SSH and may prevent the server from reconnecting. Before applying changes:

- Confirm the interface name and current configuration.
- Keep provider console, serial console, hypervisor console, or other out-of-band access available.
- Prefer a scheduled maintenance window.
- Back up the current configuration.
- Validate configuration syntax before applying it.
- Keep an existing session open and test a second connection before closing it.

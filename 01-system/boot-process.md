# Boot process and startup troubleshooting

## 1. High-level boot sequence

A typical Debian server boot proceeds through these stages:

1. **Firmware:** BIOS or UEFI initializes hardware and selects a boot device.
2. **Bootloader:** Commonly GRUB on Debian installations; it presents or selects a kernel entry and loads the kernel and initramfs.
3. **Linux kernel:** Initializes CPU, memory, device drivers, and core kernel facilities.
4. **Initramfs:** A temporary early-userspace environment that can load required modules and locate or unlock the real root filesystem.
5. **Root filesystem handoff:** The system transitions to the installed root filesystem.
6. **PID 1 / system manager:** On a standard Debian systemd installation, systemd becomes process ID 1 and starts units according to dependencies and the selected target.
7. **System services and login:** Filesystems, networking, system services, and login facilities become available according to the host's configuration.

## 2. Inspect the current boot

### Boot time and startup duration

```bash
uptime -s
systemd-analyze time
systemd-analyze blame
systemd-analyze critical-chain
```

| Command / option | Meaning |
|---|---|
| `systemd-analyze time` | Summarizes time spent in firmware, bootloader, kernel, and userspace when timing information is available. |
| `systemd-analyze blame` | Lists units ordered by time spent in initialization; it does not by itself prove which unit delayed the boot critical path. |
| `systemd-analyze critical-chain` | Shows the critical dependency chain and timing information for boot. |
| `systemd-analyze plot` | Emits an SVG boot timeline to standard output; redirect it to a file if you want to inspect it. |



### Default target and failed units

```bash
systemctl get-default
systemctl list-units --failed
systemctl --failed
```

A systemd **target** groups units into a desired system state. Common targets include `multi-user.target` for a non-graphical multi-user server and `graphical.target` for a graphical system. The default target is not necessarily the only target reached during a boot.

## 3. Read boot logs

```bash
journalctl -b
journalctl -b -p warning
journalctl -b -p err
journalctl -b -k
journalctl -b -u <unit-name>
```

| Option | Meaning |
|---|---|
| `-b` | Select the current boot. Use `-b -1` for the previous boot if that journal is retained. |
| `-p warning` | Include warning and more severe priorities. |
| `-p err` | Include error and more severe priorities. |
| `-k` | Show kernel messages. |
| `-u <unit>` | Filter entries for a systemd unit. |

If logs from a previous boot are missing, the journal may be volatile, previous logs may have been vacuumed, or the journal may not have been retained.

## 4. Kernel and initramfs files

Common paths on a Debian installation include:

| Path | Purpose |
|---|---|
| `/boot/vmlinuz-*` | Installed kernel images. |
| `/boot/initrd.img-*` | Initramfs images used during early userspace. |
| `/boot/grub/` | GRUB configuration and supporting files when GRUB is installed. |
| `/etc/default/grub` | Local defaults used when generating GRUB configuration. |
| `/etc/initramfs-tools/` | Configuration for `initramfs-tools`, when that implementation is installed. |


## 5. Common startup failure patterns

| Symptom | Possible area to investigate | First checks |
|---|---|---|
| Server stops before login is available | Firmware, bootloader, kernel, initramfs, root filesystem | Console output, bootloader entry, storage availability, `journalctl -b -k` after recovery. |
| A service fails during boot | Unit configuration, dependencies, unavailable filesystems, application error | `systemctl status <unit>` and `journalctl -b -u <unit>`. |
| Network is unavailable after boot | Network configuration, link, DHCP, DNS, firewall | Network interface and routing diagnostics; see the networking section. |
| Boot is unexpectedly slow | Device timeouts, network mounts, service dependencies, hardware | `systemd-analyze time`, `critical-chain`, and boot journal. |
| Previous boot logs are unavailable | Volatile journal or retention/rotation | `journalctl --list-boots` and journald storage configuration. |


## 6. Rescue and emergency targets

Systemd provides special targets for recovery:

- `rescue.target` generally provides a minimal system with essential services and a maintenance shell.
- `emergency.target` provides a more minimal emergency environment.

These modes can require local or out-of-band console access and may require the root password.

## 7. High-impact operations

The following operations can make a host unbootable when performed incorrectly:

- Changing bootloader configuration or kernel command-line parameters.
- Rebuilding or replacing initramfs images.
- Removing the currently required kernel or its modules.
- Changing root filesystem, encryption, or mount configuration.
- Changing firmware boot order or Secure Boot settings.

Before making such changes, confirm the active boot mode, installed bootloader, kernel package, root device, backups, and recovery-console access. Use operating system's release notes and the relevant tool documentation for the specific host.

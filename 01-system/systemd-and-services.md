
# systemd and service management

## 1. What is systemd?

On a standard Linux systemd installation, `systemd` is the system and service manager and normally runs as PID 1. It starts and supervises units, coordinates dependencies, handles service lifecycle operations, and participates in boot and shutdown.

The main command-line interface for the system manager is `systemctl`.

## 2. Unit types

A systemd unit describes an object managed by systemd. Common unit types include:

| Suffix | Type | Purpose |
|---|---|---|
| `.service` | Service | A process or action managed by systemd. |
| `.socket` | Socket | A listening socket that can activate a related service. |
| `.timer` | Timer | Activates another unit at a time or interval. |
| `.mount` | Mount | Represents a filesystem mount. |
| `.automount` | Automount | Activates a mount when its path is accessed. |
| `.target` | Target | Groups units into a synchronization point or system state. |
| `.path` | Path | Activates a unit when a filesystem path changes. |
| `.device` | Device | Represents a device exposed to systemd, usually through udev. |
| `.slice` | Slice | Groups processes for resource management. |


## 3. Inspect services and units

### `systemctl status`

```bash
systemctl status <unit-name>
systemctl status ssh
```

Status output typically includes whether the unit is loaded, active, its main PID, recent journal messages, and the unit file path. A unit can be installed but inactive, or active but not enabled to start automatically at boot.

### Common inspection commands

| Command | Purpose |
|---|---|
| `systemctl is-active <unit>` | Check whether a unit is active. |
| `systemctl is-enabled <unit>` | Check whether a unit is enabled in its installation configuration. |
| `systemctl is-failed <unit>` | Check whether a unit is in a failed state. |
| `systemctl list-units` | List units currently loaded in systemd's memory. |
| `systemctl list-units --type=service` | Show loaded service units. |
| `systemctl list-units --state=failed` | Show loaded units in failed state. |
| `systemctl list-unit-files` | List installed unit files and their enablement state. |
| `systemctl cat <unit>` | Show the unit file and associated drop-ins. |
| `systemctl show <unit>` | Show properties understood by systemd. |
| `systemctl get-default` | Show the default boot target. |

Useful filters:

```bash
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed
systemctl list-unit-files --type=service
```

`list-units` and `list-unit-files` answer different questions: loaded runtime units versus unit files found by systemd.

## 4. Start, stop, restart, and reload

These operations change runtime state and normally require root privileges.

| Command | Effect | Notes |
|---|---|---|
| `systemctl start <unit>` | Start a unit now. | Does not by itself enable it at boot. |
| `systemctl stop <unit>` | Stop a unit now. | Can interrupt users or dependent services. |
| `systemctl restart <unit>` | Stop and start a unit. | May interrupt existing connections or work. |
| `systemctl reload <unit>` | Ask a service to reload its configuration. | Only works if the unit/service supports reload. |
| `systemctl reload-or-restart <unit>` | Reload if supported; otherwise restart. | Can still interrupt service if it falls back to restart. |
| `systemctl try-restart <unit>` | Restart only if the unit is already active. | Useful for maintenance automation. |

Do not assume every service supports reload. Check `systemctl status`, `systemctl show`, and the service's documentation. For network-facing services, validate configuration with the application's own syntax-check command before reloading.

## 5. Enablement at boot

A service being **active** is different from being **enabled**. Enablement controls whether systemd should start a unit automatically through its configured installation relationships.

These operations normally require root privileges.

| Command | Effect |
|---|---|
| `systemctl enable <unit>` | Configure the unit to start through its install relationships at boot or another activation point. Does not normally start it immediately. |
| `systemctl disable <unit>` | Remove those enablement links. Does not normally stop a currently running unit. |
| `systemctl enable --now <unit>` | Enable and start the unit. |
| `systemctl disable --now <unit>` | Disable and stop the unit. |

```bash
sudo systemctl enable --now cron
systemctl is-enabled cron
systemctl is-active cron
```

Check dependencies and service requirements before disabling a unit on a production host.

## 6. Mask and unmask a unit

Masking prevents a unit from being started, including as a dependency, by linking it to `/dev/null` in systemd's unit search path. This is stronger than disabling it.

These operations normally require root privileges and can disrupt dependent services.

```bash
# Inspect first
systemctl status <unit>

# Prevent activation
sudo systemctl mask <unit>

# Remove the mask
sudo systemctl unmask <unit>
```

Do not mask core system services as a troubleshooting shortcut. A masked unit can cause other units to fail when they require it.

## 7. Unit files and configuration overrides

System unit files supplied by Debian packages normally live under locations such as `/usr/lib/systemd/system/` or `/lib/systemd/system/` depending on the system's layout. Local administrator configuration commonly belongs in `/etc/systemd/system/`.

Do not edit package-supplied unit files in place. Use a drop-in override for local changes:

```bash
sudo systemctl edit <unit-name>
```

This opens an override editor and normally creates a drop-in under a path such as `/etc/systemd/system/<unit-name>.service.d/override.conf`.

To view the complete unit definition and drop-ins:

```bash
systemctl cat <unit-name>
```

After manually creating or changing unit files, ask systemd to reload unit definitions:

```bash
sudo systemctl daemon-reload
```

`daemon-reload` makes systemd re-read unit files; it does not itself restart a service or apply every changed application setting to a running process. Restart or reload the relevant unit when required.

## 8. Failed units and reset-failed

```bash
systemctl --failed
systemctl status <unit-name>
journalctl -u <unit-name> -b
```

After resolving the underlying issue, a unit's failed state can be cleared:

```bash
sudo systemctl reset-failed <unit-name>
```

Resetting the recorded failed state is not a repair and does not fix the underlying cause. Investigate the journal and service configuration first. This operation normally requires root privileges.

## 9. Service troubleshooting workflow

1. Check the unit exists and inspect its state: `systemctl status <unit>`.
2. Read current-boot logs: `journalctl -b -u <unit>`.
3. Check unit details and overrides: `systemctl cat <unit>`.
4. Validate the application's own configuration using its documented check command.
5. Correct the underlying issue, then start or reload the service as appropriate.
6. Verify service state and application behavior; inspect logs again.

For SSH, firewall, storage, and network changes, maintain a recovery path in case the change disconnects the current session.

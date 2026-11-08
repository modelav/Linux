# Custom systemd service units

A custom systemd service unit lets systemd start, stop, supervise, and log an application. Use a system service for a server-side application that should run independently of an interactive login.


## Unit file location

Create local administrator units under:

```text
/etc/systemd/system/
```

A service unit filename ends in `.service`, for example:

```text
/etc/systemd/system/example-app.service
```

Do not edit package-supplied unit files under `/usr/lib/systemd/system/` or `/lib/systemd/system/`. Use a local unit or a drop-in override instead. 

## Create a service account

Run long-lived applications as a dedicated unprivileged account where possible:

```bash
sudo adduser --system --group --no-create-home --home /nonexistent --shell /usr/sbin/nologin example-app
```

Prepare the application and configuration directories with permissions appropriate to the application. Do not make configuration files or binaries writable by the service account unless the application genuinely needs that access.

## Example service unit

Create `/etc/systemd/system/example-app.service`:

```ini
[Unit]
Description=Example application service
Wants=network-online.target
After=network-online.target

[Service]
Type=exec
User=example-app
Group=example-app
WorkingDirectory=/srv/example-app
ExecStart=/usr/local/bin/example-app --config /etc/example-app/config.toml
Restart=on-failure
RestartSec=5s
StateDirectory=example-app
RuntimeDirectory=example-app
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

Important settings:

| Setting | Purpose |
|---|---|
| `Type=exec` | Consider the service started after the program has been executed successfully. |
| `User=` / `Group=` | Run the process with the specified account. |
| `WorkingDirectory=` | Set the process working directory. |
| `ExecStart=` | Absolute path and arguments for the program. systemd does not run this through a shell by default. |
| `Restart=on-failure` | Restart after abnormal exit, subject to systemd's restart limits. |
| `RestartSec=` | Wait before attempting a restart. |
| `StateDirectory=` | Create a persistent writable directory under `/var/lib/` for the service. |
| `RuntimeDirectory=` | Create a runtime directory under `/run/`, removed when the service stops. |
| `NoNewPrivileges=true` | Prevent the process and its children from gaining additional privileges through exec. |
| `PrivateTmp=true` | Give the service a private `/tmp` and `/var/tmp` view. |
| `ProtectSystem=strict` | Make most of the filesystem read-only for the service, while systemd-managed writable directories remain available. |
| `ProtectHome=true` | Hide user home directories from the service. |

`network-online.target` is only useful when the system has a suitable wait-online service configured; it does not guarantee that a remote application endpoint is reachable. Applications should still handle temporary network failures.

`StateDirectory=example-app` provides `/var/lib/example-app`; `RuntimeDirectory=example-app` provides `/run/example-app`. systemd creates and manages these directories for the configured service user.

Adapt the binary path, arguments, account, and writable directories to the application. Some sandboxing options can prevent an application from working if it needs access to other paths; add only the access it requires and test the service.

## Validate, load, and start

```bash
sudo systemd-analyze verify /etc/systemd/system/example-app.service
sudo systemctl daemon-reload
sudo systemctl enable --now example-app.service
systemctl status example-app.service
journalctl -u example-app.service -b
```

- `systemd-analyze verify` checks the unit file for errors and unknown directives.
- `daemon-reload` makes systemd re-read unit definitions.
- `enable --now` enables the service at boot and starts it immediately.
- `journalctl -u` shows the service's standard output, standard error, and journal messages.

## Common operations

```bash
sudo systemctl start example-app
sudo systemctl stop example-app
sudo systemctl restart example-app
sudo systemctl disable --now example-app
sudo systemctl edit example-app
systemctl cat example-app
```

Use `systemctl edit` to create a drop-in override instead of editing a packaged unit file. After manual unit-file changes, run `systemctl daemon-reload`. Restart or reload the application when required for the changed settings to take effect.

## Environment variables

For non-secret settings, a service can use an environment file:

```ini
[Service]
EnvironmentFile=-/etc/example-app/example-app.env
```

The leading `-` makes the file optional. Protect the file with appropriate ownership and permissions. 

## Troubleshooting

```bash
systemctl status example-app
journalctl -u example-app -b
systemd-analyze verify /etc/systemd/system/example-app.service
systemctl cat example-app
```

Common causes of failure include an incorrect executable path, missing file permissions, invalid arguments, a missing working directory, unavailable configuration, and sandboxing restrictions. Check the journal before repeatedly restarting the service.

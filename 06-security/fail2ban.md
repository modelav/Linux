# Fail2ban

Fail2ban monitors authentication and service logs for repeated failures. When a configured jail detects a pattern, it can temporarily block the source address using a firewall action.

Fail2ban is an additional control, not a replacement for SSH keys, strong authentication, updates, and a properly configured firewall.

## Install and check the service

```bash
sudo apt install fail2ban
systemctl status fail2ban
sudo fail2ban-client status
```

## Configuration files

| Path | Purpose |
|---|---|
| `/etc/fail2ban/fail2ban.conf` | Main daemon configuration. |
| `/etc/fail2ban/jail.conf` | Default jail settings; package-managed. |
| `/etc/fail2ban/jail.local` | Local jail overrides. |
| `/etc/fail2ban/jail.d/` | Additional jail configuration files. |
| `/etc/fail2ban/filter.d/` | Log-matching filters. |
| `/etc/fail2ban/action.d/` | Ban and unban actions. |

Do not edit `jail.conf` directly. Put local changes in a `.local` file so package upgrades do not overwrite them.

## Configure the SSH jail

Create `/etc/fail2ban/jail.d/sshd.local`:

```ini
[sshd]
enabled = true
port = ssh
maxretry = 5
findtime = 10m
bantime = 1h
```

| Setting | Meaning |
|---|---|
| `enabled` | Enable the jail. |
| `port` | Service port or port name to protect. `ssh` resolves according to the system's service definitions. |
| `maxretry` | Failed attempts allowed within the observation window. |
| `findtime` | Time window used to count failures. |
| `bantime` | How long a matching source address remains banned. |

The SSH jail uses the packaged filter. Log backend selection depends on the host. If using the systemd journal, use the `systemd` backend and ensure the required Python journal support (`python3-systemd`) is installed. If using a file backend, point it at a log file that actually exists, such as `/var/log/auth.log` when rsyslog is configured.



## Validate and reload configuration

```bash
sudo fail2ban-client -t
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

The The `-t` option tests configuration. Check the service logs if startup fails:

```bash
sudo journalctl -u fail2ban --since today
```

## Unban an address

```bash
sudo fail2ban-client set sshd unbanip 198.51.100.25
```

Replace the documentation address with the actual address to unban. Confirm the jail and address before running the command.

## Test a filter

`fail2ban-regex` tests whether a filter matches the log format:

```bash
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf
```

For a journal backend, test using the journal input syntax supported by the installed version. Consult `man fail2ban-regex` for details.

## Important considerations

- Add trusted administrator and management addresses to the configured ignore list where appropriate, and verify the syntax before restarting.
- Be careful when setting very short ban times or low retry counts; shared NAT addresses can cause legitimate users to be banned.
- Fail2ban actions depend on firewall tooling and backend compatibility. Avoid mixing multiple firewall managers without understanding their interaction.
- Confirm SSH remains reachable from a trusted second session after changing the jail or firewall action.

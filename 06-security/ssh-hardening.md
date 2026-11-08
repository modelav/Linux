# SSH hardening

Hardening should reduce unnecessary authentication paths without locking administrators out.

## Important files

| Path | Purpose |
|---|---|
| `/etc/ssh/sshd_config` | Main SSH server configuration. |
| `/etc/ssh/sshd_config.d/` | Configuration snippets included by Debian's server configuration. |
| `~/.ssh/authorized_keys` | Public keys allowed to log in to a user account. |
| `/etc/ssh/ssh_host_*` | Server host keys. |

Use the files under `/etc/ssh/sshd_config.d/` for local settings when the main configuration includes that directory. Check the actual include order and effective configuration on the host; 

## Check the SSH service

```bash
systemctl status ssh
sshd -t
sshd -T | less
```

- `sshd -t` checks configuration syntax and key sanity.
- `sshd -T` prints the effective configuration after processing configuration files and applicable options.
- `sshd -T -C user=alice,host=server.example.com,addr=192.0.2.20` evaluates settings for a connection context; replace the example values.

## Common hardening settings

Create a local drop-in, for example `/etc/ssh/sshd_config.d/10-hardening.conf`:

```sshconfig
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
MaxAuthTries 4
LoginGraceTime 30
X11Forwarding no
```

| Setting | Purpose |
|---|---|
| `PermitRootLogin no` | Disallow direct root login over SSH. |
| `PubkeyAuthentication yes` | Allow public-key authentication. |
| `PasswordAuthentication no` | Disable password-based SSH authentication. |
| `KbdInteractiveAuthentication no` | Disable keyboard-interactive authentication. Check whether your MFA setup depends on it before changing. |
| `MaxAuthTries 4` | Limit authentication attempts per connection. |
| `LoginGraceTime 30` | Limit the time allowed to authenticate. |
| `X11Forwarding no` | Disable X11 forwarding on a headless server when it is not needed. |


## Apply changes safely

Keep the current SSH session open. Verify that at least one administrator has a working key and sudo access, then:

```bash
sudo sshd -t
sudo systemctl reload ssh
```

Open a **second** terminal and test a new login before closing the original session. If the new connection fails, use the existing session or provider console to correct the configuration.


## Restrict users and groups

Examples:

```sshconfig
AllowGroups ssh-admins
```

Or:

```sshconfig
AllowUsers alice bob
```

Use one deliberate allow-listing strategy and verify that all required administrator accounts are included. `AllowGroups` checks supplementary group membership, so confirm it with `id <user>`.

## Inspect authentication logs

```bash
sudo journalctl -xeu -n 50 ssh 
sudo journalctl -u ssh -p warning
```

If rsyslog is installed and configured, authentication events may also be written to `/var/log/auth.log`.

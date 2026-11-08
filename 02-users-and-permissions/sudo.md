# sudo

`sudo` lets an authorized user run a command with another user's privileges, commonly as `root`. It is preferable to using a permanent root shell for routine administration.

## Run commands with sudo

| Command / option | Meaning |
|---|---|
| `sudo <command>` | Run a command as root by default. |
| `sudo -u <user> <command>` | Run a command as another user. |
| `sudo -i` | Start a login shell as the target user, root by default. |
| `sudo -s` | Start a shell as the target user; it is not a login shell. |
| `sudo -l` | List the commands the current user may run with sudo. |
| `sudo -k` | Invalidate cached authentication credentials. |
| `sudo -v` | Refresh the authentication timestamp. |
| `sudoedit <file>` | Edit a protected file using a safer temporary-copy workflow. |

Use `sudo <command>` for one-off administration where possible. Be careful when working in a root shell: every command in that shell runs with elevated privileges unless you explicitly change user.

## Add a user to the sudo group

On a Debian system configured to grant administrative privileges to the `sudo` group:

```bash
sudo adduser alice sudo
```

Or:

```bash
sudo usermod -aG sudo alice
```

The user normally needs to start a new login session before the new group membership applies.

The `sudo` package and the system's sudoers policy must be configured for this group. Do not assume every Debian installation has the same policy.

## Configure sudoers safely

The main configuration file is `/etc/sudoers`. Debian installations commonly also include files from `/etc/sudoers.d/`.

Never edit `/etc/sudoers` with a normal text editor. Use `visudo`, which locks the file while editing and checks the syntax before saving.

```bash
sudo visudo
```

To create a separate policy file:

```bash
sudo visudo -f /etc/sudoers.d/ops-team
```

Example rule granting full sudo privileges to a group:

```sudoers
%ops-team ALL=(ALL:ALL) ALL
```

This grants broad administrative access. Grant only the privileges the group actually needs.

Validate the configuration:

```bash
sudo visudo -c
```

## Important sudoers syntax

A rule generally follows this form:

```text
USER HOST=(RUNAS) COMMANDS
```

| Part | Meaning |
|---|---|
| `USER` | User or `%group` receiving the permission. |
| `HOST` | Host or host pattern where the rule applies. |
| `RUNAS` | User and optionally group the command may run as. |
| `COMMANDS` | Commands the user may run. |

Avoid broad `NOPASSWD: ALL` rules unless there is a carefully reviewed operational reason. A narrowly scoped command rule is usually safer.

## Troubleshooting

- `sudo -l` shows the current user's sudo permissions.
- `id <user>` verifies group membership.
- `visudo -c` checks sudoers syntax.
- Check that `/etc/sudoers.d/` files are readable by sudo and have appropriate ownership and permissions.
- After changing group membership, start a new login session.


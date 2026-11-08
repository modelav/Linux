# User account management

Manage local user accounts with Debian's `adduser` and `deluser` tools, and use `passwd` and `chage` for password status and expiry information.

## Create a user

### `adduser`

Debian's `adduser` is the convenient interactive tool for creating a regular user. It creates the account, home directory, and initial group according to the system's Debian configuration.

```bash
sudo adduser alice
```

Useful options:

| Option | Meaning |
|---|---|
| `--home <directory>` | Set the user's home directory. |
| `--shell <path>` | Set the user's login shell. |
| `--ingroup <group>` | Use an existing group as the user's initial group. |
| `--disabled-password` | Create the account without setting a usable password. Other authentication methods may still be possible. |
| `--system` | Create a system account rather than a regular user account. |
| `--no-create-home` | Do not create a home directory. 

Example:

```bash
sudo adduser --home /srv/alice --shell /bin/bash alice
sudo adduser --system --group --no-create-home --home /nonexistent --shell /usr/sbin/nologin app-service
```

### `useradd`

`useradd` is the lower-level account-creation utility. It is useful in scripts and advanced account provisioning, but it does not apply all of Debian's `adduser` front-end behavior.

```bash
sudo useradd --create-home --shell /bin/bash alice
```

| Option | Meaning |
|---|---|
| `-m`, `--create-home` | Create the user's home directory. |
| `-d`, `--home-dir <path>` | Set the home directory path. |
| `-s`, `--shell <path>` | Set the login shell. |
| `-g <group>` | Set the primary group. |
| `-G <group-list>` | Set supplementary groups. |
| `-r`, `--system` | Create a system account. |

## Modify an account

### `usermod`

Change account properties.

| Option | Meaning |
|---|---|
| `-aG <groups>` | Append the user to supplementary groups. |
| `-g <group>` | Change the primary group. |
| `-d <directory>` | Change the home directory path. |
| `-m` | Move the existing home directory contents when used with `-d`. |
| `-s <shell>` | Change the login shell. |
| `-L` | Lock the password. This does not necessarily disable other authentication methods such as SSH keys. |
| `-U` | Unlock the password. |
| `-e <date>` | Set the account expiration date; use `YYYY-MM-DD`. |
Examples:

```bash
sudo usermod -aG developers alice
sudo usermod -s /bin/bash alice
sudo usermod -d /home/alice-new -m alice
```

Be careful with `-G`: without `-a`, it replaces the supplementary group list.

## Password management

### `passwd`

Change a password or inspect its status.

```bash
passwd                   # Change your own password
sudo passwd alice        # Set another user's password
sudo passwd -S alice     # Show password status
```

| Option | Meaning |
|---|---|
| `-S` | Show password status. |
| `-l` | Lock password authentication. This does not disable SSH key authentication. |
| `-u` | Unlock a previously locked password. |
| `-e` | Expire the password and require a change at the next password-based login. |

Do not use `passwd -d` as a shortcut for disabling an account; it removes the password and can create an unintended passwordless account state.

### `chage`

View or change password-aging information.

```bash
sudo chage -l alice
sudo chage -M 90 -W 14 alice
sudo chage -d 0 alice
```

| Option | Meaning |
|---|---|
| `-l` | List password-aging information. |
| `-M <days>` | Set maximum password age. |
| `-m <days>` | Set minimum days between password changes. |
| `-W <days>` | Set warning days before password expiry. |
| `-I <days>` | Set the inactive period after password expiry. |
| `-E <date>` | Set account expiration date. |
| `-d 0` | Require a password change at the next password-based login. |


## Remove a user

### `deluser`

Remove a regular user account:

```bash
sudo deluser alice
```

Remove the account and its home directory:

```bash
sudo deluser --remove-home alice
```

`--remove-home` removes the home directory and mail spool. It does not find and remove every file owned by that user elsewhere on the system.

**Warning:** `--remove-all-files` can delete the user's files across the system. Inventory the account's files and confirm backups before using it.

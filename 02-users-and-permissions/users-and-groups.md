# Users and groups

Linux uses user and group accounts to identify processes and control access to files and services. This guide covers how to inspect users and manage groups on Debian.

## Important files

| Path | Purpose |
|---|---|
| `/etc/passwd` | User account names, IDs, home directories, and login shells. |
| `/etc/shadow` | Password hashes and password-aging data. Access is restricted. |
| `/etc/group` | Group names, IDs, and supplementary group members. |
| `/etc/gshadow` | Secure group information. |
| `/etc/adduser.conf` | Defaults used by Debian's `adduser` and `addgroup` tools. |

Do not edit `/etc/passwd`, `/etc/shadow`, `/etc/group`, or `/etc/gshadow` directly. Use the account-management commands.

## View users and groups

### `id`

Show a user's UID, primary GID, and supplementary groups.

```bash
id
id alice
id -u alice       # UID only
id -g alice       # primary GID only
id -G alice       # all group IDs
```

### `getent`

Query the system's account databases. Unlike reading `/etc/passwd` directly, `getent` can also query accounts provided by configured sources such as LDAP.

```bash
getent passwd alice
getent group sudo
getent passwd
getent group
```

### `groups`

Show group membership for the current user or another account.

```bash
groups
groups alice
```

## Create groups

Debian provides `addgroup` as a convenient front end for group creation.

```bash
sudo addgroup developers
sudo addgroup --system app-service
```

`--system` creates a system group using the system account ID range configured on the host.

For lower-level administration, `groupadd` is also available:

```bash
sudo groupadd developers
sudo groupadd --system app-service
```

## Add and remove group members

### `adduser`

Add an existing user to a supplementary group:

```bash
sudo adduser alice developers
```

### `gpasswd`

| Option | Meaning |
|---|---|
| `-a <user>` | Add a user to a group. |
| `-d <user>` | Remove a user from a group. |

```bash
sudo gpasswd -a alice developers
sudo gpasswd -d alice developers
```

### `usermod`

Add a user to one or more supplementary groups:

```bash
sudo usermod -aG developers,docker alice
```

Do not omit `-a` when adding supplementary groups with `usermod -G`. Without `-a`, the listed groups replace the user's existing supplementary group list.

Group changes normally take effect in new login sessions. The user may need to log out and back in.

## Remove groups

```bash
sudo delgroup developers
sudo groupdel developers
```

A group cannot normally be removed while it is the primary group of an existing user. Check membership and file ownership before deleting a group; files may retain the old numeric group ID.

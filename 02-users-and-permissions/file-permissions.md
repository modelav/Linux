# File permissions and ownership

Linux file permissions control what the file owner, the owning group, and other users can do. This guide covers `rwx` permissions, ownership, special permission bits, and the default permissions used when files are created.

## Read permissions

Use `ls -l` or `stat` to inspect permissions and ownership:

```bash
ls -l /etc/ssh/sshd_config
stat /etc/ssh/sshd_config
```

A permission string such as `-rwxr-x---` is read in three groups after the file-type character:

| Class | Applies to |
|---|---|
| User (`u`) | The file owner. |
| Group (`g`) | Members of the file's owning group. |
| Other (`o`) | Everyone else. |

| Permission | Regular file | Directory |
|---|---|---|
| `r` | Read file contents. | List directory entries. |
| `w` | Modify file contents. | Create, remove, or rename entries (subject to other checks). |
| `x` | Execute the file. | Search/traverse the directory. |

For a directory, execute (`x`) is required to access entries by name. Read (`r`) without execute generally does not let a user access individual entries.

## Numeric permissions

Each permission has a numeric value:

| Permission | Value |
|---|---:|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |

Add the values for each class. For example, `750` means owner `rwx` (7), group `r-x` (5), and other `---` (0).

## Change permissions with `chmod`

```bash
chmod 640 app.conf
chmod 750 /srv/app
chmod u+x deploy.sh
chmod g-w shared.txt
chmod o-rwx private-data
```

Useful options:

| Option | Meaning |
|---|---|
| `-R` | Apply changes recursively to a directory tree. |
| `-v` | Show each processed file. |
| `-c` | Show only when a change is made. |
| `--reference=<file>` | Copy the mode from another file. |

Examples:

```bash
chmod --reference=known-good.conf app.conf
chmod -R u=rwX,g=rX,o= /srv/app
```

**Warning:** Recursive permission changes can break services or expose private data. Inspect the target tree first.

## Change ownership

### `chown`

Change the owner, group, or both.

```bash
sudo chown alice app.conf
sudo chown alice:developers app.conf
sudo chown :developers shared-directory
sudo chown -R app:app /srv/app
```

| Option | Meaning |
|---|---|
| `-R` | Change ownership recursively. |
| `-h` | Change a symbolic link itself rather than its target, where supported. |
| `-v` | Show each processed file. |

Use recursive ownership changes only after checking the exact path and intended owner/group. If a file has an extended ACL, changing its mode with `chmod` can also change the ACL mask and reduce the effective permissions of named users or groups; inspect it with `getfacl`.

### `chgrp`

Change only the group owner:

```bash
sudo chgrp developers shared-directory
sudo chgrp -R developers /srv/shared
```

## Special permission bits

The leading numeric digit in a mode can set special bits:

| Value | Name | Purpose |
|---:|---|---|
| `4` | Set-user-ID (setuid) | On executable files, run with the file owner's effective UID, subject to system rules. |
| `2` | Set-group-ID (setgid) | On executables, run with the file's group ID; on directories, new entries inherit the directory's group. |
| `1` | Sticky bit | On directories, restricts who can remove or rename entries, commonly used on shared temporary directories. |

A common shared-directory mode is `2775`:

```bash
sudo chown root:developers /srv/shared
sudo chmod 2775 /srv/shared
```

New entries in the directory inherit the `developers` group. Actual permissions still depend on the creating program's requested mode and umask.

Be very cautious when setting setuid or setgid on executable files. These bits can change the privileges under which a program runs.

## Default permissions with `umask`

`umask` removes permission bits from the mode requested when a new file or directory is created.

```bash
umask
umask -S
umask 027
```

A common `umask` of `027` results in new regular files typically being `640` and new directories typically being `750`, assuming the application requests the usual `666` and `777` base modes. An application can request more restrictive permissions.


## Create directories with specific ownership and permissions

`install` is useful for creating application directories with the intended owner, group, and mode in one command:

```bash
sudo install -d -o app -g app -m 0750 /srv/app
```

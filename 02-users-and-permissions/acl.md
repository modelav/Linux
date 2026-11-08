# Access Control Lists (ACLs)

Access Control Lists (ACLs) extend traditional Linux permissions. They let you grant access to additional named users or groups without changing the file's owning user or group.

The `getfacl` and `setfacl` commands are provided by the Debian `acl` package.

## Check ACLs

```bash
getfacl file.txt
getfacl /srv/shared
getfacl -R /srv/shared
```

`getfacl` shows the owner, owning group, basic permissions, and any extended or default ACL entries.

## Add access for a user or group

### `setfacl -m`

Modify an existing ACL entry or add a new one.

```bash
setfacl -m u:alice:rw file.txt
setfacl -m g:developers:rwx /srv/shared
```

Common permission values are `r`, `w`, and `x`.

| ACL entry | Meaning |
|---|---|
| `u:alice:rw` | Give user `alice` read and write permissions. |
| `g:developers:rwx` | Give group `developers` read, write, and execute/search permissions. |
| `m::rwx` | Set the ACL mask, which limits the effective permissions of named users, named groups, and the owning group entry. |

The ACL mask can make effective permissions narrower than the permissions written on an individual ACL entry. `getfacl` displays effective permissions when they differ.

## Default ACLs for new files and directories

A default ACL can be set on a directory. New entries created inside that directory inherit ACL entries from it, subject to the mode requested by the creating application.

```bash
setfacl -d -m u:alice:rwx /srv/shared
setfacl -d -m g:developers:rwx /srv/shared
getfacl /srv/shared
```

Default ACLs apply to directories; regular files do not have default ACLs.

## Remove ACL entries

```bash
setfacl -x u:alice file.txt
setfacl -x g:developers /srv/shared
```

Useful options:

| Option | Meaning |
|---|---|
| `-m` | Modify or add ACL entries. |
| `-x` | Remove specified ACL entries. |
| `-d` | Work with the directory's default ACL. |
| `-b` | Remove all extended ACL entries, retaining the basic owner/group/other entries. |
| `-k` | Remove the default ACL from a directory. |
| `-R` | Apply recursively. |

To remove all extended ACL entries from a file:

```bash
setfacl -b file.txt
```

To remove the default ACL from a directory:

```bash
setfacl -k /srv/shared
```

**Warning:** Recursive ACL changes affect every matching object below the target. 

## Save and restore ACLs

Save ACL information before a large change:

```bash
getfacl -R /srv/shared > shared-acls.backup
```

Restore it later:

```bash
sudo setfacl --restore=shared-acls.backup
```

The restore file contains path and ACL information. Keep it protected and restore it from the expected directory context.

## Troubleshooting

- Check the ACL with `getfacl <path>`.
- Look for an `effective:` comment; the ACL mask may limit access.
- Check every parent directory: the user needs search (`x`) permission on each directory in the path.
- Confirm the mounted filesystem supports ACLs.
- Check whether the application created the file with a restrictive mode that limits inherited default ACL permissions.


# Archive and compression

Archives combine files and directories into one file. Compression reduces storage or transfer size. On Debian, `tar` with gzip, xz, or zstd is common for backups and file transfer.

## tar basics

### Create an archive

```bash
# Create an uncompressed tar archive
tar -cvf config.tar -C / etc/example

# Create a gzip-compressed archive
tar -czvf config.tar.gz -C / etc/example

# Create an xz-compressed archive
tar -cJvf config.tar.xz -C / etc/example
```

| Option | Meaning |
|---|---|
| `-c` | Create an archive. |
| `-x` | Extract an archive. |
| `-t` | List archive contents. |
| `-v` | Verbose output. |
| `-f <file>` | Use the following path as the archive file. |
| `-z` | Filter the archive through gzip. |
| `-J` | Filter the archive through xz. |
| `-I <program>` | Use a specified compression program, such as `zstd`. |
| `-C <dir>` | Change to a directory before processing following paths. |

For zstd compression, if the `zstd` program is installed:

```bash
tar --zstd -cvf app-data.tar.zst -C /srv app
```

Use `-C` to control path names in the archive. This avoids accidentally storing unexpected absolute or deeply nested paths.

## List and inspect an archive

```bash
tar -tf config.tar.gz
 tar -tvf config.tar.gz | less
```

The first command lists member names. The verbose form shows permissions, ownership, sizes, and timestamps.

## Extract an archive

```bash
# Extract into the current directory
 tar -xvf config.tar

# Extract into a chosen directory
mkdir -p /tmp/restore-check
tar -xvzf config.tar.gz -C /tmp/restore-check
```

Inspect unfamiliar archives before extracting them, especially when extracting as root. Archive members can contain paths and file types that overwrite existing files or place files outside the intended location if handled carelessly.

## Preserve metadata

For system configuration or service data, file contents alone may not be enough. GNU tar supports options such as:

| Option | Meaning |
|---|---|
| `--acls` | Store POSIX ACL information. |
| `--xattrs` | Store extended attributes. |
| `--numeric-owner` | Store numeric UID/GID values instead of resolving account names. |
| `--one-file-system` | Do not cross filesystem boundaries while walking directories. |
| `--same-owner` | When extracting as root, restore stored ownership where possible. |

Example for a system configuration archive:

```bash
sudo tar --acls --xattrs --numeric-owner -czpf etc-backup.tar.gz -C / etc
```

The `-p` option preserves permissions when extracting; ownership restoration also depends on privilege and extraction options.
## Exclude files and directories

```bash
tar -czvf app-backup.tar.gz \
  --exclude='app/cache' \
  --exclude='app/tmp' \
  -C /srv app
```

Check exclude patterns carefully. Excluding a path required for application recovery can make the archive incomplete.

## gzip and xz commands

| Command | Purpose |
|---|---|
| `gzip file` | Compress a file to `file.gz`, normally removing the original after success. |
| `gzip -k file` | Keep the original file. |
| `gzip -d file.gz` | Decompress a gzip file. |
| `xz file` | Compress using xz. |
| `xz -T0 file` | Use available CPU threads. |
| `xz -d file.xz` | Decompress an xz file. |

`gzip` and `xz` compress individual streams/files. They do not preserve a directory tree by themselves; use tar when multiple files and their paths must be collected into one archive.

## Verify an archive

```bash
gzip -t config.tar.gz
xz -t config.tar.xz
tar -tzf config.tar.gz >/dev/null
sha256sum config.tar.gz > config.tar.gz.sha256
sha256sum -c config.tar.gz.sha256
```

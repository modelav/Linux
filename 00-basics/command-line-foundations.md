# Command-line foundations

The command line is the main way to inspect and administer a Debian server. This guide covers the commands used most often to navigate directories and work with files.

## Command structure

A typical command has this form:

```text
command [options] [arguments]
```

- **Command:** the program or shell builtin to run.
- **Option:** changes the command's behavior, such as `-l` or `--all`.
- **Argument:** the object the command acts on, such as a path, username, or service name.

Example:

```bash
ls -lah /var/log
```

Use `--` to mark the end of options when a filename might begin with a hyphen:

```bash
ls -- -old-file
rm -- ./-old-file
```

## Paths and navigation

- An **absolute path** starts with `/`, for example `/etc/ssh/sshd_config`.
- A **relative path** starts from the current directory, for example `logs/app.log`.
- `.` means the current directory; `..` means its parent.
- `~` expands to the current user's home directory in Bash.
- Linux paths are case-sensitive: `Backup` and `backup` are different names.

| Command | Purpose | Useful options |
|---|---|---|
| `pwd` | Print the current directory. | `-P` shows the physical path after resolving symbolic links. |
| `cd <path>` | Change directory. | `cd -` returns to the previous directory; `cd` goes home. |
| `ls` | List directory entries. | See the options below. |

### Common `ls` options

| Option | Meaning |
|---|---|
| `-l` | Long format: permissions, owner, group, size and timestamp. |
| `-a` | Include hidden entries beginning with `.`. |
| `-A` | Include hidden entries but omit `.` and `..`. |
| `-h` | Human-readable sizes when used with a size listing such as `-l`. |
| `-d` | List the directory itself instead of its contents. |
| `-t` | Sort by modification time, newest first. |
| `-S` | Sort by file size, largest first. |
| `-R` | Recursively list subdirectories. |

```bash
pwd
ls -lah
ls -ld /etc /var/log
ls -lt /var/log
```

## Create, copy, move and remove

### `mkdir` and `touch`

```bash
mkdir reports
mkdir -p /srv/app/logs/archive
touch notes.txt
```

| Option | Command | Meaning |
|---|---|---|
| `-p` | `mkdir` | Create missing parent directories and do not fail if the target already exists. |
| `-m <mode>` | `mkdir` | Set permissions when creating the directory. |
| `-v` | `mkdir` | Print each created directory. |

`touch` creates an empty file if it does not exist; otherwise it updates timestamps.

### `cp` — copy

| Option | Meaning |
|---|---|
| `-a` | Archive mode: copy recursively and preserve common attributes and symbolic links. |
| `-r` | Copy directories recursively. |
| `-i` | Ask before overwriting. |
| `-n` | Do not overwrite existing files. |
| `-v` | Show what is copied. |

```bash
cp -i app.conf app.conf.bak
cp -a /srv/app/ /srv/app-backup/
```

### `mv` — move or rename

| Option | Meaning |
|---|---|
| `-i` | Ask before overwriting. |
| `-n` | Do not overwrite an existing destination. |
| `-v` | Show each move. |

```bash
mv -i old-name.conf new-name.conf
mv report.txt /srv/reports/
```

### `rm` and `rmdir` — remove

```bash
rm -i old-file.txt
rmdir empty-directory
```

`rmdir` removes empty directories. `rm` removes directory entries and normally does not move files to a trash bin.

**Warning:** Check the exact path before using `rm -r`. Recursive deletion can remove large directory trees permanently. Avoid copying commands that combine `-r` and `-f` without understanding every path and variable.

## Create symbolic links

`ln -s` creates a symbolic link that points to another path:

```bash
ln -s /var/log/myapp /srv/myapp-logs
readlink -f /srv/myapp-logs
```

Use an absolute target when the link should work regardless of the current directory. A symbolic link can point to a path that does not currently exist.

## Inspect files

| Command | Purpose |
|---|---|
| `file <path>` | Identify a file type from its contents. |
| `stat <path>` | Show metadata such as size, ownership, permissions and timestamps. |
| `readlink <path>` | Display a symbolic link's target. |
| `readlink -f <path>` | Resolve a path through symbolic links where possible. |

```bash
file /etc/hostname
stat /etc/hostname
readlink -f /bin/sh
```

## Read text files

| Command | Purpose | Useful options |
|---|---|---|
| `cat <file>` | Print file contents. | Best for short files. |
| `less <file>` | Read a file interactively. | Press `q` to quit; `/word` searches. |
| `head <file>` | Print the first lines. | `-n 20` prints 20 lines. |
| `tail <file>` | Print the last lines. | `-n 100` prints 100 lines; `-f` follows appended output. |
| `wc <file>` | Count lines, words and bytes. | `-l`, `-w`, `-c` count lines, words and bytes. |

```bash
less /etc/ssh/sshd_config
head -n 20 /etc/passwd
tail -n 50 /var/log/syslog
tail -f /var/log/syslog
wc -l /etc/passwd
```

Some minimal installations do not have `/var/log/syslog`; logs may be available through `journalctl` instead.

## Find files

`find` searches a directory tree by name, type, time, size and other attributes.

| Expression | Meaning |
|---|---|
| `-type f` | Regular files. |
| `-type d` | Directories. |
| `-name '<pattern>'` | Match a filename pattern. Quote the pattern. |
| `-iname '<pattern>'` | Case-insensitive filename match. |
| `-maxdepth <n>` | Do not descend beyond the given depth. |
| `-mtime -<n>` | Modified less than approximately `<n>` whole days ago. |

```bash
find /var/log -type f -name '*.log' -print
find /etc -maxdepth 2 -type f -iname '*ssh*' -print
find /srv/app -type f -mtime -1 -print
```

Start with `-print` to inspect matches before using actions such as `-delete` or `-exec` that change files.


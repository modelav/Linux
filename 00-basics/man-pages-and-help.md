# Man pages and help

Linux installs manual pages for many commands, services, system calls and configuration formats. Checking the local documentation is often the quickest way to confirm an option or understand a command's behavior.

## `man` — open a manual page

```bash
man <COMMAND>
```

Manual pages are grouped into sections:

| Section | Contents | Example |
|---:|---|---|
| `1` | User commands | `man 1 ls` |
| `2` | System calls | `man 2 open` |
| `3` | Library functions | `man 3 printf` |
| `4` | Special files and device interfaces | `man 4 null` |
| `5` | File formats and configuration files | `man 5 fstab` |
| `6` | Games and demonstrations | `man 6` |
| `7` | Conventions and overviews | `man 7 regex` |
| `8` | System administration commands | `man 8 useradd` |

If a name has more than one manual page, specify the section number.

### Useful `man` options

| Option | Meaning |
|---|---|
| `-k <keyword>` | Search manual-page names and short descriptions; similar to `apropos`. |
| `-f <name>` | Show a short description; similar to `whatis`. |
| `-a <name>` | Display all matching manual pages. |
| `-w <name>` | Print the path of the manual page. |

```bash
man -k filesystem
man -f systemctl
man -a printf
man -w fstab
```

## Search by keyword

### `apropos`

```bash
apropos network
apropos 'user account'
```

### `whatis`

```bash
whatis rsync
whatis systemctl
```

These tools search the manual-page description database. If results are missing, the database may not be installed or updated.

## Ask a command for help

Many programs provide a short usage summary:

```bash
ls --help
find --help
systemctl --help
```

`--help` is common but not universal. Some programs use `-h` differently or have no help option.

## Find how the shell resolves a command

A command name may refer to a shell builtin, alias, function or executable.

```bash
type cd
type -a ls
command -v systemctl
```

- `type` explains how the current shell interprets a name.
- `type -a` shows all matching resolutions.
- `command -v` reports how the shell resolves a command; it is useful in scripts.

Use `help` for Bash builtins:

```bash
help cd
help printf
help test
```

## GNU Info

Some GNU tools provide longer documentation in the Info format:

```bash
info coreutils
info bash
info sed
```

## Navigate a manual page

Manual pages commonly open in `less`.

| Key | Action |
|---|---|
| `q` | Quit. |
| `/word` | Search forward. |
| `?word` | Search backward. |
| `n` | Repeat the search in the same direction. |
| `N` | Repeat in the opposite direction. |
| `g` | Go to the beginning. |
| `G` | Go to the end. |

## When documentation is missing

Minimal Debian installations may not include every manual page. Check whether the `man` command is available and whether the relevant package is installed:

```bash
command -v man
man -w systemctl
apt show man-db manpages
```

`man-db` provides the manual-page system. The `manpages` package supplies many general manual pages; other packages provide their own documentation.

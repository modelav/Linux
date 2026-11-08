# Shell basics

A shell reads commands, expands variables and patterns, handles redirection, and starts programs. Bash is commonly used for interactive administration and scripting on Debian, but not every script or service uses Bash.

## Identify the shell

```bash
printf 'Login shell: %s\n' "$SHELL"
ps -p "$$" -o comm=
```

`$SHELL` usually contains the configured login shell; `ps` shows the current shell process. A script's shebang, such as `#!/bin/bash` or `#!/bin/sh`, specifies the interpreter used when it is executed directly. On typical Debian installations, `/bin/sh` points to `dash`, not Bash; check with `readlink -f /bin/sh` before using Bash-only syntax in a script that declares `#!/bin/sh`.

## Quoting

Quoting prevents the shell from splitting paths or expanding characters unexpectedly.

| Form | Behavior | Example |
|---|---|---|
| `'single quotes'` | Preserves all characters literally. | `printf '%s\n' '$HOME'` |
| `"double quotes"` | Allows variable and command substitution, but protects spaces and most special characters. | `printf '%s\n' "$HOME"` |
| `\` | Escapes the next special character. | `mkdir My\ Reports` |

Quote variable expansions and paths in scripts:

```bash
backup_dir='/srv/backup files'
mkdir -p -- "$backup_dir"
printf 'Directory: %s\n' "$backup_dir"
```

## Wildcards and expansion

Bash expands unquoted patterns before running the command.

| Pattern | Meaning |
|---|---|
| `*` | Zero or more characters in a filename component. |
| `?` | Exactly one character. |
| `[abc]` | One character from the set. |
| `[0-9]` | One character in the range. |

```bash
ls -- *.conf
printf '%s\n' /var/log/*.log
find /var/log -type f -name '*.log' -print
```

Quote patterns when another program should interpret them. For example, `find -name '*.log'` needs the quotes so Bash does not expand the pattern first.

## Variables and environment

Assign a shell variable without spaces around `=`:

```bash
app_dir='/srv/myapp'
printf '%s\n' "$app_dir"
```

Export a variable so child processes receive it:

```bash
export app_dir
```

Set a variable for one command only:

```bash
LANG=C date
```

Common environment variables:

| Variable | Typical meaning |
|---|---|
| `HOME` | Current user's home directory. |
| `PATH` | Directories searched for commands. |
| `USER` | Login name in many environments. |
| `LANG` | Default locale. |
| `EDITOR` | Preferred text editor for many programs. |
| `PWD` | Shell's logical current directory. |

Do not use environment variables as a secure secret store; they can be exposed through process inspection or diagnostics.

## Command substitution and arithmetic

Use `$(...)` to capture command output:

```bash
kernel_release=$(uname -r)
printf 'Kernel: %s\n' "$kernel_release"
```

Use `$((...))` for integer arithmetic in Bash:

```bash
count=3
next_count=$((count + 1))
printf '%s\n' "$next_count"
```

## Standard input, output and error

| Stream | File descriptor | Default |
|---|---:|---|
| Standard input (`stdin`) | `0` | Terminal input |
| Standard output (`stdout`) | `1` | Terminal output |
| Standard error (`stderr`) | `2` | Terminal output |

| Syntax | Meaning |
|---|---|
| `command > file` | Write stdout to a file, replacing its previous contents. |
| `command >> file` | Append stdout to a file. |
| `command < file` | Read stdin from a file. |
| `command 2> file` | Write stderr to a file. |
| `command > file 2>&1` | Write stdout and stderr to the same file. |
| `command 2>/dev/null` | Discard stderr. |

Redirections are processed from left to right, so order matters.

```bash
find /etc -type f -name '*.conf' >config-files.txt
find /etc -type f -name '*.conf' >find.log 2>&1
```

A redirection is opened by the current shell before `sudo` runs the command. This is why `sudo echo text > /etc/file` usually fails for a protected file. For a simple replacement, use `sudo tee`:

```bash
printf '%s\n' 'example' | sudo tee /etc/example.conf >/dev/null
```

This overwrites the destination file; back it up and verify the path first.

## Pipes and command chaining

A pipe sends stdout from one command to stdin of the next. Stderr is not included unless redirected.

```bash
journalctl -b | less
ps aux | grep '[s]shd'
grep -i 'error' app.log | tail -n 20
```

| Operator | Meaning |
|---|---|
| `first && second` | Run `second` only if `first` succeeds. |
| `first || second` | Run `second` only if `first` fails. |
| `first ; second` | Run `second` regardless of the first command's status. |

In Bash, `set -o pipefail` makes a pipeline return a failure status if a command in the pipeline fails, instead of reporting only the last command's status.

## Exit status

A command's exit status is an integer. `0` conventionally means success; non-zero means an error or another condition.

```bash
some-command
printf 'Exit status: %s\n' "$?"
```

Read `$?` immediately because the next command replaces it. Prefer a direct condition in scripts:

```bash
if systemctl is-active --quiet ssh; then
    printf '%s\n' 'SSH is active.'
else
    printf '%s\n' 'SSH is not active.'
fi
```

## Job control shortcuts

| Key or command | Purpose |
|---|---|
| `Ctrl+C` | Interrupt the foreground command. |
| `Ctrl+Z` | Suspend the foreground job. |
| `jobs` | List jobs started by the current shell. |
| `bg` | Resume a suspended job in the background. |
| `fg` | Bring a job to the foreground. |
| `Ctrl+D` | Send end-of-file; often exits an interactive shell on an empty line. |

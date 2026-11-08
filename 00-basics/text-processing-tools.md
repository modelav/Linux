# Text processing tools

Text-processing commands are useful for searching logs, extracting fields from command output, comparing files, and building small command pipelines. Most of these tools read from standard input or files and write results to standard output.

## Search text with `grep`

`grep` prints lines that match a pattern.

| Option | Meaning |
|---|---|
| `-i` | Ignore case. |
| `-n` | Include line numbers. |
| `-v` | Print non-matching lines. |
| `-r` | Search directories recursively. |
| `-l` | Print only names of files with matches. |
| `-E` | Use extended regular expressions. |
| `-F` | Treat the pattern as fixed text, not a regular expression. |

```bash
grep -n 'PermitRootLogin' /etc/ssh/sshd_config
grep -i 'failed password' /var/log/auth.log
grep -RIn -- 'listen' /etc/nginx/
grep -E 'error|warning' app.log
grep -F 'literal [text]' file.txt
```

Quote patterns so the shell does not expand special characters before `grep` receives them. Use `-F` when searching for text that contains regular-expression characters.

## Transform text with `sed`

`sed` is a stream editor. It reads input and applies editing commands, commonly substitutions.

| Option | Meaning |
|---|---|
| `-n` | Disable automatic printing; combine with `p` to print selected lines. |
| `-E` | Use extended regular expressions. |
| `-e <script>` | Provide an editing command. |
| `-i[.backup]` | Edit a file in place; adding a suffix creates a backup. |

```bash
# Print lines 1 through 20
sed -n '1,20p' file.txt

# Replace the first match on each line in the output
sed 's/old/new/' file.txt

# Replace all matches on each line in the output
sed 's/old/new/g' file.txt

# Edit the file and keep the original as file.txt.bak
sed -i.bak 's/old/new/g' file.txt
```

Check the output without `-i` before editing a configuration file in place. `sed -i` changes the file directly.

## Extract fields with `awk`

`awk` processes text records and fields. By default, fields are separated by whitespace.

| Expression | Meaning |
|---|---|
| `$0` | Entire current line. |
| `$1`, `$2` | First and second fields. |
| `$NF` | Last field. |
| `-F <separator>` | Set the input field separator. |
| `NR` | Current record (line) number. |
| `BEGIN { ... }` | Run before input is read. |
| `END { ... }` | Run after input is processed. |

```bash
awk '{print $1}' access.log
awk '{print $1, $NF}' access.log
awk -F: '{print $1}' /etc/passwd
awk -F: '$7 ~ /nologin$/ {print $1}' /etc/passwd
```

The `awk` command is an implementation-independent name. Debian's default implementation may differ from GNU awk (`gawk`); use portable awk syntax unless you explicitly install and target GNU awk.

## Select columns with `cut`

`cut` extracts bytes, characters, or delimiter-separated fields.

| Option | Meaning |
|---|---|
| `-d <char>` | Set the field delimiter. |
| `-f <list>` | Select fields. |
| `-c <list>` | Select character positions. |
|

```bash
cut -d: -f1 /etc/passwd
cut -d: -f1,3 /etc/passwd
cut -c1-20 file.txt
```

`cut` is suitable for simple delimiter-separated data. It is not a full CSV parser; quoted CSV fields can contain delimiters.

## Sort and remove duplicates

### `sort`

| Option | Meaning |
|---|---|
| `-n` | Numeric sort. |
| `-h` | Human-readable number sort (such as `2K`, `1G`). |
| `-r` | Reverse sort order. |
| `-u` | Sort and output unique lines. |
| `-k <field>` | Sort by a key/field. |
| `-t <char>` | Set the field delimiter. |

```bash
sort names.txt
sort -u names.txt
sort -n numbers.txt
sort -t: -k3,3n /etc/passwd
```

### `uniq`

`uniq` removes or reports adjacent repeated lines. Sort first when duplicates may not be next to each other.

| Option | Meaning |
|---|---|
| `-c` | Prefix each line with its count. |
| `-d` | Show only repeated lines. |
| `-u` | Show only lines that are not repeated. |
| `-i` | Ignore case when comparing. |

```bash
sort names.txt | uniq
sort names.txt | uniq -c
sort names.txt | uniq -d
```

## Count and translate characters

### `wc`

```bash
wc file.txt
wc -l file.txt
wc -w file.txt
wc -c file.txt
```

`-l` counts lines, `-w` counts words, and `-c` counts bytes. `-m` counts characters according to the current locale.

### `tr`

`tr` translates or deletes individual characters. It reads from standard input.

```bash
printf '%s\n' 'Debian Server' | tr '[:lower:]' '[:upper:]'
printf '%s\n' 'a:b:c' | tr ':' '\n'
printf '%s\n' 'a   b' | tr -s ' '
```

`-d` deletes characters; `-s` squeezes repeated characters in the selected set.

## Pass filenames safely with `xargs`

`xargs` builds command lines from standard input. Default whitespace parsing can break filenames containing spaces or newlines. Use NUL-delimited input from `find -print0` with `xargs -0` when handling arbitrary filenames.

```bash
find /var/log -type f -name '*.log' -print0 | xargs -0 -r ls -lh --
```

| Option | Meaning |
|---|---|
| `-0` | Read NUL-delimited items. |
| `-r` | Do not run the command when input is empty. |
| `-n <count>` | Pass at most that many input items per command invocation. |
| `-P <count>` | Run up to that many commands in parallel. |

Before using `xargs` with a command that deletes or modifies files, first run a harmless preview such as `printf '%s\n'` to confirm the list.

## Write output with `tee`

`tee` writes standard input to both standard output and one or more files.

```bash
command | tee output.log
command | tee -a output.log
```

`-a` appends rather than replacing the destination. With `sudo`, `tee` can write to a protected file because the `tee` process performs the write:

```bash
printf '%s\n' 'example' | sudo tee /etc/example.conf >/dev/null
```

This replaces the file's contents; verify the path and back up the file first.

## Useful combinations

```bash
# Count error lines in a log
grep -ic 'error' app.log

# Show the ten most common source IPs in a simple access log format
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head

# Find large log files and display their sizes
find /var/log -type f -name '*.log' -print0 | xargs -0 -r du -h
```

Log formats vary. Confirm the field layout before using `awk` to extract information from application logs.

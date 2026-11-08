# Logs and journald

## 1. Logging

On a systemd-managed installation, `systemd-journald` collects and stores journal entries from the kernel, services, and other sources. Query the journal with `journalctl`.

A separate syslog daemon such as `rsyslog` may also be installed. If present, it can write traditional text logs under `/var/log`, depending on its configuration.

## 2. `journalctl` — read the system journal

```bash
journalctl
```

### Common filters and options

| Option | Meaning |
|---|---|
| `-b` | Show entries from the current boot. |
| `-b -1` | Show entries from the previous boot, if retained. |
| `--list-boots` | List boots for which journal entries are available. |
| `-u <unit>` | Filter entries for a systemd unit. |
| `-k` | Show kernel messages. |
| `-p <priority>` | Filter by priority and more severe messages. |
| `--since <time>` | Show entries since a time expression. |
| `--until <time>` | Stop at an end time. |
| `-n <count>` | Show the most recent number of entries. |
| `-f` | Follow new entries, similar to `tail -f`. |
| `-o <format>` | Choose output format. |
| `--no-pager` | Do not open the output in a pager. |
| `--disk-usage` | Show disk space used by journal files. |

Priority names, from most severe to least severe, include `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, and `debug`. A filter such as `-p warning` includes warning and more severe priorities.

### Output formats

| Format | Purpose |
|---|---|
| `short` | Default human-readable format. |
| `short-iso` | Human-readable timestamps in ISO-like form. |
| `json` | One JSON object per entry. |
| `json-pretty` | Pretty-printed JSON. |
| `verbose` | Show detailed journal fields. |

```bash
journalctl -u <unit> -o short-iso
```

## 3. Read kernel messages with `dmesg`

`dmesg` displays the kernel ring buffer. It is useful for inspecting messages about devices, drivers, hardware, and kernel events.

```bash
dmesg -T | less
dmesg --level=err,warn
```

| Option | Meaning |
|---|---|
| `-T` | Attempt to convert timestamps to human-readable wall-clock time. The conversion can be imprecise, especially after resume or clock changes. |
| `--level=<list>` | Filter messages by severity level. |
| `-w` | Wait for new messages. |
| `-H` | Human-readable output in implementations that support it. |

## 4. Traditional log files

When a syslog daemon is installed and configured, common paths may include:

| Path | Common content | Notes |
|---|---|---|
| `/var/log/syslog` | General system messages. | Common with rsyslog, not guaranteed to exist. |
| `/var/log/auth.log` | Authentication and authorization messages. | Common Debian rsyslog configuration; access may be restricted. |
| `/var/log/kern.log` | Kernel messages. | Depends on syslog configuration. |
| `/var/log/dpkg.log` | Package-manager activity recorded by dpkg. | Availability and retention depend on package configuration. |
| `/var/log/apt/` | APT history and terminal logs. | Useful for package operations. |

## 5. Journal persistence and configuration

Journald commonly stores volatile data under `/run/log/journal/` and persistent data under `/var/log/journal/`.

Inspect relevant configuration and storage usage:

```bash
systemd-analyze cat-config systemd/journald.conf
journalctl --disk-usage
ls -ld /var/log/journal /run/log/journal
```

Main configuration file:

- `/etc/systemd/journald.conf`
- Drop-in configuration directory: `/etc/systemd/journald.conf.d/`

If persistent storage is required, plan disk capacity and retention deliberately. Creating `/var/log/journal` or changing journald configuration modifies the system; verify ownership and permissions.

## 6. Journal retention and cleanup

Inspect current usage first:

```bash
journalctl --disk-usage
```

The following commands remove archived journal files to meet a retention limit:

```bash
sudo journalctl --vacuum-time=14d
sudo journalctl --vacuum-size=500M
```

**Warning:** This removes retained journal history. Make sure older logs are no longer needed before vacuuming them.

| Option | Meaning |
|---|---|
| `--vacuum-time=<time>` | Remove archived journal files older than the specified time. |
| `--vacuum-size=<size>` | Remove archived journal files until archived usage is reduced toward the specified size. |
| `--vacuum-files=<count>` | Limit the number of archived journal files. |

Vacuuming affects archived files; active journal files may remain. Do not use vacuuming as an unplanned response to a full disk. First determine why logs are growing, preserve evidence required for incident investigation, and configure sensible retention limits.

## 7. Log rotation

Traditional text logs are often rotated by `logrotate` when installed. It can rotate, compress, and remove old log files according to configuration.

Common paths:

- `/etc/logrotate.conf` — main configuration.
- `/etc/logrotate.d/` — package or administrator-specific rules.

Do not manually delete active log files to reclaim space; applications may continue writing to an unlinked file until they reopen it.

## 8. Troubleshooting workflow

1. Identify the affected service or subsystem.
2. Check service state: `systemctl status <unit>`.
3. Read relevant entries: `journalctl -b -u <unit>`.
4. Narrow by time and priority: `--since`, `--until`, and `-p`.
5. For hardware or driver issues, inspect `journalctl -k -b` or `dmesg`.
6. Check whether logs are persistent and whether retention or disk space is affecting evidence.
7. Change the underlying service or system configuration only after identifying a likely cause.

Logs can contain usernames, IP addresses, request paths, and other sensitive information. Redact sensitive data before sharing logs publicly.

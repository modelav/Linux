# Scheduled tasks: cron and systemd timers

## 1. Scheduling choices

Debian-based servers commonly use two scheduling approaches:

- **cron:** A traditional scheduler for calendar-based recurring commands. User crontabs and system-wide cron files are simple and widely supported.
- **systemd timers:** Schedule activation of systemd units. They integrate with service dependencies, journal logging, unit sandboxing, resource controls, and missed-run handling.

Choose the mechanism already used and understood by the host. Prefer a systemd timer for a new server task when service lifecycle, journal integration, dependencies, or persistent catch-up behavior are useful. Cron remains appropriate for simple recurring jobs and existing infrastructure.

## 2. Cron service and configuration locations

The cron daemon is provided by the `cron` package. Check whether the package and service are present:

```bash
dpkg-query -W cron 2>/dev/null
systemctl status cron
```

Common locations:

| Path | Purpose |
|---|---|
| `/etc/crontab` | System-wide crontab; includes a user field. |
| `/etc/cron.d/` | Additional system-wide crontab files; includes a user field. |
| `/etc/cron.hourly/` | Periodic scripts commonly run hourly by system scheduling. |
| `/etc/cron.daily/` | Periodic scripts commonly run daily. |
| `/etc/cron.weekly/` | Periodic scripts commonly run weekly. |
| `/etc/cron.monthly/` | Periodic scripts commonly run monthly. |
| `/var/spool/cron/crontabs/` | User crontab spool directory; manage it with `crontab`, not by editing spool files directly. |

## 3. Manage a user's crontab

### `crontab`

| Option | Meaning |
|---|---|
| `-l` | List the current user's crontab. |
| `-e` | Edit the current user's crontab using the configured editor. |
| `-r` | Remove the current user's entire crontab. This is destructive. |
| `-u <user>` | Select another user's crontab; typically requires root privileges. |

```bash
crontab -l
crontab -e
sudo crontab -u root -l
```

## 4. Cron schedule syntax

A standard crontab entry has five time fields followed by a command:

```text
minute hour day-of-month month day-of-week command
```

| Field | Allowed values | Common special characters |
|---|---|---|
| Minute | `0-59` | `* , - /` |
| Hour | `0-23` | `* , - /` |
| Day of month | `0-31` (use real dates `1-31`) | `* , - /` |
| Month | `0-12` (use real months `1-12`) or names | `* , - /` |
| Day of week | `0-7` (Sunday is commonly `0` or `7`) or names | `* , - /` |

| Syntax | Meaning |
|---|---|
| `*` | Every valid value in the field. |
| `,` | List of values, such as `1,15`. |
| `-` | Range, such as `1-5`. |
| `/` | Step, such as `*/10` for every ten minutes. |

Examples:

| Schedule | Meaning |
|---|---|
| `*/5 * * * *` | Every five minutes. |
| `0 * * * *` | At the start of every hour. |
| `30 2 * * *` | Every day at 02:30. |
| `0 4 * * 0` | Sunday at 04:00. |
| `0 3 1 * *` | On the first day of each month at 03:00. |
| `0 9 * * 1-5` | At 09:00 Monday through Friday. |

## 5. User crontab versus system crontab

A user's crontab has five time fields followed immediately by the command. It runs as that crontab's owner.

```cron
# User crontab: run a script every day at 02:30
30 2 * * * /usr/local/sbin/example-backup
```

`/etc/crontab` and files under `/etc/cron.d/` include an extra username field between the schedule and the command:

```cron
# System crontab format
30 2 * * * root /usr/local/sbin/example-backup
```

Ensure they have the expected ownership and permissions. Put comments on separate lines: a `#` after a cron command is treated as part of the command, not as an inline comment.

## 6. Cron environment and reliability

Cron jobs run with a limited environment. Do not assume they inherit the same `PATH`, shell startup files, working directory, or interactive environment as your SSH session.

Recommended practices:

- Use absolute paths to executables and scripts.
- Set any required `PATH` explicitly in the crontab.
- Redirect output to a log or configure mail delivery deliberately.
- Quote paths and variables in shell commands.
- Ensure scripts are executable and readable by the job's user.
- Make jobs safe to run more than once where practical.
- Prevent overlapping executions if a previous run may still be active (for example, with a carefully designed `flock` wrapper).
- Test the script manually as the intended user before scheduling it.

Example:

```cron
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""

# Example: append stdout and stderr to a dedicated log
30 2 * * * /usr/local/sbin/example-backup >> /var/log/example-backup.log 2>&1
```


## 7. Inspect cron execution

```bash
systemctl status cron
journalctl -u cron --since today
```

If a syslog daemon is installed, cron messages may also appear in traditional logs such as `/var/log/syslog`. The exact destination depends on logging configuration.

When a job does not run, check:

1. Is the cron package installed and the daemon active?
2. Is the entry in the correct user's crontab or system-wide file?
3. Is the schedule valid, including day-of-month/day-of-week semantics?
4. Does the script exist, have the correct interpreter, and have executable permission?
5. Does the job's user have access to its files, directories, and commands?
6. Does the job depend on environment variables, network mounts, or services that are unavailable at that time?
7. Are output and errors captured somewhere useful?

## 8. `run-parts` and periodic directories

`run-parts` executes eligible scripts in a directory, subject to filename rules and options. Files that do not match the naming rules may be silently ignored.

Inspect which files would be run:

```bash
run-parts --test /etc/cron.daily
```

Execute only after understanding the impact:

```bash
sudo run-parts --report /etc/cron.daily
```

Running a periodic directory manually can repeat maintenance or cleanup jobs outside their normal schedule. It is not a harmless preview; use `--test` to preview the commands.

## 9. systemd timers

A systemd timer is a `.timer` unit that activates another unit, usually a service with the same base name. For a task called `example-backup`, create matching files:

- `/etc/systemd/system/example-backup.service`
- `/etc/systemd/system/example-backup.timer`

### Example service unit

```ini
# /etc/systemd/system/example-backup.service
[Unit]
Description=Run the example backup task

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/example-backup
```

### Example timer unit

```ini
# /etc/systemd/system/example-backup.timer
[Unit]
Description=Run the example backup task daily

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=5m

[Install]
WantedBy=timers.target
```

Explanation:

| Setting | Meaning |
|---|---|
| `Type=oneshot` | The service runs a command and exits rather than maintaining a long-running main process. |
| `ExecStart=` | Absolute path to the program to execute. |
| `OnCalendar=daily` | Calendar-based schedule; inspect exact matching times with `systemd-analyze calendar`. |
| `Persistent=true` | For calendar timers, trigger after activation if at least one scheduled run was missed while the timer was inactive. |
| `RandomizedDelaySec=5m` | Add a randomized delay up to the configured interval to spread load. |
| `WantedBy=timers.target` | Allows the timer to be enabled through the standard timer target. |

These are example unit files. **A system service runs as root by default when `User=` is not specified.** For production tasks, create a dedicated unprivileged service account and set `User=` (and, where appropriate, `Group=`) unless the task genuinely requires root. Review file access, permissions, environment, logging, failure behavior, and backup safety before deployment. Do not put passwords or secrets directly in unit files.

### Load, enable, and inspect the timer

These changes normally require root privileges.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now example-backup.timer
systemctl status example-backup.timer
systemctl list-timers --all
journalctl -u example-backup.service
```

| Command | Purpose |
|---|---|
| `systemctl daemon-reload` | Re-read unit definitions after files have changed. |
| `systemctl enable --now` | Enable the timer and start it immediately. |
| `systemctl list-timers --all` | List timers, including inactive ones. |
| `journalctl -u <service>` | Read logs from the service activated by the timer. |

Test the calendar expression before enabling a timer:

```bash
systemd-analyze calendar 'Mon..Fri 09:00'
```

Use `systemd-analyze calendar --iterations=5 '<expression>'` where supported to inspect upcoming occurrences. Time zone and clock synchronization matter for calendar timers.

## 10. Cron or systemd timer?

| Requirement | Often a good fit |
|---|---|
| Simple recurring command already managed by cron | Cron. |
| Job needs service dependencies or unit-level resource controls | systemd timer. |
| Need to inspect output through the system journal | systemd service activated by a timer. |
| Need a calendar job to catch up after the machine was down | systemd timer with `Persistent=true`, where appropriate. |
| Existing application or operations standard uses cron | Keep cron unless there is a reason to migrate. |

Neither scheduler automatically makes a job reliable. The script itself should handle failures, locking, logging, permissions, retries, and partial completion.

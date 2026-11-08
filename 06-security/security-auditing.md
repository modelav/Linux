# Security auditing

Security auditing means reviewing authentication, privilege use, service activity, and important configuration changes. Start with the logs already available on the server; install a dedicated audit system only when its additional detail is needed.

## Review login activity

```bash
who
w
last -n 20
lastb -n 20
```

| Command | Purpose |
|---|---|
| `who` | Show users currently logged in. |
| `w` | Show logged-in users and their current activity. |
| `last` | Read successful login history from `wtmp`. |
| `lastb` | Read failed login history from `btmp`; access may require root. |

These records depend on the relevant accounting files being present and maintained. They are not a complete record of all authentication activity.

## Review SSH and sudo activity

```bash
sudo journalctl -u ssh --since today
sudo journalctl _COMM=sudo --since today
```

If rsyslog is installed, authentication events may also be recorded in `/var/log/auth.log`:

```bash
sudo less /var/log/auth.log
```

Logs may contain IP addresses, usernames, and other sensitive information. Restrict access and redact sensitive details before sharing them.

## Check recent changes and service failures

```bash
sudo journalctl -p warning --since '24 hours ago'
systemctl --failed
sudo find /etc -type f -mtime -7 -print
```

The `find` command lists files under `/etc` modified in the last seven 24-hour periods. It is a useful starting point, not proof that every security-relevant change is captured.

## Use Linux Audit (`auditd`) when detailed audit records are needed

The optional `auditd` package provides kernel audit event collection and tools such as `auditctl`, `ausearch`, and `aureport`.

```bash
sudo apt install auditd
systemctl status auditd
sudo auditctl -s
sudo auditctl -l
```

### Add a temporary watch

For example, monitor write and attribute changes to the local account databases:

```bash
sudo auditctl -a always,exit -F arch=b64 -F path=/etc/passwd -F perm=wa -k identity
sudo auditctl -a always,exit -F arch=b64 -F path=/etc/shadow -F perm=wa -k identity
```

- `-a always,exit` records matching system-call exit events.
- `-F arch=b64` selects the 64-bit system-call ABI.
- `-F path=...` selects the file to watch.
- `-F perm=wa` watches write and attribute changes.
- `-k identity` adds a search key to matching events.

The [Debian `auditctl` manual](https://manpages.debian.org/trixie/auditd/auditctl.8.en.html) documents the rule syntax and architecture filters.

These are runtime rules and are not automatically persistent across reboot. On systems that also run 32-bit programs, add equivalent `arch=b32` rules when those accesses must be audited. For persistent rules, place supported rules in `/etc/audit/rules.d/*.rules` and load them using the Debian audit rules tooling, commonly `augenrules --load`.

The older `-w <path> -p <permissions>` watch form is retained for compatibility but is deprecated due to performance concerns. If rules do not appear to record events, inspect `auditctl -l` for a `never,task` rule that disables auditing for newly created processes, and follow the installed Debian audit rules configuration before changing it. Audit rules can generate substantial data. Plan retention and storage before monitoring broad paths or high-volume system calls.

### Search audit events

```bash
sudo ausearch -k identity -i
sudo ausearch -m USER_LOGIN -ts today -i
sudo aureport --auth
sudo aureport --summary
```

| Option | Meaning |
|---|---|
| `-k <key>` | Search by audit rule key. |
| `-m <type>` | Search by audit message type. |
| `-ts today` | Limit results to today. |
| `-i` | Interpret numeric fields where possible. |
| `aureport --auth` | Summarize authentication-related events. |

## A practical review routine

- Review failed SSH logins and unexpected successful logins.
- Check privileged commands and changes to sudo policy.
- Review failed services and unusual warnings.
- Verify that audit and system logs have sufficient retention and disk space.
- Investigate unexplained changes to `/etc`, service definitions, and privileged executables.
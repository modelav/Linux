# Security updates

Debian publishes security fixes through its security archive and Debian Security Advisories (DSAs). Keep package metadata current and apply updates regularly; delaying security updates leaves known vulnerabilities unpatched.

## Check Debian security information

- [Debian Security Information](https://www.debian.org/security/) lists current advisories and links to the Debian Security Tracker.
- [Debian Security Tracker](https://security-tracker.debian.org/tracker/) provides package and CVE status by Debian release.
- [Debian 13 release notes](https://www.debian.org/releases/trixie/releasenotes) describe release-specific changes.

For a package or CVE, check the tracker for the Debian 13 status and fixed package version instead of assuming an upstream version number alone indicates whether a vulnerability is fixed. Debian may backport security fixes.

## Apply updates manually

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

| Command | Purpose |
|---|---|
| `apt update` | Refresh package indexes; it does not install upgrades. |
| `apt list --upgradable` | List packages with available upgrades. |
| `apt upgrade` | Upgrade installed packages without removing packages to satisfy the upgrade plan. |
| `apt full-upgrade` | Resolve dependency changes and may remove packages when required. Review the proposed changes carefully. |

For a planned maintenance window, simulate the package operation first:

```bash
sudo apt-get -s upgrade
sudo apt-get -s full-upgrade
```

Review package removals, service restarts, kernel updates, and disk space before confirming a major upgrade.

## Install unattended-upgrades

`unattended-upgrades` can automatically install updates from configured origins, commonly including Debian security updates. Debian describes the package and periodic update configuration in its [Periodic Updates guide](https://wiki.debian.org/PeriodicUpdates). Install and inspect it:

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

The reconfiguration dialog can enable automatic updates. Actual behavior depends on the origin patterns and APT periodic settings configured on the host.

## Inspect automatic-update configuration

Common files:

| Path | Purpose |
|---|---|
| `/etc/apt/apt.conf.d/20auto-upgrades` | Common periodic update and unattended-upgrade switches. |
| `/etc/apt/apt.conf.d/50unattended-upgrades` | Allowed origins and unattended-upgrade behavior. |
| `/etc/apt/apt.conf.d/` | APT configuration snippets. |

Review the active configuration:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
less /etc/apt/apt.conf.d/50unattended-upgrades
```

Do not assume the package's installation means security updates are enabled. Check the effective origin configuration and periodic settings. Prefer a local drop-in or documented configuration override rather than editing package-managed defaults without a reason.

Useful commands:

```bash
sudo unattended-upgrade --dry-run --debug
systemctl list-timers 'apt-*'
systemctl status apt-daily.timer apt-daily-upgrade.timer
```

`--dry-run` simulates the unattended-upgrade operation and `--debug` provides diagnostic detail.

## Review update logs

```bash
less /var/log/unattended-upgrades/unattended-upgrades.log
less /var/log/unattended-upgrades/unattended-upgrades-dpkg.log
less /var/log/apt/history.log
```

These files exist only when the corresponding package and logging configuration are present.

## Check whether services need restarting

Library updates may leave running services using old libraries until they are restarted. The optional `needrestart` package can identify processes that need restarting:

```bash
sudo needrestart
sudo needrestart -r l
```

`-r l` lists items requiring restart without restarting them. Review results and restart services during an appropriate maintenance window.

After kernel updates, a reboot is normally required to run the newly installed kernel. Check the running kernel with `uname -r` and compare it with installed kernel packages; do not rely on a reboot marker file being present on every installation.

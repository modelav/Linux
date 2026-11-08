# Package upgrades and maintenance

Regular package maintenance keeps a Debian server secure and supported. Plan upgrades around service availability, backups, and a way to recover access if a change fails.

## Routine upgrade workflow

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

Review the list of packages and the proposed transaction. `apt upgrade` avoids removing installed packages; some upgrades may be kept back if they require package removals or other changes outside that operation's rules.

For a more complete upgrade within the current Debian release:

```bash
sudo apt full-upgrade
```

`full-upgrade` may install new dependencies and remove packages. Review the removal list before confirming, especially on servers running databases, web services, storage, or remote-access services.

## Simulate a transaction

Use simulation before a significant installation, removal, or upgrade:

```bash
apt-get -s upgrade
apt-get -s full-upgrade
apt-get -s install package-name
apt-get -s remove package-name
```

`-s` / `--simulate` prints the planned changes without performing the package transaction. It is a useful preview, but the actual result can change if package indexes or system state change before the real operation.

## Check for reboot requirements

After a kernel, libc, or other core-system upgrade, a reboot may be required to use the updated components.

```bash
if [ -f /var/run/reboot-required ]; then
    cat /var/run/reboot-required
    cat /var/run/reboot-required.pkgs 2>/dev/null
else
    echo "No reboot-required marker found"
fi
```

The marker is created by packages that use Debian's reboot-notification mechanism. Its absence is not a universal guarantee that a reboot is unnecessary. Check service-specific guidance and whether important processes still use deleted or old libraries.

## Check package logs

```bash
less /var/log/apt/history.log
less /var/log/apt/term.log
tail -n 100 /var/log/dpkg.log
```

Use these logs to review what was installed, upgraded, removed, or configured and when.

## Unattended security upgrades

The `unattended-upgrades` package can install updates automatically from configured APT sources according to its policy.

Check whether it is installed and inspect its configuration:

```bash
dpkg-query -W unattended-upgrades
systemctl list-timers "apt-daily*"
sed -n '1,240p' /etc/apt/apt.conf.d/50unattended-upgrades
```

Common configuration locations:

- `/etc/apt/apt.conf.d/20auto-upgrades` — periodic APT activity settings.
- `/etc/apt/apt.conf.d/50unattended-upgrades` — unattended-upgrade policy.

Run a dry run when the package is installed:

```bash
sudo unattended-upgrade --dry-run --debug
```

Logs are commonly written to:

- `/var/log/unattended-upgrades/unattended-upgrades.log`
- `/var/log/unattended-upgrades/unattended-upgrades-dpkg.log`

Automatic updates do not replace monitoring, backups, reboot planning, or testing application compatibility. Decide whether service restarts and reboots may happen automatically before enabling unattended behavior on a production host.

## Major Debian release upgrades

A major release upgrade (for example, Debian 12 to Debian 13) is different from a routine package upgrade. Follow the official release notes for the source and target releases. Review repository entries, backups, available disk space, third-party repositories, and service compatibility before starting.

Do not change a source from one codename to another and immediately run `apt full-upgrade` without following the release notes and their preparation steps.

# APT troubleshooting

Use these checks when APT cannot refresh indexes, install packages, or complete a package operation.

## Update fails or repository cannot be reached

```bash
sudo apt update
```

Read the first error rather than focusing only on the final summary. Check:

- The server's date and time.
- DNS resolution and network connectivity.
- The repository URI and suite/codename in source files.
- Whether the repository still publishes the configured Debian suite and architecture.
- Whether a proxy or firewall is blocking access.

Check time and name resolution:

```bash
timedatectl status
getent hosts deb.debian.org
```

A `404` or `does not have a Release file` error often means the repository URL or suite is wrong or no longer supported.

## Signature or public-key errors

APT verifies repository metadata signatures. If verification fails, check that the repository is legitimate, its key has not changed unexpectedly, and the correct repository-scoped keyring is configured with `Signed-By`. See [`apt-secure(8)`](https://manpages.debian.org/trixie/apt/apt-secure.8.en.html) for the authentication model.

Do not disable signature verification or mark an unknown repository as trusted to make the error disappear. Obtain key changes from the repository owner through a trusted channel and verify the fingerprint.

## dpkg was interrupted

If APT reports that dpkg was interrupted, first allow any other package operation to finish. Then inspect package state:

```bash
sudo dpkg --audit
sudo dpkg --configure -a
sudo apt --fix-broken install
```

These commands attempt to finish unpacked package configuration and repair dependency state. Read their output. 

## Package manager lock is held

APT and dpkg use locks to prevent simultaneous package operations. Check for an active package-management process:

```bash
ps -ef | grep -E '[a]pt|[d]pkg|unattended'
systemctl status apt-daily.service apt-daily-upgrade.service
```

Wait for a legitimate package operation to finish. Do not delete lock files while a package process is active. Removing lock files does not safely release a lock held by a running process and can corrupt package state if another operation starts.

## Package has unmet dependencies

Check the package and candidate versions:

```bash
apt policy package-name
apt-cache depends package-name
sudo apt-get check
```

Then refresh indexes and simulate a repair:

```bash
sudo apt update
apt-get -s --fix-broken install
```

If the proposed changes are appropriate, run the repair without `-s`:

```bash
sudo apt --fix-broken install
```

Review any packages that APT proposes to remove or downgrade.

## Package is held back

```bash
apt list --upgradable
apt-mark showhold
apt policy package-name
```

A package may be kept back because its upgrade needs new dependencies, package removals, or because it is held. Understand the reason before unholding it or using `full-upgrade`.

## Package files are missing or damaged

Reinstall a package from configured repositories:

```bash
sudo apt install --reinstall package-name
```

This replaces files managed by the package, but does not necessarily repair local configuration or application data. Back up changed configuration before reinstalling.

## Disk space is full

Package operations can fail when `/`, `/var`, or `/var/cache` is full.

```bash
df -h
df -i
sudo du -xhd1 /var 2>/dev/null | sort -h
sudo apt clean
```

`apt clean` removes downloaded package archives from APT's cache. It does not remove installed packages. Investigate the actual space consumer before deleting files under `/var/lib` or `/var/log`.

## Inspect package logs and status

```bash
tail -n 100 /var/log/apt/term.log
tail -n 100 /var/log/dpkg.log
dpkg --audit
dpkg-query -W -f='${Package}\t${Version}\t${Status}\n' package-name
```

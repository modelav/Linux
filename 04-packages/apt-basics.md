# APT basics

APT (Advanced Package Tool) is Debian's main package-management interface. Use it to search repositories, install and remove software, and upgrade installed packages.

## Refresh package information

```bash
sudo apt update
```

`update` downloads the latest package indexes from configured repositories. It does not install or upgrade packages.

Useful options:

| Option | Meaning |
|---|---|
| `-y` | Automatically answer yes to prompts. Review the planned operation before using it. |
| `-q` | Reduce output verbosity; can be repeated for quieter output. |
| `-o <setting>` | Override an APT configuration setting for this invocation. |

## Search and inspect packages

```bash
apt search nginx
apt show nginx
apt list --installed
apt list --upgradable
```

| Command | Purpose |
|---|---|
| `apt search <term>` | Search package names and descriptions. |
| `apt show <package>` | Show package description, version, dependencies, and other metadata. |
| `apt list --installed` | List packages currently installed. |
| `apt list --upgradable` | List installed packages with available upgrades. |
| `apt policy <package>` | Show installed and candidate versions and the repositories offering them. |

For dependency details, use `apt-cache depends <package>` and `apt-cache rdepends <package>`.

## Install packages

```bash
sudo apt install nginx
sudo apt install curl vim git
sudo apt install --no-install-recommends package-name
sudo apt install --reinstall package-name
sudo apt install package-name=1.2.3-1
```

| Option / syntax | Meaning |
|---|---|
| `--no-install-recommends` | Do not automatically install packages marked as Recommends. Required dependencies are still installed. |
| `package=version` | Request a specific available package version. Check `apt policy` first. |
| `package/release` | Request a version from a particular suite or codename, such as `package/trixie`. Review dependency changes carefully. |
| `--reinstall` | Reinstall a package that is already installed. |


## Remove packages

```bash
sudo apt remove package-name
sudo apt purge package-name
sudo apt autoremove
```

| Command | Purpose |
|---|---|
| `remove` | Remove the package but usually keep its system-wide configuration files. |
| `purge` | Remove the package and its packaged system-wide configuration files. |
| `autoremove` | Remove packages installed automatically as dependencies that are no longer required. |

Purging a package does not necessarily remove data stored in home directories or application data directories such as `/var/lib`.

## Upgrade installed packages

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

`upgrade` installs available upgrades without removing currently installed packages. Packages that require removal of another installed package are kept back.

```bash
sudo apt full-upgrade
```

`full-upgrade` can install new dependencies and remove packages when required to complete the system upgrade.

## Download and clean package files

```bash
apt download package-name
sudo apt clean
sudo apt autoclean
```

| Command | Purpose |
|---|---|
| `apt download` | Download a package file into the current directory without installing it. |
| `apt clean` | Remove downloaded package archives from APT's local cache. |
| `apt autoclean` | Remove cached archives that can no longer be downloaded from configured repositories. |

The APT archive cache is normally under `/var/cache/apt/archives/`.

## Useful logs

| Path | Content |
|---|---|
| `/var/log/apt/history.log` | APT transaction history. |
| `/var/log/apt/term.log` | APT terminal output. |
| `/var/log/dpkg.log` | Package state changes recorded by dpkg. |

## Scripting note

Use `apt` for interactive administration. Its command-line output and behavior may change between versions. For scripts, prefer `apt-get`, `apt-cache`, and other dedicated APT tools with documented, stable interfaces.

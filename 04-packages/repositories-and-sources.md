# Repositories and sources

APT repositories provide package indexes and package archives. Debian recommends the deb822 `.sources` format for configuring repositories.

## Important paths

| Path | Purpose |
|---|---|
| `/etc/apt/sources.list` | Traditional one-line repository configuration; may be empty or unused when deb822 files are used. |
| `/etc/apt/sources.list.d/` | Directory for additional `.list` and `.sources` files. |
| `/etc/apt/apt.conf.d/` | APT configuration fragments. |
| `/etc/apt/preferences.d/` | Package version-priority (pinning) rules. |
| `/etc/apt/keyrings/` | Recommended location for administrator-managed repository keyrings. |
| `/usr/share/keyrings/` | Common location for package-managed keyrings. |

## View configured sources

```bash
apt policy
apt-get indextargets
ls -l /etc/apt/sources.list.d/
sed -n '1,200p' /etc/apt/sources.list.d/debian.sources
```

The exact filename varies. Inspect all files in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/` before changing repository configuration.

## Debian 13 deb822 format

A typical Debian source file is `/etc/apt/sources.list.d/debian.sources`:

```text
Types: deb
URIs: https://deb.debian.org/debian
Suites: trixie trixie-updates
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb
URIs: https://security.debian.org/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

Meaning of the fields:

| Field | Purpose |
|---|---|
| `Types` | Package index type: `deb` for binary packages, `deb-src` for source packages. |
| `URIs` | Repository base address. |
| `Suites` | Release suite or codename, such as `trixie` or `trixie-security`. |
| `Components` | Repository sections, such as `main`, `contrib`, `non-free`, and `non-free-firmware`. |
| `Signed-By` | Keyring used to authenticate this repository. |

After editing sources, refresh package indexes:

```bash
sudo apt update
```

Read the output. Signature failures, missing Release files, or a repository that does not support the configured suite should be fixed rather than bypassed.

## Third-party repository keys

For a third-party repository, use the repository vendor's official instructions and verify its signing-key fingerprint through a trusted channel. Prefer a dedicated keyring and a `Signed-By` entry scoped to that repository.

Example layout:

```text
/etc/apt/keyrings/vendor-archive-keyring.gpg
/etc/apt/sources.list.d/vendor.sources
```

Example deb822 entry:

```text
Types: deb
URIs: https://packages.vendor.example/debian
Suites: trixie
Components: main
Signed-By: /etc/apt/keyrings/vendor-archive-keyring.gpg
```

The URL and suite above are placeholders, not a real repository. The keyring must be readable by APT's `_apt` user.

Avoid old instructions that use `apt-key add`; the global trusted-key approach is deprecated and grants a key broader trust than necessary. Do not use `trusted=yes` to silence signature errors. APT’s repository authentication model is described in [`apt-secure(8)`](https://manpages.debian.org/trixie/apt/apt-secure.8.en.html).

## `main`, `contrib`, `non-free`, and `non-free-firmware`

| Component | Meaning |
|---|---|
| `main` | Debian packages that comply with the Debian Free Software Guidelines. |
| `contrib` | Free software that depends on software outside `main`. |
| `non-free` | Packages that do not meet the Debian Free Software Guidelines. |
| `non-free-firmware` | Non-free firmware packages, separated into their own component. |


## Stable suite versus codename

Repository suites can use moving names such as `stable` or fixed codenames such as `trixie`. A moving suite name changes meaning when Debian releases a new stable version. For servers where predictable release upgrades matter, using the codename makes the configured release explicit.

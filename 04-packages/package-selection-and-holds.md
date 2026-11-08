# Package selection and holds

Use APT policy, package marks, and preferences to see which version APT will install and to control whether selected packages are upgraded automatically.

## Check package versions and candidates

```bash
apt policy nginx
apt-cache madison nginx
apt-cache show nginx
```

| Command | Purpose |
|---|---|
| `apt policy <package>` | Show installed version, candidate version, repository origins, and priorities. |
| `apt-cache madison <package>` | List available versions from configured package indexes. |
| `apt-cache show <package>` | Show package metadata for available versions. |

APT selects a candidate based on version availability and pin priorities. Do not assume that the newest version number is always the candidate.

## Install a specific version

```bash
apt policy package-name
sudo apt install package-name=1.2.3-1
```

Check that the requested version is available and review all dependency changes before confirming. APT may need to install or downgrade dependencies to satisfy the request.

To request a package from a particular release:

```bash
sudo apt install package-name/trixie
```

Use release targeting only when you understand the package's dependencies and repository priorities.

## Hold a package

A hold prevents APT from automatically upgrading or removing the package through normal upgrade operations.

```bash
sudo apt-mark hold package-name
apt-mark showhold
sudo apt-mark unhold package-name
```

| Command | Purpose |
|---|---|
| `apt-mark hold <package>` | Place a package on hold. |
| `apt-mark unhold <package>` | Remove a hold. |
| `apt-mark showhold` | List held packages. |

A hold does not guarantee that a package can never be changed: explicit requests and lower-level tools can still alter package state.

## Manual and automatic package marks

APT marks packages installed explicitly by the administrator as manual. Dependencies installed only to satisfy another package are generally marked automatic.

```bash
apt-mark showmanual
apt-mark showauto
sudo apt-mark manual package-name
sudo apt-mark auto package-name
```

Mark an automatically installed dependency as manual when you want to keep it even if APT no longer sees it as required by another package.

## Package pinning

Pinning assigns priorities to packages from particular releases, origins, or versions. Rules are stored in:

- `/etc/apt/preferences`
- `/etc/apt/preferences.d/*.pref`

Example: give packages from a particular origin a lower priority unless explicitly requested:

```text
Package: *
Pin: origin "packages.vendor.example"
Pin-Priority: 100
```

This is an illustrative rule. It gives that origin a lower priority than ordinary repositories (commonly 500), but it is not a full allowlist or a guarantee that no package from that origin can be installed. Pin matching and priorities are subtle; verify the result with `apt policy <package>` before installing or upgrading.

Important priority behavior:

| Priority | General effect |
|---:|---|
| `100` | Common priority for installed packages and some not-automatic archives. |
| `500` | Common default for ordinary repository versions. |
| `990` | Common priority for the configured target release. |
| Above `1000` | Can allow a downgrade; use only with a clear reason. |
| Below `0` | Prevents a version from being selected. |

Avoid using pinning as a substitute for a coherent Debian release strategy. Mixing stable, testing, and unstable repositories can make upgrades difficult or break dependency consistency.

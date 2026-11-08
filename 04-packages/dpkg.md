# dpkg

`dpkg` is the lower-level Debian package tool. APT normally handles repository access and dependency resolution, while dpkg performs package installation, unpacking, configuration, and removal on the local system.

## Query installed packages

### `dpkg -l` and `dpkg-query`

```bash
dpkg -l nginx
dpkg-query -W nginx
dpkg-query -W -f='${Package}\t${Version}\t${Status}\n' 'linux-image-*'
```

| Command / option | Meaning |
|---|---|
| `dpkg -l <pattern>` | List package records matching a pattern. |
| `dpkg-query -W <pattern>` | Show package name and installed version. |
| `-f=<format>` | Choose fields to display. |
| `${Status}` | Show the package's status field. |

`dpkg -l` shows a three-character status near the start of each row. The usual installed state is `ii`; other combinations can indicate unpacked, half-configured, removed, or failed states. See `man dpkg-query` for the full status codes.

### Package files and ownership

```bash
dpkg -L nginx
dpkg -S /usr/sbin/nginx
dpkg-query -s nginx
```

| Command | Purpose |
|---|---|
| `dpkg -L <package>` | List files installed by a package. |
| `dpkg -S <path>` | Find which installed package owns a path. |
| `dpkg-query -s <package>` | Show the local package status and metadata. |

`dpkg -S` searches the local package database. It does not locate files belonging to packages that are not installed; `apt-file` can search file lists from repository packages when installed and configured.

## Install a local `.deb` file

```bash
sudo dpkg -i ./package-file.deb
```

`dpkg -i` installs a local package file but does not download and resolve missing dependencies from repositories. If dependency errors occur, use APT to resolve them:

```bash
sudo apt --fix-broken install
```

Inspect the proposed changes before confirming.

For routine repository packages, prefer `sudo apt install <package>` so APT can resolve dependencies before dpkg performs the installation.

## Remove packages

```bash
sudo dpkg -r package-name
sudo dpkg -P package-name
```

| Action | Meaning |
|---|---|
| `-r`, `--remove` | Remove an installed package but retain configuration files where possible. |
| `-P`, `--purge` | Remove the package and its packaged configuration files. |

Prefer APT for normal removal so reverse dependencies and related package changes are considered.

## Repair an interrupted package configuration

If a package operation was interrupted, some packages may be unpacked but not configured.

```bash
sudo dpkg --configure -a
sudo apt --fix-broken install
```

`dpkg --configure -a` attempts to configure all unpacked but unconfigured packages. It does not fix every possible dependency or filesystem problem; read the error output and diagnose the cause.

## Audit package state

```bash
sudo dpkg --audit
dpkg --verify package-name
```

| Command | Purpose |
|---|---|
| `dpkg --audit` | Find packages that are partially installed or otherwise need attention. |
| `dpkg --verify <package>` | Compare installed files with metadata recorded by dpkg where verification data is available. |

Verification does not prove that every file is secure or unmodified; not all file attributes or package contents can be verified this way.

## Inspect a `.deb` without installing it

### `dpkg-deb`

```bash
dpkg-deb -I ./package-file.deb
dpkg-deb -c ./package-file.deb
```

| Option | Meaning |
|---|---|
| `-I`, `--info` | Show package control information. |
| `-c`, `--contents` | List files contained in the package archive. |
| `-x <deb> <directory>` | Extract package files without installing the package. |

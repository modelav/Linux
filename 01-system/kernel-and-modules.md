# Kernel and kernel modules

## 1. Kernel and userspace

The Linux kernel manages hardware, memory, processes, networking, filesystems, and system calls. User applications interact with the kernel through system calls and kernel interfaces.

A **kernel module** is code that can be loaded into or removed from the running kernel. Modules commonly provide device drivers, filesystems, and other kernel functionality. Not every driver or kernel feature is a loadable module; some are built into the kernel.

## 2. Identify the running kernel

```bash
uname -r
uname -a
cat /proc/version
cat /proc/cmdline
```

| Path / command | Purpose |
|---|---|
| `uname -r` | Running kernel release string. |
| `/proc/version` | Kernel build/version information exposed by the running kernel. |
| `/proc/cmdline` | Kernel command-line parameters used for this boot. |
| `/proc/modules` | Snapshot of currently loaded kernel modules. |
| `/sys/module/` | Kernel module objects and related attributes exposed through sysfs. |


## 3. List loaded modules with `lsmod`

```bash
lsmod
```

`lsmod` formats information from `/proc/modules`. Common columns:

| Column | Meaning |
|---|---|
| `Module` | Module name. |
| `Size` | Reported module size. |
| `Used by` | Use count and, where available, modules depending on it. |

A module not appearing in `lsmod` may be built into the kernel, unused, or unavailable for the running kernel. Use `modinfo` and kernel configuration or package documentation for further investigation.

## 4. Inspect a module with `modinfo`

```bash
modinfo ext4
modinfo -F filename ext4
modinfo -F description ext4
```

| Option | Meaning |
|---|---|
| `-F <field>` | Print one metadata field, such as `filename`, `description`, `license`, or `vermagic`. |
| `-n` | Print the module filename. |
| `-p` | Show module parameters. |

## 5. Load a module with `modprobe`

`modprobe` loads a module and resolves dependencies using module metadata. Use it instead of low-level `insmod` for normal module management.

These changes normally require root privileges.

```bash
sudo modprobe <module-name>
lsmod | grep -w '<module-name>'
```

### Useful options

| Option | Meaning |
|---|---|
| `-v` / `--verbose` | Show more details about actions. |
| `-r` / `--remove` | Remove a module and attempt to remove unused dependencies. |
| `-n` / `--dry-run` | Do not actually insert or remove modules; show what would be done. |
| `-f` / `--force` | Override selected version or state checks; dangerous and generally not appropriate for routine troubleshooting. |
| `-c` | Dump effective modprobe configuration. |
| `-b` / `--use-blacklist` | Apply blacklist rules from modprobe configuration to module names. |

Loading a module can activate a device driver or change system behavior. Confirm its purpose, parameters, dependencies, and compatibility with the running kernel before loading it on a production server.

## 6. Remove a module

These operations normally require root privileges and can disrupt dependent services.

```bash
lsmod | grep -w '<module-name>'
sudo modprobe -r <module-name>
```


## 7. Module configuration and automatic loading

Common configuration locations:

| Path | Purpose |
|---|---|
| `/etc/modprobe.d/*.conf` | Local options, aliases, install rules, and blacklist configuration for modprobe. |
| `/etc/modules-load.d/*.conf` | Module names to load during boot. |
| `/usr/lib/modules-load.d/` | Vendor-supplied modules-load configuration on systems using this layout. |
| `/lib/modules/$(uname -r)/` | Module tree and dependency metadata for the running kernel on traditional Debian layouts; inspect actual paths because `/lib` may be merged into `/usr`. |

`/etc/modules` is a legacy configuration mechanism that may still be present on some installations. Prefer the documented `modules-load.d` mechanism for new configuration when appropriate.

### Example: load a module at boot

Do not enable a module at boot until you have confirmed that it is required and works safely on the running kernel. Create a dedicated configuration file rather than editing vendor files:

```ini
# /etc/modules-load.d/example.conf
example_module
```

### Module options

Options can be supplied through `/etc/modprobe.d/*.conf`. The syntax and supported parameters are module-specific:

```ini
# Example only; option names depend on the module
options example_module example_option=1
```

Do not copy example parameters to unrelated modules. Check `modinfo -p <module>` and the driver's documentation first.

## 8. Blacklisting modules

Blacklisting can affect automatic module loading, but it is not a universal guarantee that a module cannot be loaded. Built-in kernel drivers and explicit loading mechanisms may behave differently.

A blacklist rule in `/etc/modprobe.d/<name>.conf` has the form:

```ini
blacklist example_module
```


## 9. Rebuild module dependency metadata

`depmod` generates module dependency and map files for a kernel's module tree. Package installation and kernel package hooks normally maintain this metadata.

```bash
# Rebuild dependency metadata for the currently running kernel
sudo depmod -a "$(uname -r)"

# Preview generated metadata instead of writing it
sudo depmod -n "$(uname -r)"
```

| Option | Meaning |
|---|---|
| `-a` | Probe all modules. This is the default when no filenames are specified. |
| `-A` | Quick check; exit without regenerating files if no modules are newer than the existing dependency file. |
| `-n` | Write generated metadata to standard output instead of the module directory. |

`depmod` writes module metadata when run without `-n`; package hooks normally maintain it. Use it manually only when a driver or kernel installation procedure calls for it, and confirm the target kernel version before changing metadata.

## 10. Troubleshooting module load failures

1. Confirm the running kernel: `uname -r`.
2. Check whether the module file exists: `modinfo <module>`.
3. Inspect the kernel log: `journalctl -k -b` or `dmesg`.
4. Check module dependencies and options: `modinfo -p <module>` and `modprobe -c`.
5. Check whether the module is already loaded: `lsmod`.
6. Confirm the module is compatible with the running kernel and architecture.
7. Avoid force options until the underlying compatibility or dependency issue is understood.

DKMS-based third-party modules may need to be rebuilt for a newly installed kernel. This depends on the module package and build environment; consult the specific driver's documentation.

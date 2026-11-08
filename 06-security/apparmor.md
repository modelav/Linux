# AppArmor

AppArmor is a Linux security module that confines programs using per-program profiles. A profile defines which files, capabilities, and other resources an application may access. See the [Debian AppArmor manual](https://manpages.debian.org/trixie/apparmor/apparmor.7.en.html).

## Check AppArmor status

```bash
sudo aa-status
systemctl status apparmor
cat /sys/module/apparmor/parameters/enabled
```

The `aa-status` command reports whether AppArmor is enabled and which profiles are loaded and enforcing or in complain mode. The profile-management commands `aa-enforce`, `aa-complain`, and `aa-disable` are provided by `apparmor-utils`; install the packages when needed: `sudo apt install apparmor apparmor-utils`.

## Profile modes

| Mode | Behavior |
|---|---|
| Enforce | Deny operations that violate the profile and log relevant denials. |
| Complain | Allow operations that violate the profile but log them for profile development and troubleshooting. |

Complain mode is useful for diagnosing or developing a profile. It is not equivalent to enforcing protection.

## Manage profiles

```bash
sudo aa-status
sudo aa-enforce /etc/apparmor.d/usr.sbin.example
sudo aa-complain /etc/apparmor.d/usr.sbin.example
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.example
```

| Command | Purpose |
|---|---|
| `aa-enforce <profile>` | Put a profile into enforce mode. |
| `aa-complain <profile>` | Put a profile into complain mode. |
| `apparmor_parser -r <profile>` | Replace or reload a profile after editing it. |
| `aa-disable <profile>` | Disable a profile. Use only when there is a clear reason and an alternative control. |

Use the actual profile file path under `/etc/apparmor.d/`. Profile names and paths depend on installed packages.

## Find AppArmor denials

AppArmor messages commonly appear in the kernel journal:

```bash
sudo journalctl -k | grep -i apparmor
sudo journalctl -k --since '1 hour ago' | grep -i 'apparmor="DENIED"'
```

A denial often includes the profile, operation, requested path, and process. Check whether the application genuinely needs the denied access before changing the profile.

## Profile configuration

Common locations:

| Path | Purpose |
|---|---|
| `/etc/apparmor.d/` | Local and package-provided profiles. |
| `/etc/apparmor.d/local/` | Local profile customization snippets where a profile includes them. |
| `/sys/kernel/security/apparmor/` | AppArmor information exposed by securityfs when mounted. |

Prefer local profile snippets when the package profile provides them. Direct edits to package-maintained profiles may be overwritten during upgrades.

## Troubleshooting

- Confirm AppArmor is enabled in the running kernel and the service is active.
- Check that the relevant profile is loaded and in the intended mode.
- Read kernel journal messages for denials.
- Verify the application's expected file paths and runtime behavior.
- Make the narrowest profile change that allows the required operation; avoid disabling the whole profile to bypass one denial.


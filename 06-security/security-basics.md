# Security basics

A secure server starts with a small attack surface, controlled access, timely updates, and useful logs. These practices are a foundation; apply them according to the services and risks of the actual server.

## Keep the system minimal

Install only the packages and services the server needs. Review listening sockets and enabled services periodically:

```bash
systemctl --type=service --state=running
ss -tulpn
apt list --installed
```

Remove an unnecessary package only after checking dependencies and whether another service relies on it.

## Use dedicated accounts and least privilege

- Use a normal user account for administration and elevate individual commands with `sudo`.
- Give each application its own service account where practical.
- Avoid running application processes as root unless their function requires it.
- Keep service data and configuration permissions restricted to the users that need access.


## Secure remote access

- Prefer SSH public-key authentication.
- Use a unique account for each administrator.
- Disable root SSH login where operationally possible.
- Disable password authentication only after key login has been tested in a separate session.
- Restrict SSH access at the firewall or through an access network when practical.

## Keep packages updated

Refresh package metadata and apply security updates regularly:

```bash
sudo apt update
sudo apt upgrade
```

For unattended security updates, review `unattended-upgrades` and its allowed origins rather than assuming that automatic updates are enabled.

## Reduce network exposure

Only expose ports required by the server's role. Check listening sockets and compare them with the intended service inventory:

```bash
sudo ss -tulpn
systemctl list-units --type=service --state=running
```

Use the host firewall to restrict incoming access.

## Use host security controls

- Keep AppArmor enabled where supported and investigate denials rather than disabling profiles as a first response.
- Keep security-sensitive configuration files writable only by administrators.
- Review authentication and service logs.
- Use TLS for services that transmit sensitive data.



## Protect backups and secrets

- Encrypt backups that contain sensitive information.
- Keep at least one recovery copy separate from the server being protected.
- Restrict access to private keys, API tokens, database credentials, and backup repositories.
- Do not store secrets in shell history, public repositories, or world-readable configuration files.
- Test restoration; an untested backup is not a reliable recovery plan.

## A short review checklist

```text
[ ] Only required services are enabled and listening.
[ ] Administrator access uses named accounts and sudo.
[ ] SSH access is restricted and key authentication works.
[ ] Security updates are applied on a defined schedule.
[ ] Firewall rules match the intended exposed services.
[ ] AppArmor and other host controls are active where applicable.
[ ] Logs are retained and reviewed.
[ ] Backups are protected and restore-tested.
```

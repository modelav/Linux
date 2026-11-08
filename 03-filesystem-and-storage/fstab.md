# `/etc/fstab`

`/etc/fstab` defines filesystems and swap areas that should be mounted or activated by the system. It is commonly used to make data disks and logical volumes available after reboot.

## File format

Each non-comment line contains six fields:

```text
<spec>  <mountpoint>  <type>  <options>  <dump>  <pass>
```

| Field | Meaning |
|---|---|
| `spec` | Device, UUID, label, or other filesystem source. |
| `mountpoint` | Directory where the filesystem is mounted; use `none` for swap. |
| `type` | Filesystem type, such as `ext4`, `xfs`, or `swap`. |
| `options` | Comma-separated mount or activation options. |
| `dump` | Legacy `dump` backup utility field; usually `0`. |
| `pass` | Filesystem check order at boot: `1` for the root filesystem, `2` for other filesystems, `0` to skip the check. |

## Use UUIDs

Find filesystem UUIDs:

```bash
lsblk -f
sudo blkid
```

Example entry for an ext4 data filesystem:

```fstab
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /srv/data  ext4  defaults  0  2
```

Use the real UUID from the server. Do not copy the placeholder UUID.

## Common options

| Option | Meaning |
|---|---|
| `defaults` | Use the filesystem's default options. |
| `noatime` | Avoid updating access times on file reads (workload-dependent). |
| `ro` | Mount read-only. |
| `rw` | Mount read-write. |
| `nofail` | Do not make boot fail solely because this mount is unavailable. Use only for genuinely optional storage. |
| `noauto` | Do not mount automatically at boot or through `mount -a`. |
| `x-systemd.automount` | Let systemd mount the filesystem when its mount point is first accessed. |
| `_netdev` | Tell mount/systemd that the filesystem depends on network availability. |

Be careful with `nofail`: if an expected data disk is missing, applications may write into the ordinary directory underneath the mount point instead. That can fill the root filesystem and hide the missing disk.

## Edit and validate `fstab`

Back up the file before editing it:

```bash
sudo cp -a /etc/fstab /etc/fstab.bak
sudoedit /etc/fstab
```

Validate the entries, reload systemd's generated units, and test mounts:

```bash
sudo findmnt --verify --verbose
sudo systemctl daemon-reload
sudo mount -a
findmnt --target /srv/data
```

Check the output and exit status after each command. `mount -a` attempts to mount applicable entries in `fstab`; it is a useful test, but it does not prove that every future boot condition will be correct. Do not reboot until the file has been checked and required mounts are present.

If the server fails to boot because of an incorrect entry, use the console or rescue environment to correct `/etc/fstab`.

## Swap entry example

A swap file is commonly referenced by its path:

```fstab
/swapfile  none  swap  sw  0  0
```

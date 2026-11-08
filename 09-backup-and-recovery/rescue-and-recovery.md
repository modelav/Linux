# Rescue and recovery

Rescue environments help recover a server that cannot boot normally, has a broken bootloader, or needs offline filesystem repair. Before making changes, identify the installed system's disk layout and preserve important data where possible.

## Prepare a recovery path

For physical servers, keep trusted Debian installation or live media available. For virtual machines and cloud servers, know how to use the provider's console, rescue image, or recovery environment. Remote SSH access is not enough if the machine cannot boot or networking is broken.

## Identify disks and filesystems

From the rescue environment:

```bash
lsblk -f
blkid
findmnt
```

Identify the correct root filesystem and any separate `/boot`, EFI System Partition, data, LVM, RAID, or encrypted volumes. Never assume device names such as `/dev/sda` remain the same between boots or machines.

## Activate storage layers

For LVM, inspect and activate the volume group:

```bash
sudo pvscan
sudo vgscan
sudo vgchange -ay
sudo lvs -a -o +devices
```

For encrypted storage, open the correct LUKS device with the recovery passphrase:

```bash
sudo cryptsetup open /dev/<device> cryptroot
```

For software RAID, inspect the array state and assemble only the intended arrays. Storage activation commands depend on the installed layout; consult the LVM, encryption, or RAID guide before proceeding.

## Mount the installed system

After identifying the root filesystem, mount it under a temporary mount point:

```bash
sudo mkdir -p /mnt/debian
sudo mount /dev/<root-device> /mnt/debian
```

Mount separate boot filesystems when the installed system uses them:

```bash
sudo mount /dev/<boot-device> /mnt/debian/boot
sudo mount /dev/<efi-device> /mnt/debian/boot/efi
```

## Prepare a chroot

A chroot lets you run commands with the installed system's root directory as `/`. Bind-mount system interfaces before entering it:

```bash
sudo mount --rbind /dev /mnt/debian/dev
sudo mount --make-rslave /mnt/debian/dev
sudo mount -t proc proc /mnt/debian/proc
sudo mount --rbind /sys /mnt/debian/sys
sudo mount --make-rslave /mnt/debian/sys
sudo mount --rbind /run /mnt/debian/run
sudo mount --make-rslave /mnt/debian/run
sudo chroot /mnt/debian /bin/bash
```

Inside the chroot, verify the mounted `/boot` and `/boot/efi` filesystems before changing kernels or the bootloader. Network access, DNS, and device access may not be identical to a normally booted system.

When finished, exit the chroot and unmount in reverse order. Check for processes using the mount points before attempting to unmount them.

## Repair common boot problems

Depending on the system and installed packages, common recovery commands inside the chroot include:

```bash
update-initramfs -u -k all
update-grub
```

These commands regenerate initramfs images and GRUB configuration. They do not repair every bootloader installation problem. Reinstalling GRUB, updating EFI boot entries, Secure Boot configuration, or repairing a separate `/boot` filesystem requires steps specific to the installed firmware mode and disk layout.

Check `/etc/fstab` for incorrect UUIDs or mount options. Use `blkid` to confirm UUIDs and `findmnt --verify` or `mount -a` to validate entries carefully. A bad root or boot mount configuration can prevent startup.

## Filesystem checks

Filesystem repair tools should normally be run on an unmounted filesystem. First inspect the filesystem type and ensure it is not mounted:

```bash
lsblk -f
findmnt --source /dev/<device>
```

For ext2/ext3/ext4, `e2fsck` is the usual checker:

```bash
sudo e2fsck -f /dev/<device>
```

Do not run `fsck` or `e2fsck` against a mounted read-write filesystem. For XFS, use `xfs_repair` while the filesystem is unmounted; the repair procedure and log-recovery behavior differ from ext filesystems. For example: `sudo xfs_repair /dev/<device>`. Never run repair commands against an uncertain device.

## Enter systemd rescue mode on a working system

If the system still boots and you have local or out-of-band console access:

```bash
sudo systemctl rescue
```

This switches the running system to rescue mode and can stop services. Do not use it casually on a remote-only production server; it may terminate the services or network access needed to reconnect.

## Recovery checklist

- Identify the actual root and boot devices.
- Check LVM, encryption, and RAID state before mounting filesystems.
- Mount the correct `/boot` and EFI filesystems before bootloader work.
- Back up important data before filesystem repair or bootloader changes.
- Use filesystem-specific repair tools and run them offline.
- Verify `/etc/fstab`, initramfs, kernel, and bootloader configuration.
- Reboot only when you have console access and a clear validation plan.

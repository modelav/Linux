# Disks and partitions

This guide covers identifying storage devices and viewing or creating disk partitions. Partitioning changes the layout of a disk and can make existing data inaccessible.

## Inspect disks

### `lsblk`

Show disks, partitions, sizes, filesystem types, UUIDs, and mount points.

```bash
lsblk
lsblk -f
lsblk -o NAME,MODEL,SIZE,TYPE,FSTYPE,UUID,MOUNTPOINTS
```

### `fdisk`

List partition tables or open an interactive partition editor.

```bash
sudo fdisk -l
sudo fdisk /dev/sdb
```

Inside interactive `fdisk`, common keys are:

| Key | Meaning |
|---|---|
| `p` | Print the current partition table. |
| `g` | Create a new empty GPT partition table in memory. |
| `n` | Add a partition. |
| `d` | Delete a partition from the in-memory table. |
| `w` | Write changes to disk and exit. |
| `q` | Quit without writing pending changes. |
| `m` | Show help. |


### `parted`

Inspect GPT and MBR partition tables, including disks larger than 2 TiB with GPT.

```bash
sudo parted --list
sudo parted /dev/sdb print
```

Useful interactive commands include `print`, `unit`, `mklabel`, `mkpart`, `resizepart`, and `quit`.

## Create a partition

The following is an example for a **new, empty disk** `/dev/sdb`. Replace the device name only after checking `lsblk` and confirming that the disk contains no data you need.

```bash
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart data ext4 1MiB 100%
sudo partprobe /dev/sdb
sudo udevadm settle
lsblk -f /dev/sdb
```

`mklabel gpt` replaces the disk's partition table. `mkpart` creates a partition entry; it does **not** create an ext4 filesystem. Filesystem creation is a separate step covered in [Filesystems](filesystems.md).

> **Warning:** Selecting the wrong disk or replacing an existing partition table can destroy access to data. Confirm the device path, size, model, mount points, RAID/LVM membership, and backups before making partition changes. Never use these example commands on a disk merely because its name looks like `/dev/sdb`.

## Refresh and verify the partition view

After changing a partition table, ask the kernel to reread it and verify what the system sees:

```bash
sudo partprobe /dev/sdb
sudo udevadm settle
lsblk -f /dev/sdb
sudo fdisk -l /dev/sdb
```

If the kernel cannot reread a busy disk's partition table, a reboot may be required before the new layout is available. Do not continue to filesystem creation until the intended partition is visible and verified.


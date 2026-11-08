# Logical Volume Management (LVM)

LVM adds a flexible storage layer between physical disks and filesystems. It lets you combine storage devices into pools and allocate logical volumes that can be extended as capacity is added.

## Overview

| Component | Meaning |
|---|---|
| PV — Physical Volume | A disk or partition initialized for LVM. |
| VG — Volume Group | A storage pool made from one or more PVs. |
| LV — Logical Volume | A virtual block device allocated from a VG. |

A typical layout is:

```text
/dev/sdb ── PV ──┐
                 ├── VG: vg_data ── LV: lv_data ── ext4 ── /srv/data
/dev/sdc ── PV ──┘
```

## Install and inspect LVM

Install the LVM tools if they are not already present:

```bash
sudo apt update
sudo apt install lvm2
```

Show physical volumes, volume groups, and logical volumes:

```bash
sudo pvs
sudo vgs
sudo lvs
sudo lvs -a -o +devices
```

| Command | Shows |
|---|---|
| `pvs` / `pvdisplay` | Physical volumes. |
| `vgs` / `vgdisplay` | Volume groups and free space. |
| `lvs` / `lvdisplay` | Logical volumes. |

## Create a PV, VG, and LV

This example uses a **new, empty disk** at `/dev/sdb`. It initializes the whole disk for LVM; it does not create a partition first. You can also use a dedicated partition such as `/dev/sdb1`.

First confirm the target device:

```bash
lsblk -f
findmnt
```

Then create the LVM layers:

```bash
sudo pvcreate /dev/sdb
sudo vgcreate vg_data /dev/sdb
sudo lvcreate -L 100G -n lv_data vg_data
```

Create a filesystem and mount it:

```bash
sudo mkfs.ext4 /dev/vg_data/lv_data
sudo mkdir -p /srv/data
sudo mount /dev/vg_data/lv_data /srv/data
findmnt /srv/data
```

> **Warning:** `pvcreate` writes LVM metadata to the selected device, and `mkfs.ext4` creates a new filesystem. Both can destroy access to existing data. Confirm the exact device and take backups before proceeding.

To mount the LV automatically after reboot, add its UUID to `/etc/fstab`.

## Extend a logical volume

A VG must have free extents before an LV can be extended. One way to add capacity is to initialize another empty disk and add it to the VG:

```bash
sudo pvcreate /dev/sdc
sudo vgextend vg_data /dev/sdc
sudo vgs
```

Extend the LV and its filesystem together:

```bash
sudo lvextend -r -l +100%FREE /dev/vg_data/lv_data
```

| Option | Meaning |
|---|---|
| `-L +20G` | Add 20 GiB to the LV. |
| `-l +100%FREE` | Add all currently free extents in the VG. |
| `-r` | Also resize the filesystem where supported. |

Check the result:

```bash
sudo lvs
df -hT /srv/data
```

`lvextend -r` delegates filesystem resizing to the available filesystem tools. If it is not supported for the filesystem or configuration, extend the LV and filesystem using the appropriate separate procedure.

## Grow a PV after expanding its device

If the underlying disk or partition has been enlarged, tell LVM to recognize the new size:

```bash
sudo pvresize /dev/sdb
sudo pvs
sudo vgs
```

If the PV is a partition, the partition itself must be enlarged first. Verify the partition table and device layout before running `pvresize`.

## Move data off a PV

`pvmove` relocates allocated extents from one PV to another PV in the same VG:

```bash
sudo pvmove /dev/sdb /dev/sdc
```

Only remove the old PV from the VG after verifying that it no longer contains allocated extents:

```bash
sudo pvs -o+pv_used
sudo vgreduce vg_data /dev/sdb
```

Removing a PV or VG without checking its contents can destroy access to logical volumes. Confirm backups and the LVM layout first.

## Snapshots

A classic LVM snapshot records the state of an LV at the time the snapshot is created and stores changed blocks in a separate copy-on-write area:

```bash
sudo lvcreate --snapshot --size 10G --name lv_data_snap vg_data/lv_data
sudo lvs
```

The snapshot's copy-on-write area must be large enough for changes made while it exists. If it fills, the snapshot can become unusable. Snapshots are generally temporary tools for backup workflows or point-in-time operations; they are not a replacement for independent backups.

Remove a snapshot when it is no longer needed:

```bash
sudo lvremove vg_data/lv_data_snap
```

`lvremove` permanently removes the selected logical volume or snapshot.

## Shrinking logical volumes

Shrinking is more dangerous than extending. The filesystem must be reduced **before** the LV is reduced. The exact steps depend on the filesystem and may require unmounting it. XFS cannot be shrunk with the standard XFS tools. Do not use `lvreduce` on a filesystem that has not first been safely shrunk and checked; doing so can truncate live filesystem data.

## Back up LVM metadata

LVM maintains metadata backups for volume groups, but it is useful to create an explicit backup before major changes:

```bash
sudo vgcfgbackup
```

This backs up LVM configuration metadata, not the data stored inside logical volumes.


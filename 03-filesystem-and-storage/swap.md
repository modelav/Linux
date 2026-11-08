# Swap

Swap is disk space that Linux can use to store memory pages that are not currently needed in RAM. It can help absorb memory pressure, but it is much slower than RAM and does not replace sufficient physical memory.

## Inspect swap

```bash
free -h
swapon --show
cat /proc/swaps
```

`free -h` summarizes RAM and swap usage. `swapon --show` lists active swap areas.

## Enable a swap file

This example creates a 2 GiB swap file at `/swapfile`. It is intended for a filesystem such as ext4 that supports the usual swap-file procedure.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show
```

| Command / option | Meaning |
|---|---|
| `fallocate -l 2G` | Allocate a file of the requested size. |
| `chmod 600` | Allow only root to read and write the swap file. |
| `mkswap` | Set up the file as swap space. |
| `swapon` | Activate the swap area. |

> **Note:** Swap-file requirements vary by filesystem. Btrfs has additional constraints and a dedicated swap-file creation procedure;

## Make swap persistent

Add this line to `/etc/fstab`:

```fstab
/swapfile  none  swap  sw  0  0
```

Then verify the configuration and activate the entry:

```bash
sudo findmnt --verify --verbose
sudo swapon -a
swapon --show
```

## Disable or remove a swap file

Deactivate it first:

```bash
sudo swapoff /swapfile
```

Remove its `/etc/fstab` entry, then remove the file:

```bash
sudo rm /swapfile
```

`swapoff` may require enough free RAM to move swapped pages back into memory. If memory is under pressure, it may fail or worsen the situation. Check memory use and available RAM before deactivating an active swap area.

## Adjust swappiness

`vm.swappiness` influences the kernel's tendency to use swap; it is not a percentage of RAM that will be swapped.

Show the current value:

```bash
cat /proc/sys/vm/swappiness
```

Temporarily change it until reboot:

```bash
sudo sysctl vm.swappiness=10
```

For a persistent setting, add a file such as `/etc/sysctl.d/99-swappiness.conf` containing:

```ini
vm.swappiness = 10
```

Apply sysctl settings:

```bash
sudo sysctl --system
```

Choose values based on workload and monitoring rather than copying a tuning value without testing.


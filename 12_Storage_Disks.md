# Linux Storage & Disk Management — Partitions, Filesystems, and LVM

## Objective
Master disk management: understand storage devices, partitions, filesystems, mounting, and Logical Volume Manager (LVM).

---

## 1) Storage Device Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        LINUX STORAGE HIERARCHY                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Physical Storage   →   Partition/Volume   →   Filesystem   →   Mount Point        │
│                                                                                     │
│   ┌─────────────┐       ┌─────────────┐       ┌──────────┐     ┌────────────┐       │
│   │ /dev/sda    │──────▶│ /dev/sda1   │──────▶│  ext4    │────▶│     /      │       │
│   │ (HDD/SSD)   │       │ /dev/sda2   │       │  xfs     │     │ /home      │       │
│   └─────────────┘       └─────────────┘       │  btrfs   │     │ /var       │       │
│                                               └──────────┘     └────────────┘       │
│   ┌─────────────┐       ┌─────────────┐                                             │
│   │ /dev/nvme0n1│──────▶│ /dev/nvme0n1p1                                            │
│   │ (NVMe SSD)  │       │ /dev/nvme0n1p2                                            │
│   └─────────────┘       └─────────────┘                                             │
│                                                                                     │
│   ┌─────────────┐       ┌─────────────┐                                             │
│   │ /dev/vda    │──────▶│ /dev/vda1   │       (Virtual disks in VMs)                │
│   │ (VirtIO)    │       │ /dev/vda2   │                                             │
│   └─────────────┘       └─────────────┘                                             │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Device Naming

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         DEVICE NAMING CONVENTIONS                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Device Type     │ Pattern           │ Example                                     │
│   ────────────────┼───────────────────┼────────────────────────────────             │
│   SATA/SCSI/SAS   │ /dev/sd[a-z]      │ /dev/sda, /dev/sdb                          │
│   NVMe            │ /dev/nvme[0-9]n[1-9] │ /dev/nvme0n1, /dev/nvme1n1               │
│   Virtual (KVM)   │ /dev/vd[a-z]      │ /dev/vda, /dev/vdb                          │
│   CD/DVD          │ /dev/sr[0-9]      │ /dev/sr0                                    │
│   Floppy          │ /dev/fd[0-9]      │ /dev/fd0                                    │
│   Loop devices    │ /dev/loop[0-9]    │ /dev/loop0                                  │
│                                                                                     │
│   Partitions:                                                                       │
│   /dev/sda1, /dev/sda2, ...          (SATA/SCSI)                                    │
│   /dev/nvme0n1p1, /dev/nvme0n1p2     (NVMe)                                         │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) Viewing Disk Information

### Block Devices

```bash
# List block devices
lsblk                              # Tree view of disks and partitions
lsblk -f                           # Include filesystem and mount info
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT

# Detailed disk info
sudo fdisk -l                      # All disks and partitions
sudo fdisk -l /dev/sda             # Specific disk

# Disk info with partprobe
sudo partprobe -s                  # Inform kernel of partition changes

# Hardware details
sudo hdparm -I /dev/sda            # Detailed drive info (HDD/SSD)
sudo smartctl -a /dev/sda          # SMART data (health)
```

### Disk Space Usage

```bash
# Filesystem space
df -h                              # Human readable
df -Th                             # Include filesystem type
df -h /                            # Specific mount point
df -i                              # Inode usage

# Directory space
du -sh /var                        # Size of directory
du -sh /var/*                      # Size of subdirectories
du -h --max-depth=1 /home          # One level deep
du -h / --max-depth=1 2>/dev/null | sort -h  # Largest directories

# ncdu (interactive disk usage - install: apt install ncdu)
ncdu /                             # Interactive browser
ncdu -x /                          # Stay on same filesystem
```

---

## 3) Partition Tables: MBR vs GPT

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          MBR vs GPT                                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Feature              │ MBR (Master Boot Record) │ GPT (GUID Partition Table)      │
│   ─────────────────────┼──────────────────────────┼────────────────────────────     │
│   Max disk size        │ 2 TB                     │ 9.4 ZB (zettabytes)             │
│   Max partitions       │ 4 primary (or 3+extended)│ 128 by default                  │
│   Boot mode            │ BIOS (Legacy)            │ UEFI                            │
│   Data integrity       │ No redundancy            │ CRC32 checksums                 │
│   Backup               │ No partition table backup│ Backup at end of disk           │
│                                                                                     │
│   RECOMMENDATION: Use GPT for modern systems                                        │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   MBR Structure:                      GPT Structure:                                │
│   ┌────────────────┐                  ┌────────────────┐                            │
│   │ MBR (512 bytes)│                  │ Protective MBR │                            │
│   ├────────────────┤                  ├────────────────┤                            │
│   │ Partition 1    │                  │ GPT Header     │                            │
│   ├────────────────┤                  ├────────────────┤                            │
│   │ Partition 2    │                  │ Partition      │                            │
│   ├────────────────┤                  │ Entries        │                            │
│   │ Partition 3    │                  ├────────────────┤                            │
│   ├────────────────┤                  │ Partition 1    │                            │
│   │ Partition 4    │                  │ Partition 2    │                            │
│   └────────────────┘                  │ ...            │                            │
│                                       ├────────────────┤                            │
│                                       │ Backup Entries │                            │
│                                       ├────────────────┤                            │
│                                       │ Backup GPT Hdr │                            │
│                                       └────────────────┘                            │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4) Partitioning Tools

### fdisk (MBR and GPT)

```bash
# Interactive mode
sudo fdisk /dev/sdb

# Common fdisk commands:
# p   Print partition table
# n   Create new partition
# d   Delete partition
# t   Change partition type
# w   Write changes and exit
# q   Quit without saving

# Non-interactive (scripted)
echo -e "n\np\n1\n\n\nw" | sudo fdisk /dev/sdb
```

### gdisk (GPT)

```bash
# GPT-specific tool
sudo gdisk /dev/sdb

# Commands similar to fdisk:
# p   Print partition table
# n   New partition
# d   Delete partition
# w   Write and exit
# ?   Help
```

### parted (MBR and GPT)

```bash
# Interactive mode
sudo parted /dev/sdb

# Non-interactive
sudo parted /dev/sdb mklabel gpt              # Create GPT table
sudo parted /dev/sdb mkpart primary ext4 0% 100%  # Create partition
sudo parted /dev/sdb print                    # Show partitions
```

---

## 5) Filesystems

### Common Filesystems

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          FILESYSTEM COMPARISON                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Filesystem │ Max File │ Max Volume │ Journal │ Features                           │
│   ───────────┼──────────┼────────────┼─────────┼─────────────────────               │
│   ext4       │ 16 TB    │ 1 EB       │ Yes     │ Default Linux, stable, mature      │
│   xfs        │ 8 EB     │ 8 EB       │ Yes     │ High performance, enterprise       │
│   btrfs      │ 16 EB    │ 16 EB      │ CoW     │ Snapshots, compression             │
│   zfs        │ 16 EB    │ 256 ZB     │ CoW     │ Advanced features, Oracle license  │
│   ntfs       │ 16 TB    │ 256 TB     │ Yes     │ Windows compatibility              │
│   fat32      │ 4 GB     │ 2 TB       │ No      │ USB drives, universal              │
│   exfat      │ 16 EB    │ 128 PB     │ No      │ Large files on USB                 │
│                                                                                     │
│   RECOMMENDATIONS:                                                                  │
│   • Root filesystem: ext4 (stable) or xfs (performance)                             │
│   • Database server: xfs                                                            │
│   • Snapshots needed: btrfs or zfs                                                  │
│   • USB/external: exfat (cross-platform) or ext4 (Linux-only)                       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Creating Filesystems

```bash
# ext4 (most common)
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L "my_label" /dev/sdb1        # With label

# xfs
sudo mkfs.xfs /dev/sdb1
sudo mkfs.xfs -f /dev/sdb1                    # Force (overwrite)

# btrfs
sudo mkfs.btrfs /dev/sdb1

# fat32 / exfat
sudo mkfs.vfat -F 32 /dev/sdb1                # FAT32
sudo mkfs.exfat /dev/sdb1                     # exFAT

# Check filesystem type
blkid /dev/sdb1
lsblk -f /dev/sdb1
```

### Filesystem Checks and Repair

```bash
# Check filesystem (unmounted!)
sudo fsck /dev/sdb1                           # Auto-detect type
sudo fsck.ext4 /dev/sdb1                      # Specific type
sudo fsck -y /dev/sdb1                        # Auto-yes to repairs

# XFS check
sudo xfs_repair /dev/sdb1

# Force check on next boot
sudo touch /forcefsck

# Check without fixing
sudo fsck -n /dev/sdb1
```

---

## 6) Mounting Filesystems

### Manual Mounting

```bash
# Basic mount
sudo mount /dev/sdb1 /mnt
sudo mount /dev/sdb1 /mnt/data

# Mount with options
sudo mount -o ro /dev/sdb1 /mnt              # Read-only
sudo mount -o rw,noexec /dev/sdb1 /mnt       # No execution
sudo mount -o uid=1000,gid=1000 /dev/sdb1 /mnt  # Ownership (FAT/NTFS)

# Mount by label or UUID
sudo mount LABEL=my_label /mnt
sudo mount UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt

# Unmount
sudo umount /mnt
sudo umount /dev/sdb1
sudo umount -l /mnt                          # Lazy unmount (if busy)
sudo umount -f /mnt                          # Force (NFS)

# Show all mounts
mount | grep sdb
cat /proc/mounts
findmnt
findmnt -t ext4                              # Specific type
```

### Persistent Mounting with /etc/fstab

```bash
# /etc/fstab format:
# <device>           <mount_point>  <fstype>  <options>           <dump>  <pass>

# Example entries:
UUID=xxx-xxx-xxx     /              ext4      defaults            0       1
UUID=yyy-yyy-yyy     /home          ext4      defaults            0       2
UUID=zzz-zzz-zzz     /data          xfs       defaults,noatime    0       2
/dev/sdb1            /mnt/backup    ext4      defaults,nofail     0       2
LABEL=USB            /mnt/usb       auto      defaults,nofail     0       0

# Swap
/swapfile            none           swap      sw                  0       0

# Temporary filesystem
tmpfs                /tmp           tmpfs     defaults,size=2G    0       0
```

### fstab Options Explained

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          FSTAB OPTIONS                                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Option      │ Description                                                         │
│   ────────────┼─────────────────────────────────────────────────────────────        │
│   defaults    │ rw,suid,dev,exec,auto,nouser,async                                  │
│   auto        │ Mount automatically at boot                                         │
│   noauto      │ Don't mount at boot (manual mount only)                             │
│   rw / ro     │ Read-write / read-only                                              │
│   exec/noexec │ Allow/disallow execution of binaries                                │
│   suid/nosuid │ Allow/disallow setuid bits                                          │
│   user        │ Allow users to mount                                                │
│   nouser      │ Only root can mount                                                 │
│   nofail      │ Don't fail boot if device missing                                   │
│   noatime     │ Don't update access times (performance)                             │
│   relatime    │ Update atime occasionally (default)                                 │
│   sync/async  │ Synchronous/asynchronous I/O                                        │
│                                                                                     │
│   dump (5th column): 0 = don't backup, 1 = backup                                   │
│   pass (6th column): fsck order (0=skip, 1=root, 2=others)                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Test fstab

```bash
# Test mount all from fstab
sudo mount -a

# Check for errors before reboot
sudo findmnt --verify

# Show what would be mounted
sudo mount -fav
```

---

## 7) Swap Space

### Create Swap

```bash
# Swap file (recommended)
sudo dd if=/dev/zero of=/swapfile bs=1G count=4    # 4GB
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Add to fstab for persistence
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Swap partition
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
```

### Manage Swap

```bash
# View swap
free -h
swapon --show
cat /proc/swaps

# Disable swap
sudo swapoff /swapfile
sudo swapoff -a                    # All swap

# Swappiness (how aggressively to use swap)
cat /proc/sys/vm/swappiness        # Default: 60
sudo sysctl vm.swappiness=10       # Temporary
# Persistent: add "vm.swappiness=10" to /etc/sysctl.conf
```

---

## 8) LVM — Logical Volume Manager

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          LVM ARCHITECTURE                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│    Physical      Physical        Volume          Logical        Filesystem          │
│    Devices       Volumes         Group           Volumes                            │
│                                                                                     │
│   ┌────────┐    ┌────────┐                                                          │
│   │/dev/sdb│───▶│   PV   │──┐                                                       │
│   └────────┘    └────────┘  │      ┌─────────┐    ┌────────┐    ┌────────┐          │
│                              ├────▶│         │───▶│   LV   │───▶│  ext4  │──▶ /home │
│   ┌────────┐    ┌────────┐  │      │   VG    │    │  home  │    └────────┘          │
│   │/dev/sdc│───▶│   PV   │──┤      │         │    └────────┘                        │
│   └────────┘    └────────┘  │      │ "data"  │                                      │
│                              │      │         │    ┌────────┐    ┌────────┐          │
│   ┌────────┐    ┌────────┐  │      │         │───▶│   LV   │───▶│  xfs   │──▶ /var  │
│   │/dev/sdd│───▶│   PV   │──┘      │         │    │  var   │    └────────┘          │
│   └────────┘    └────────┘         └─────────┘    └────────┘                        │
│                                                                                     │
│   Benefits of LVM:                                                                  │
│   • Resize volumes without unmounting (grow)                                        │
│   • Span multiple physical disks                                                    │
│   • Snapshots for backups                                                           │
│   • Easy disk replacement                                                           │
│   • Thin provisioning                                                               │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Creating LVM

```bash
# 1. Create Physical Volumes (PV)
sudo pvcreate /dev/sdb /dev/sdc

# View PVs
sudo pvs
sudo pvdisplay

# 2. Create Volume Group (VG)
sudo vgcreate myvg /dev/sdb /dev/sdc

# View VGs
sudo vgs
sudo vgdisplay

# 3. Create Logical Volumes (LV)
sudo lvcreate -n lv_data -L 50G myvg         # 50GB
sudo lvcreate -n lv_backup -l 100%FREE myvg  # Use remaining space

# View LVs
sudo lvs
sudo lvdisplay

# 4. Create filesystem and mount
sudo mkfs.ext4 /dev/myvg/lv_data
sudo mkdir /data
sudo mount /dev/myvg/lv_data /data

# Add to fstab
# /dev/myvg/lv_data  /data  ext4  defaults  0  2
```

### Extending LVM

```bash
# Extend Volume Group (add new disk)
sudo pvcreate /dev/sdd
sudo vgextend myvg /dev/sdd

# Extend Logical Volume
sudo lvextend -L +20G /dev/myvg/lv_data      # Add 20GB
sudo lvextend -l +100%FREE /dev/myvg/lv_data # Use all free space

# Resize filesystem (ext4)
sudo resize2fs /dev/myvg/lv_data             # Grow to fill LV

# Or in one command
sudo lvextend -r -L +20G /dev/myvg/lv_data   # -r resizes filesystem

# For XFS
sudo xfs_growfs /data                         # XFS uses mount point
```

### Shrinking LVM (ext4 only, be careful!)

```bash
# UNMOUNT FIRST!
sudo umount /data

# Check filesystem
sudo e2fsck -f /dev/myvg/lv_data

# Shrink filesystem first
sudo resize2fs /dev/myvg/lv_data 40G

# Then shrink LV
sudo lvreduce -L 40G /dev/myvg/lv_data

# Remount
sudo mount /dev/myvg/lv_data /data
```

### LVM Snapshots

```bash
# Create snapshot
sudo lvcreate -s -n snap_data -L 5G /dev/myvg/lv_data

# Mount snapshot
sudo mkdir /mnt/snapshot
sudo mount -o ro /dev/myvg/snap_data /mnt/snapshot

# Remove snapshot
sudo lvremove /dev/myvg/snap_data
```

---

## 9) Practical Exercises

### Exercise 1: Partition and Format New Disk
```bash
# 1. Identify new disk
lsblk

# 2. Create partition
sudo fdisk /dev/sdb
# n, p, 1, Enter, Enter, w

# 3. Format
sudo mkfs.ext4 /dev/sdb1

# 4. Mount
sudo mkdir /mnt/newdisk
sudo mount /dev/sdb1 /mnt/newdisk

# 5. Make permanent
UUID=$(blkid -s UUID -o value /dev/sdb1)
echo "UUID=$UUID /mnt/newdisk ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

### Exercise 2: Create Swap File
```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```

### Exercise 3: Basic LVM Setup
```bash
# Create PV, VG, LV
sudo pvcreate /dev/sdc
sudo vgcreate testvg /dev/sdc
sudo lvcreate -n testlv -L 5G testvg
sudo mkfs.ext4 /dev/testvg/testlv
sudo mkdir /test
sudo mount /dev/testvg/testlv /test
```

---

## 10) Quick Reference

| Task | Command |
|------|---------|
| List block devices | `lsblk` |
| Disk info | `sudo fdisk -l` |
| Check disk space | `df -h` |
| Directory size | `du -sh /path` |
| Partition disk | `sudo fdisk /dev/sdb` |
| Create ext4 | `sudo mkfs.ext4 /dev/sdb1` |
| Mount | `sudo mount /dev/sdb1 /mnt` |
| Unmount | `sudo umount /mnt` |
| Show mounts | `findmnt` or `mount` |
| Get UUID | `blkid /dev/sdb1` |
| Check filesystem | `sudo fsck /dev/sdb1` |
| Create PV | `sudo pvcreate /dev/sdb` |
| Create VG | `sudo vgcreate vgname /dev/sdb` |
| Create LV | `sudo lvcreate -n lvname -L 10G vgname` |
| Extend LV | `sudo lvextend -r -L +5G /dev/vg/lv` |
| Show swap | `swapon --show` |

---

## 11) Interview Questions

**Q1: Difference between MBR and GPT?**
> MBR supports up to 2TB disks and 4 primary partitions, uses BIOS. GPT supports much larger disks, 128+ partitions, and uses UEFI. GPT also has backup partition tables.

**Q2: How do you make a mount persistent across reboots?**
> Add an entry to `/etc/fstab` with the UUID, mount point, filesystem type, and options. Test with `mount -a` before rebooting.

**Q3: What is LVM and when would you use it?**
> LVM (Logical Volume Manager) abstracts physical storage, allowing dynamic resizing, spanning multiple disks, and snapshots. Use it when you need flexible storage management.

**Q4: How do you extend an ext4 filesystem on LVM?**
> `sudo lvextend -r -L +10G /dev/vgname/lvname`. The `-r` flag resizes the filesystem automatically. Or run `resize2fs` separately after extending the LV.

**Q5: What's the correct order for shrinking an LVM volume?**
> Unmount, run fsck, shrink the filesystem with resize2fs, then shrink the LV with lvreduce. Never shrink the LV first—you'll lose data.

---

*Next: [13_Shell_Scripting.md](13_Shell_Scripting.md) — Bash scripting and automation.*

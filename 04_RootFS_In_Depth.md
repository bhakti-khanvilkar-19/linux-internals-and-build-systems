# Root Filesystem (RootFS) — In-Depth Guide

## Objective
Understand the Linux root filesystem in detail: its structure, purpose, types, how it's mounted, and how it differs between desktop and embedded systems.

> **Interactive Diagrams**: 
> - [diagrams/rootfs_structure.drawio](diagrams/rootfs_structure.drawio) — Visual hierarchy of the root filesystem
> - [diagrams/distro_family_tree.drawio](diagrams/distro_family_tree.drawio) — Distribution relationships and package management

---

## 1) What is the Root Filesystem (RootFS)?

### Simple Definition
The **root filesystem (rootfs)** is the filesystem mounted at `/` — the top of the entire directory tree. Every file and directory in Linux hangs off this single root point.

### Technical Definition
RootFS is:
- A hierarchical collection of directories and files
- The first filesystem the kernel mounts after boot
- Contains everything needed to run the system: binaries, libraries, configuration, and device nodes
- Can reside on disk (ext4, btrfs), network (NFS), RAM (tmpfs/initramfs), or flash (squashfs, ubifs)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           THE ROOT FILESYSTEM CONCEPT                               │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Everything in Linux is a file, and ALL files live under root (/)                  │
│                                                                                     │
│                                    /  (root)                                        │
│                                    │                                                │
│          ┌─────────┬─────────┬─────┴─────┬─────────┬─────────┐                      │
│          │         │         │           │         │         │                      │
│         bin      etc       usr         var      home      dev                       │
│          │         │         │           │         │         │                      │
│       Commands  Config   Programs     Logs     Users    Devices                     │
│                                                                                     │
│   This structure is called FHS (Filesystem Hierarchy Standard)                      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) Why is RootFS Important?

### Without RootFS, the System Cannot Run

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           KERNEL NEEDS ROOTFS                                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Kernel boots → "I need to run /sbin/init"                                         │
│                        │                                                            │
│                        ▼                                                            │
│   Where is /sbin/init? → It's in the rootfs!                                        │
│                        │                                                            │
│                        ▼                                                            │
│   If no rootfs → KERNEL PANIC: Unable to mount root fs                              │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Common kernel panic messages:                                                     │
│   • "VFS: Cannot open root device"                                                  │
│   • "Kernel panic - not syncing: VFS: Unable to mount root fs"                      │
│   • "Please append a correct 'root=' boot option"                                   │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### RootFS Provides:
| Component | Purpose | Location in RootFS |
|-----------|---------|-------------------|
| Init system | First process (PID 1) | `/sbin/init` or `/lib/systemd/systemd` |
| C Library | All programs need it | `/lib/x86_64-linux-gnu/libc.so.6` |
| Shell | Command interpreter | `/bin/bash` or `/bin/sh` |
| Device nodes | Hardware access | `/dev/sda`, `/dev/tty`, `/dev/null` |
| Configuration | System settings | `/etc/passwd`, `/etc/fstab` |
| Kernel modules | Loadable drivers | `/lib/modules/$(uname -r)/` |

---

## 3) RootFS Directory Structure (FHS Standard)

The **Filesystem Hierarchy Standard (FHS)** defines where files should go. Most Linux distros follow this.

### Complete Directory Map

```
/                           Root of everything
│
├── bin/                    Essential user binaries (now often → /usr/bin)
│   ├── ls, cp, mv, cat     Basic commands
│   ├── bash, sh            Shells
│   └── mount, umount       Mounting tools
│
├── sbin/                   Essential system binaries (now often → /usr/sbin)
│   ├── init                Process 1
│   ├── fsck                Filesystem check
│   ├── ip, ifconfig        Network configuration
│   └── reboot, shutdown    Power management
│
├── etc/                    System configuration (Editable Text Configuration)
│   ├── passwd, shadow      User accounts
│   ├── fstab               Filesystem mount table
│   ├── hostname            System name
│   ├── network/            Network config (Debian)
│   ├── systemd/            Systemd configuration
│   └── apt/, yum.repos.d/  Package manager repos
│
├── usr/                    User programs and data (Unix System Resources)
│   ├── bin/                User binaries (applications)
│   ├── sbin/               System binaries (admin tools)
│   ├── lib/                Libraries for /usr/bin and /usr/sbin
│   ├── local/              Locally installed software
│   │   ├── bin/
│   │   ├── lib/
│   │   └── share/
│   ├── share/              Architecture-independent data
│   │   ├── man/            Manual pages
│   │   ├── doc/            Documentation
│   │   └── applications/   Desktop files
│   └── include/            C/C++ header files
│
├── var/                    Variable data (changes during operation)
│   ├── log/                System logs
│   │   ├── syslog
│   │   ├── auth.log
│   │   └── kern.log
│   ├── cache/              Application caches
│   ├── spool/              Print/mail queues
│   ├── lib/                Application state data
│   │   ├── apt/            APT package lists
│   │   └── dpkg/           DPKG database
│   ├── run/                Runtime data (→ /run)
│   └── tmp/                Persistent temp files
│
├── home/                   User home directories
│   ├── alice/
│   │   ├── .bashrc         User shell config
│   │   ├── .config/        User app config (XDG)
│   │   └── Documents/
│   └── bob/
│
├── root/                   Root user's home directory
│
├── boot/                   Boot files (kernel, initramfs, bootloader)
│   ├── vmlinuz-6.8.0       Compressed kernel
│   ├── initrd.img-6.8.0    Initial ramdisk
│   ├── config-6.8.0        Kernel config
│   ├── System.map-6.8.0    Kernel symbol table
│   └── grub/               GRUB bootloader
│       ├── grub.cfg
│       └── fonts/
│
├── dev/                    Device files (created dynamically by udev)
│   ├── sda, sda1           Block devices (disks)
│   ├── tty, tty0           Terminal devices
│   ├── null                Discard device
│   ├── zero                Infinite zeros
│   ├── random, urandom     Random number generators
│   └── pts/                Pseudo-terminals
│
├── proc/                   Process info (virtual filesystem - procfs)
│   ├── 1/                  Info about PID 1 (init)
│   ├── cpuinfo             CPU information
│   ├── meminfo             Memory information
│   ├── cmdline             Kernel boot parameters
│   └── filesystems         Supported filesystems
│
├── sys/                    Kernel/hardware info (virtual - sysfs)
│   ├── class/              Device classes
│   │   ├── net/            Network interfaces
│   │   ├── block/          Block devices
│   │   └── gpio/           GPIO pins (embedded)
│   ├── devices/            Device tree
│   └── firmware/           Firmware interfaces
│
├── lib/                    Essential libraries (now often → /usr/lib)
│   ├── x86_64-linux-gnu/   Architecture-specific libs
│   │   └── libc.so.6       GNU C Library
│   ├── modules/            Kernel modules
│   │   └── 6.8.0/          Modules for kernel version
│   └── firmware/           Device firmware blobs
│
├── lib64/                  64-bit libraries (some distros)
│
├── tmp/                    Temporary files (cleared on boot)
│
├── mnt/                    Temporary mount points
│   └── usb/                Manually mounted USB
│
├── media/                  Removable media mount points
│   └── alice/
│       └── USB_DRIVE/      Auto-mounted USB
│
├── opt/                    Optional/third-party software
│   ├── google/chrome/
│   └── vscode/
│
├── srv/                    Service data (web, FTP)
│   └── www/                Web server files
│
└── run/                    Runtime variable data (tmpfs)
    ├── user/1000/          User runtime data
    └── systemd/            Systemd runtime
```

---

## 4) Modern vs Traditional Layout

### The /usr Merge

Modern distros (Ubuntu 20.04+, Fedora 17+, Arch) have merged several directories:

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          /usr MERGE (Modern Linux)                                  │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   TRADITIONAL (Separate)              MODERN (Merged)                               │
│   ─────────────────────              ─────────────────                              │
│                                                                                     │
│   /bin/bash                    →     /usr/bin/bash                                  │
│   /sbin/init                   →     /usr/sbin/init                                 │
│   /lib/libc.so                 →     /usr/lib/libc.so                               │
│                                                                                     │
│   /bin  is now a symlink to /usr/bin                                                │
│   /sbin is now a symlink to /usr/sbin                                               │
│   /lib  is now a symlink to /usr/lib                                                │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   WHY THE MERGE?                                                                    │
│                                                                                     │
│   Historical reason for split:                                                      │
│   • Early Unix had small root disk + large /usr on separate partition               │
│   • /bin and /sbin had "essential" commands for single-user recovery                │
│                                                                                     │
│   Why merge now:                                                                    │
│   • Simpler packaging (one location for binaries)                                   │
│   • Atomic upgrades (update /usr as single unit)                                    │
│   • Stateless systems (full OS image in /usr, config in /etc)                       │
│   • Compatibility maintained via symlinks                                           │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Verify on your system:

```bash
# Check if /bin is a symlink
ls -la /bin

# Example output on modern Ubuntu:
# lrwxrwxrwx 1 root root 7 Apr 23 2024 /bin -> usr/bin

# On traditional systems:
# drwxr-xr-x 2 root root 4096 Dec 15 10:30 /bin
```

---

## 5) Virtual Filesystems (proc, sys, dev)

These are **not** stored on disk — the kernel generates their content dynamically.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         VIRTUAL FILESYSTEMS                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  PROCFS (/proc)                                                                     │
│  ─────────────                                                                      │
│  • Process information pseudo-filesystem                                            │
│  • One directory per running process (/proc/PID/)                                   │
│  • Kernel tuning via /proc/sys/                                                     │
│                                                                                     │
│  Examples:                                                                          │
│  /proc/cpuinfo      → CPU details (model, cores, flags)                             │
│  /proc/meminfo      → Memory statistics                                             │
│  /proc/1234/status  → Status of process 1234                                        │
│  /proc/sys/net/ipv4/ip_forward → Enable IP forwarding                               │
│                                                                                     │
│  ─────────────────────────────────────────────────────────────────────────────      │
│                                                                                     │
│  SYSFS (/sys)                                                                       │
│  ──────────                                                                         │
│  • Structured view of devices and drivers                                           │
│  • Interface to configure hardware                                                  │
│  • Exposes kernel objects as files                                                  │
│                                                                                     │
│  Examples:                                                                          │
│  /sys/class/net/eth0/       → Network interface settings                            │
│  /sys/class/gpio/gpio17/    → GPIO pin control (embedded)                           │
│  /sys/block/sda/size        → Disk size in sectors                                  │
│  /sys/devices/system/cpu/   → CPU topology                                          │
│                                                                                     │
│  ─────────────────────────────────────────────────────────────────────────────      │
│                                                                                     │
│  DEVTMPFS (/dev)                                                                    │
│  ──────────────                                                                     │
│  • Device nodes created by kernel + udev                                            │
│  • Block devices (disks): /dev/sda, /dev/nvme0n1                                    │
│  • Character devices: /dev/tty, /dev/random                                         │
│  • Pseudo-devices: /dev/null, /dev/zero                                             │
│                                                                                     │
│  Examples:                                                                          │
│  /dev/sda1           → First partition on first SATA disk                           │
│  /dev/nvme0n1p1      → First partition on first NVMe SSD                            │
│  /dev/ttyUSB0        → USB serial adapter                                           │
│  /dev/video0         → Webcam                                                       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Practical Examples:

```bash
# Read CPU info
cat /proc/cpuinfo | head -20

# Check memory
cat /proc/meminfo | head -10

# List network interfaces via sysfs
ls /sys/class/net/

# Enable IP forwarding (runtime)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Check disk info
cat /sys/block/sda/size    # Size in 512-byte sectors
```

---

## 6) RootFS Types and Storage

### Different Filesystem Types for RootFS

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          ROOTFS STORAGE OPTIONS                                     │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   DISK-BASED (Read-Write)                                                           │
│   ───────────────────────                                                           │
│   ext4        Most common on desktops/servers, journaling, stable                   │
│   btrfs       Copy-on-write, snapshots, compression, modern                         │
│   XFS         High performance, good for large files                                │
│   f2fs        Flash-Friendly FS, optimized for SSDs/eMMC                            │
│                                                                                     │
│   FLASH-BASED (Embedded)                                                            │
│   ──────────────────────                                                            │
│   squashfs    Read-only, heavily compressed, used for live CDs and embedded         │
│   UBIFS       For raw NAND flash with wear-leveling                                 │
│   JFFS2       Older alternative to UBIFS                                            │
│                                                                                     │
│   MEMORY-BASED (RAM)                                                                │
│   ──────────────────                                                                │
│   tmpfs       Temporary storage in RAM, fast, volatile                              │
│   ramfs       Simple RAM filesystem, no size limit                                  │
│   initramfs   Initial RAM filesystem for early boot                                 │
│                                                                                     │
│   NETWORK-BASED                                                                     │
│   ─────────────                                                                     │
│   NFS         Network File System, root over network (diskless clients)             │
│   iSCSI       Block-level network storage                                           │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Check Your RootFS:

```bash
# What filesystem is your root mounted as?
df -Th /

# Example output:
# Filesystem     Type  Size  Used Avail Use% Mounted on
# /dev/sda2      ext4  100G   25G   70G  26% /

# Detailed mount info
mount | grep " / "

# All mounted filesystems
findmnt
```

---

## 7) RootFS in the Boot Process

### The Journey to RootFS

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          BOOT → ROOTFS FLOW                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌────────────────┐                                                                │
│   │ 1. BOOTLOADER  │  GRUB reads kernel command line                                │
│   │    (GRUB)      │  root=/dev/sda2  ← tells kernel where rootfs is                │
│   └───────┬────────┘                                                                │
│           │                                                                         │
│           ▼                                                                         │
│   ┌────────────────┐                                                                │
│   │ 2. KERNEL      │  Loads kernel + initramfs into RAM                             │
│   │    LOADS       │  initramfs = temporary mini-rootfs                             │
│   └───────┬────────┘                                                                │
│           │                                                                         │
│           ▼                                                                         │
│   ┌────────────────┐                                                                │
│   │ 3. INITRAMFS   │  Mini system runs from RAM                                     │
│   │    RUNS        │  • Loads disk drivers                                          │
│   │                │  • Decrypts disk (if encrypted)                                │
│   │                │  • Assembles RAID/LVM                                          │
│   │                │  • Finds and validates real rootfs                             │
│   └───────┬────────┘                                                                │
│           │                                                                         │
│           ▼                                                                         │
│   ┌────────────────┐                                                                │
│   │ 4. switch_root │  Pivots from initramfs to real rootfs                          │
│   │                │  • Mounts /dev/sda2 at the new /                               │
│   │                │  • Deletes initramfs contents                                  │
│   │                │  • Changes root directory                                      │
│   └───────┬────────┘                                                                │
│           │                                                                         │
│           ▼                                                                         │
│   ┌────────────────┐                                                                │
│   │ 5. REAL ROOTFS │  /dev/sda2 is now mounted as /                                 │
│   │    ACTIVE      │  Kernel runs /sbin/init (systemd)                              │
│   │                │  System boots normally                                         │
│   └────────────────┘                                                                │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Why initramfs exists:

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        WHY USE INITRAMFS?                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   PROBLEM: Kernel needs drivers to access disk, but drivers are ON the disk!        │
│                                                                                     │
│   Chicken-and-egg:                                                                  │
│   • Kernel: "I need to mount /dev/sda2 for rootfs"                                  │
│   • Kernel: "But I need AHCI/NVMe driver to access /dev/sda2"                       │
│   • Kernel: "Driver is in /lib/modules on /dev/sda2"                                │
│   • Kernel: "😱"                                                                    │
│                                                                                     │
│   SOLUTION: initramfs                                                               │
│   • Bootloader loads initramfs + kernel together                                    │
│   • initramfs contains essential drivers in RAM                                     │
│   • Boot scripts in initramfs mount the real rootfs                                 │
│   • switch_root transitions to real rootfs                                          │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   CONTENTS OF initramfs:                                                            │
│                                                                                     │
│   /init                  Boot script (shell or binary)                              │
│   /bin/busybox           Swiss-army-knife utility                                   │
│   /lib/modules/          Essential drivers (disk, filesystem)                       │
│   /etc/                   Minimal config                                            │
│   /sbin/                  Essential tools (switch_root)                             │
│                                                                                     │
│   View initramfs contents:                                                          │
│   $ lsinitramfs /boot/initrd.img-$(uname -r) | head -30                             │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8) Desktop vs Embedded RootFS

### Desktop/Server RootFS (Ubuntu, Fedora)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          DESKTOP ROOTFS                                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Size: 5GB - 50GB+                                                                 │
│   Filesystem: ext4, btrfs                                                           │
│   Storage: SSD/HDD, NVMe                                                            │
│                                                                                     │
│   Typical contents:                                                                 │
│   • Full GNU coreutils (bash, ls, grep, awk, sed, etc.)                             │
│   • Multiple shells (bash, zsh, fish)                                               │
│   • Package manager (apt, dnf, pacman)                                              │
│   • Desktop environment (GNOME, KDE)                                                │
│   • Many applications (browsers, editors, office)                                   │
│   • Development tools (gcc, python, nodejs)                                         │
│   • Full documentation and man pages                                                │
│                                                                                     │
│   Characteristics:                                                                  │
│   ✓ Read-write                                                                      │
│   ✓ Full-featured                                                                   │
│   ✓ Package manager for updates                                                     │
│   ✓ User home directories                                                           │
│   ✓ Logs and variable data                                                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Embedded RootFS (Yocto, Buildroot)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          EMBEDDED ROOTFS                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Size: 8MB - 500MB                                                                 │
│   Filesystem: squashfs (read-only), ubifs, ext4                                     │
│   Storage: eMMC, NAND flash, SD card                                                │
│                                                                                     │
│   Typical contents:                                                                 │
│   • BusyBox (single binary providing 100+ utilities)                                │
│   • musl or uClibc (smaller than glibc)                                             │
│   • Minimal shell (ash/busybox sh)                                                  │
│   • Only application-specific programs                                              │
│   • No desktop environment                                                          │
│   • No package manager (image is pre-built)                                         │
│                                                                                     │
│   Characteristics:                                                                  │
│   ✓ Often read-only (reliability, security)                                         │
│   ✓ Compressed (squashfs saves space)                                               │
│   ✓ Minimalist                                                                      │
│   ✓ Custom-built for specific hardware                                              │
│   ✓ Fast boot (less to load)                                                        │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   COMPARISON:                                                                       │
│                                                                                     │
│   Feature         │ Desktop           │ Embedded                                    │
│   ────────────────┼───────────────────┼─────────────────                            │
│   /bin/ls         │ GNU coreutils     │ BusyBox symlink                             │
│   C Library       │ glibc (~2MB)      │ musl (~100KB)                               │
│   Init system     │ systemd           │ BusyBox init, or custom                     │
│   Updates         │ apt/dnf           │ Full image replacement                      │
│   /home           │ Many users        │ Rarely present                              │
│   /var/log        │ Extensive         │ Minimal or tmpfs                            │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 9) Creating a Minimal RootFS

### Method 1: Using debootstrap (Debian/Ubuntu)

```bash
# Create a minimal Debian rootfs
sudo apt install debootstrap

# Bootstrap a minimal Ubuntu into a directory
sudo debootstrap --arch=amd64 jammy /mnt/myrootfs http://archive.ubuntu.com/ubuntu

# Chroot into it
sudo chroot /mnt/myrootfs /bin/bash
```

### Method 2: BusyBox (Embedded style)

```bash
# On a build machine:

# 1. Create directory structure
mkdir -p myrootfs/{bin,sbin,etc,proc,sys,dev,lib,usr/bin,usr/sbin}

# 2. Build static busybox (or download)
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
make defconfig
make menuconfig  # Enable "Build static binary"
make
make install CONFIG_PREFIX=../myrootfs

# 3. Create essential files
cat > myrootfs/etc/passwd << EOF
root:x:0:0:root:/root:/bin/sh
EOF

cat > myrootfs/etc/inittab << EOF
::sysinit:/etc/init.d/rcS
::askfirst:/bin/sh
EOF

# 4. Create init script
mkdir -p myrootfs/etc/init.d
cat > myrootfs/etc/init.d/rcS << 'EOF'
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
echo "Welcome to minimal Linux!"
EOF
chmod +x myrootfs/etc/init.d/rcS

# Result: ~2MB rootfs!
```

### Method 3: Buildroot (Automated embedded)

```bash
git clone https://github.com/buildroot/buildroot.git
cd buildroot
make qemu_x86_64_defconfig
make menuconfig  # Configure packages
make             # Builds kernel + rootfs

# Output in output/images/rootfs.ext4
```

---

## 10) Inspecting Your RootFS

### Useful Commands

```bash
# Size of root filesystem
df -h /

# Disk usage by directory
sudo du -sh /* 2>/dev/null | sort -h

# Largest directories
sudo du -sh /usr/* 2>/dev/null | sort -h

# What's eating space?
sudo ncdu /  # Interactive (install with apt install ncdu)

# List mounted filesystems
findmnt --real

# Check root device
cat /proc/cmdline | grep -oP 'root=\S+'

# Filesystem type
stat -f /

# Inode usage
df -i /
```

### Example Output Analysis

```bash
$ df -Th /
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/nvme0n1p2 ext4  234G   45G  177G  21% /

$ sudo du -sh /* 2>/dev/null | sort -h | tail -10
16K     /media
24K     /root
260M    /boot
520M    /var
600M    /opt
1.5G    /home
6.2G    /snap
29G     /usr

# /usr is largest because it contains most installed programs
```

---

## 11) Common RootFS Problems and Solutions

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          TROUBLESHOOTING ROOTFS                                     │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Problem: "Kernel panic - VFS: Unable to mount root fs"                            │
│   Causes:                                                                           │
│   • Wrong root= parameter in bootloader                                             │
│   • Missing disk driver in kernel/initramfs                                         │
│   • Corrupted filesystem                                                            │
│   Fix: Boot from live USB, check fstab and grub config                              │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Problem: Root filesystem full                                                     │
│   Check: df -h /                                                                    │
│   Find culprit: sudo du -sh /* | sort -h                                            │
│   Common causes: /var/log, /tmp, /var/cache                                         │
│   Fix: sudo apt clean, truncate logs, remove old kernels                            │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Problem: Read-only root filesystem                                                │
│   Causes:                                                                           │
│   • Filesystem errors (mounted read-only for safety)                                │
│   • Intentionally read-only (embedded systems)                                      │
│   Fix: sudo fsck /dev/sdXN (from live USB), then remount rw                         │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Problem: /dev/sda changed to /dev/sdb after adding disk                           │
│   Cause: Device names not persistent                                                │
│   Fix: Use UUID or LABEL in /etc/fstab and GRUB:                                    │
│   UUID=abc123-... /  ext4  defaults  0  1                                           │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 12) RootFS Best Practices

### For Desktop/Servers:
- Use UUIDs in /etc/fstab (not device names like /dev/sda1)
- Separate /home on different partition (easy reinstalls)
- Regular backups (especially /etc and /home)
- Monitor disk usage (alerts at 80%)

### For Embedded:
- Keep rootfs minimal (only what's needed)
- Use read-only rootfs + overlay for wear leveling
- Verify rootfs integrity (dm-verity, IMA)
- Plan for atomic updates (A/B partitions)

---

## 13) Hands-On Exercises

1. **Explore your rootfs:**
   ```bash
   tree -L 2 / 2>/dev/null | head -50
   ls -la /bin  # Is it a symlink?
   ```

2. **Find essential files:**
   ```bash
   which init
   ls -la /sbin/init
   file /sbin/init
   ```

3. **Check initramfs contents:**
   ```bash
   lsinitramfs /boot/initrd.img-$(uname -r) | head -50
   ```

4. **View kernel's root parameter:**
   ```bash
   cat /proc/cmdline
   ```

5. **Compare filesystem sizes:**
   ```bash
   df -Th
   mount | grep -E "proc|sys|dev|tmp"
   ```

---

## 14) Key Takeaways

| Concept | Summary |
|---------|---------|
| **RootFS** | The filesystem mounted at `/`, containing all OS files |
| **FHS** | Standard that defines where files should be located |
| **Virtual FS** | /proc, /sys, /dev — kernel provides these dynamically |
| **initramfs** | Temporary RAM-based rootfs used during early boot |
| **switch_root** | The moment boot transitions from initramfs to real rootfs |
| **Modern layout** | /bin, /sbin, /lib are now symlinks to /usr/* |
| **Embedded** | Minimal rootfs (BusyBox + musl), often read-only |

---

*Next up: [02_Boot_Process.md](02_Boot_Process.md) — detailed walkthrough of the entire boot sequence from power-on to login prompt.*

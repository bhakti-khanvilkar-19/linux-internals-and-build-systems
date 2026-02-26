# Linux Fundamentals — What is Linux, Kernel, Bootloader, RootFS, and Distributions

## Objective
Explain, in simple terms and with technical accuracy, what "Linux" means, what a kernel is, and how components like the bootloader and root filesystem fit together. Also clarify what a distribution (distro) is, how Ubuntu relates, and practical commands to inspect your system.

---

## 1) The History — How Linux Came to Be

### The Backstory (Timeline)

| Year | Event |
|------|-------|
| 1969 | **Unix** is created at AT&T Bell Labs by Ken Thompson and Dennis Ritchie. It becomes the grandfather of modern operating systems. |
| 1983 | Richard Stallman starts the **GNU Project** — goal: create a completely free (as in freedom) Unix-like OS. Tools like `gcc`, `bash`, `glibc`, `coreutils` are born here. |
| 1987 | Andrew Tanenbaum creates **MINIX**, a small Unix-like OS for teaching. |
| **1991** | **Linus Torvalds**, a 21-year-old Finnish student, announces a free kernel he's writing "just as a hobby". This becomes the **Linux kernel**. |
| 1992 | Linux is relicensed under the **GNU GPL v2**, making it truly free and open-source. |
| 1993 | First distributions appear — **Slackware** and **Debian**. |
| 1994 | Linux kernel 1.0 released. |
| 2004 | **Ubuntu** 4.10 ("Warty Warthog") launches — aims to make Linux easy for everyone. |
| Today | Linux runs on everything: phones (Android), servers (90%+ of the cloud), supercomputers (100% of Top500), embedded devices, cars, TVs, routers. |

### The Key Insight
The GNU project had built all the user-space tools (compiler, shell, utilities) but was missing a **kernel**. Linus built the kernel. Together: **GNU tools + Linux kernel = a complete OS**. That's why some people call it "GNU/Linux" — but most just say "Linux".

---

## 2) Short answer (layman's terms)
- "Linux" commonly refers to an operating system built around the Linux **kernel** plus many user programs and utilities.
- The **kernel** is the core software that talks to hardware (CPU, memory, disk, network) and provides services to programs.
- A **distribution (distro)** packages the kernel together with user-space tools, installers, package manager, and default configuration (e.g., Ubuntu, Fedora, Arch).

When people say "I run Linux" they usually mean "I run a Linux distribution (Ubuntu, etc.)."

### Simple Analogy
Think of it like a **car**:
- **Kernel** = the **engine** — it does the real work (runs processes, manages memory, drives hardware). You never touch it directly.
- **Rootfs + Userspace tools** = the **body, seats, dashboard, steering wheel** — the parts you interact with.
- **Bootloader** = the **ignition key** — it starts the engine.
- **Distribution (distro)** = the **car brand/model** (Toyota Camry, Honda Civic) — same type of engine (Linux kernel) but different body style, features, and target audience.

So: Ubuntu, Fedora, Arch = different car brands. They all use a Linux engine underneath.

---

## 3) Main components and roles (simplified)

### Visual: How the pieces stack up

```
┌─────────────────────────────────────────────────┐
│              USER APPLICATIONS                  │
│   (Firefox, VS Code, your C program, bash)      │
├─────────────────────────────────────────────────┤
│            SYSTEM LIBRARIES                     │
│      (glibc, libpthread, libm, etc.)            │
├─────────────────────────────────────────────────┤
│       SYSTEM CALLS INTERFACE (API boundary)     │
│  open() read() write() fork() ioctl() mmap()    │
╠═════════════════════════════════════════════════╣  ← User / Kernel boundary
│                 LINUX KERNEL                    │
│  ┌───────────┬──────────┬──────────────────┐    │
│  │ Process   │ Memory   │ Virtual File     │    │
│  │ Scheduler │ Manager  │ System (VFS)     │    │
│  ├───────────┼──────────┼──────────────────┤    │
│  │ Network   │ Device Drivers (modules)    │    │
│  │ Stack     │ (disk, USB, GPU, NIC, ...)  │    │
│  └───────────┴──────────┴──────────────────┘    │
├─────────────────────────────────────────────────┤
│              HARDWARE                           │
│   (CPU, RAM, SSD/HDD, NIC, GPU, USB, ...)       │
└─────────────────────────────────────────────────┘
```

### Component roles:
- **Bootloader**: small program that runs first and loads the kernel (example: GRUB on many PCs).
- **Kernel**: manages hardware, processes, memory, drivers, filesystems.
- **init / init system**: first user-space process started by kernel, typically `systemd` on Ubuntu; it starts services and user sessions.
- **Root filesystem (rootfs)**: the file tree that contains user-space programs and configuration (directories like `/bin`, `/etc`, `/usr`, `/home`).
- **Userspace**: shells, GUI, daemons, libraries — everything outside the kernel.

---

## 4) The boot sequence (typical modern PC, step-by-step)
1. Firmware (BIOS or UEFI) initializes hardware and finds a boot device.
2. Bootloader (GRUB) loads — it may present a menu and then loads the kernel and an initramfs (initial ram filesystem).
3. Kernel starts, initializes hardware and drivers, mounts the initramfs (a tiny temporary root filesystem in RAM).
4. Kernel executes `init` from the initramfs (or directly `/sbin/init`), which then switches root to the real root filesystem on disk (e.g., `/dev/sda2`) and starts the real init system (usually systemd).
5. `systemd` starts services, brings up networking, mounts filesystems, and spawns login managers and getty shells.

Why initramfs? It provides early userspace (drivers, scripts) needed to mount the real root filesystem (useful for encrypted disks, LVM, busybox utilities).

### Visual: Boot Sequence Flow

```
 Power On
    │
    ▼
┌────────────┐
│  FIRMWARE   │  BIOS / UEFI
│  (ROM/Flash)│  Initialize hardware, find boot device
└─────┬──────┘
      ▼
┌────────────┐
│ BOOTLOADER  │  GRUB / systemd-boot
│ (on disk)   │  Show menu, load kernel + initramfs into RAM
└─────┬──────┘
      ▼
┌────────────┐
│   KERNEL    │  Initialize drivers, mount initramfs
│ (vmlinuz)   │  Decompress itself, set up memory, interrupts
└─────┬──────┘
      ▼
┌────────────┐
│  INITRAMFS  │  Temporary root in RAM
│ (initrd.img)│  Load disk drivers, find real root partition
└─────┬──────┘
      ▼
┌────────────┐
│ REAL ROOTFS │  switch_root to /dev/sda2 (or similar)
│ (on disk)   │  Kernel hands off to /sbin/init
└─────┬──────┘
      ▼
┌────────────┐
│  SYSTEMD    │  PID 1 — starts services, networking,
│ (init)      │  mounts filesystems, spawns login
└─────┬──────┘
      ▼
┌────────────┐
│  LOGIN /    │  getty (terminal) or Display Manager (GUI)
│  DESKTOP    │  You're in!
└────────────┘
```

> **Want more detail?** See [02_Boot_Process.md](02_Boot_Process.md) for a comprehensive breakdown of every boot stage, including memory maps, flow diagrams, and a comparison between x86 PC and embedded systems.

---

## 5) What is the kernel exactly?
- The kernel is a single program running in a privileged CPU mode that provides:
  - Process scheduling (who runs and when)
  - Memory management (virtual memory, page tables)
  - Device drivers (network, disk, GPU)
  - Filesystem support (ext4, btrfs, xfs)
  - System call interface (how user programs request services)
- Linux kernel types: monolithic (most drivers in kernel), loadable modules (can insert drivers at runtime).

Useful commands to inspect your kernel:

```bash
uname -sr           # Kernel name and release (e.g., Linux 6.8.0-...)
uname -a            # Full kernel and system info
cat /proc/version   # Kernel build info
ls /lib/modules/$(uname -r)   # Installed kernel modules for current kernel
```

---

## 6) What is a distribution (distro)? How does it differ from the kernel?
- Kernel = kernel only. It cannot run a desktop or shell by itself very comfortably — you still need many user programs.
- Distro = kernel + userland packages + installer + defaults.
  - Userland includes `glibc` (C library), shell (bash), coreutils (ls, cp, mv), package manager, init system, GUI, and many applications.
- Examples:
  - Ubuntu: Debian-based, uses APT, systemd, regular LTS releases and Ubuntu-specific patches.
  - Fedora: upstream-focused, uses rpm/dnf, newer packages, SELinux by default.
  - Arch: rolling release, pacman, minimal base system and user config.

Differences between distros are mainly: package format & manager, release model (LTS vs rolling), default configuration and included packages, system administration tools, target audiences.

### Distro Family Tree (simplified)

```
                    Linux Kernel (kernel.org)
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Debian           Red Hat           Independent
        │                │                │
   ┌────┴────┐     ┌────┴────┐      ┌───┴────┐
   ▼         ▼     ▼         ▼      ▼        ▼
 Ubuntu    Mint  Fedora    CentOS  Arch    Gentoo
   │              │        /RHEL   Alpine   Slackware
   ▼              ▼
 Pop!_OS     Rocky Linux
 Kubuntu
```

### Quick Comparison Table

| Distro | Based On | Package Manager | Release Model | Best For |
|--------|----------|----------------|---------------|----------|
| Ubuntu | Debian | `apt` / `.deb` | LTS (2yr) + interim | Beginners, servers, desktops |
| Fedora | Independent (Red Hat sponsored) | `dnf` / `.rpm` | ~6 months | Developers, newest features |
| Arch | Independent | `pacman` | Rolling | Power users, learning Linux |
| Debian | Independent | `apt` / `.deb` | Stable (~2yr) | Servers, stability |
| CentOS/Rocky | RHEL | `dnf` / `.rpm` | LTS | Enterprise servers |
| Alpine | Independent | `apk` | Rolling | Containers (Docker), minimal |

---

## 7) What is the root filesystem (rootfs)?
- `rootfs` or root filesystem is the top of directory tree `/` which contains directories like `/bin`, `/etc`, `/usr`, `/var`, `/home`, `/boot`.
- The rootfs lives on a disk partition (or is in RAM for initramfs). The kernel mounts it as `/` so all programs can find their files.
- Minimal contents required for booting: kernel (in `/boot/`), init system (e.g., `/sbin/init`), essential libraries and device nodes.

List typical top-level directories and short purpose:
- `/bin` - essential user binaries (ls, cat)
- `/sbin` - system binaries (ip, mount)
- `/etc` - configuration files
- `/usr` - user utilities and apps
- `/var` - variable data (logs, spool)
- `/home` - user home directories
- `/boot` - kernel and bootloader files
- `/dev` - device nodes (created dynamically)
- `/proc` and `/sys` - kernel-provided pseudo-filesystems for runtime info

Example: show `/boot` and kernels installed:

```bash
ls -l /boot
# Example output contains vmlinuz-*, initrd.img-*, grub/ directory
```

---

## 8) Bootloader examples and role
- GRUB (GRand Unified Bootloader): very common on PCs, shows menu, can load multiple kernels, passes parameters to kernel.
- systemd-boot, LILO, rEFInd are alternatives.

Inspect GRUB config (Ubuntu):

```bash
# Show GRUB menu entries (read-only)
sudo grep -E "menuentry|linux" /boot/grub/grub.cfg | sed -n '1,40p'
```

Note: `/boot/grub/grub.cfg` is usually autogenerated by `update-grub`.

---

## 9) Userspace vs kernel: who does what?
- Kernel: hardware abstraction and core services.
- Userspace: programs you interact with (terminal, editor, GUI).
- System calls are the bridge: programs call OS functions (open, read, write, fork).

Example: `ls` is a user program that calls `open()` and `getdents()` system calls implemented by the kernel.

---

## 10) Practical examples and commands (Ubuntu 24.04)
- Check distro and version:

```bash
lsb_release -a
cat /etc/os-release
```

- Kernel version:

```bash
uname -sr
cat /proc/version
```

- List installed kernels:

```bash
ls -1 /boot | grep vmlinuz
```

- Show boot loader entries (GRUB):

```bash
sudo awk '/menuentry /{print $0}' /boot/grub/grub.cfg
```

- View root filesystem usage:

```bash
df -h /
```

- Inspect pseudo-filesystems:

```bash
ls /proc | head -n 20
ls /sys | head -n 20
```

---

## 11) Frequently asked confusions (FAQ)
Q: "Is Linux the same as Ubuntu?"
A: No — Ubuntu is a distribution that uses the Linux kernel plus many other packages and configuration choices. "Linux" properly means the kernel.

Q: "Can I run programs without a kernel?"
A: No — the kernel provides CPU scheduling, memory, I/O. Without a kernel there's no managed hardware access for programs.

Q: "What is the difference between initramfs and rootfs?"
A: initramfs is a temporary in-RAM filesystem used very early during boot to help mount the real rootfs on disk. After switching, the real rootfs becomes `/`.

Q: "Do all distros use the same kernel?"
A: Most distros use the Linux kernel but may apply different patches or package different kernel builds/version. Some distributions maintain specific kernels (Ubuntu's LTS kernel). The kernel source is common upstream at kernel.org.

---

## 12) Suggested exercises (hands-on)
1. Run these commands and read outputs:

```bash
lsb_release -a
uname -a
ls -l /boot
df -h /
cat /proc/cmdline   # kernel boot parameters
```

2. Reboot and watch GRUB menu; note different kernel entries.
3. Inspect `/proc` and `/sys` to see how kernel exposes hardware info:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
ls /sys/class/net
```

---

## 13) Key Terminology — Quick Reference

| Term | What It Means |
|------|---------------|
| **Kernel** | The core program that manages hardware and provides services to all other programs. Runs in privileged (ring 0) CPU mode. |
| **Userspace / Userland** | Everything that is NOT the kernel — your shell, apps, libraries. Runs in unprivileged (ring 3) CPU mode. |
| **Bootloader** | First software after firmware; loads the kernel into RAM (e.g., GRUB). |
| **Rootfs** | The root (`/`) filesystem on disk containing all OS files and directories. |
| **initramfs / initrd** | A temporary root filesystem loaded into RAM during early boot, used to prepare and mount the real rootfs. |
| **Distro** | A packaged combination of kernel + userspace + package manager + configuration = a usable OS (e.g., Ubuntu). |
| **systemd** | The most common init system on modern Linux; PID 1, starts and manages all services. |
| **Shell** | A command-line interpreter (e.g., `bash`, `zsh`) that lets you type commands. |
| **System call (syscall)** | The API through which user programs request kernel services (e.g., `read()`, `write()`, `fork()`). |
| **Module** | A piece of kernel code that can be loaded/unloaded at runtime (e.g., a device driver). |
| **Package manager** | Tool to install/remove/update software (e.g., `apt` on Ubuntu, `dnf` on Fedora). |
| **GNU** | "GNU's Not Unix" — the project that created most of the userspace tools (gcc, bash, coreutils) used alongside the Linux kernel. |

---

## 14) Where to read next
- Kernel basics and architecture: "Linux Kernel Development" (book) and `kernel.org` docs
- Boot process details: "What happens when Linux boots" articles and systemd docs
- Ubuntu specific: `help.ubuntu.com` and `/usr/share/doc` on your system
- GNU Project: `gnu.org` — understanding the tools that complement the kernel

---

*Next up: `02_File_System.md` — deep dive into the Linux directory structure, permissions, and filesystem types.*
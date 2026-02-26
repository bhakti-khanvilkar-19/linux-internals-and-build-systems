# Ubuntu Core & Snaps

## 📚 What is Ubuntu Core?

**Ubuntu Core** is Canonical's minimal, containerized version of Ubuntu designed for:
- IoT devices
- Edge computing
- Embedded systems
- Appliances

> **Key Difference:** Unlike traditional Ubuntu, everything in Ubuntu Core is a snap - including the kernel and bootloader.

---

## 🏗️ Ubuntu Core Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        UBUNTU CORE STACK                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                      APPLICATION SNAPS                         │     │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │     │
│  │  │ App 1   │  │ App 2   │  │ App 3   │  │ App N   │            │     │
│  │  │ (snap)  │  │ (snap)  │  │ (snap)  │  │ (snap)  │            │     │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘            │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│                              │ Interfaces (plugs/slots)                 │
│                              ▼                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                    SNAPD (Snap Daemon)                         │     │
│  │  • Manages snap lifecycle                                      │     │
│  │  • Handles updates (transactional)                             │     │
│  │  • Enforces security (AppArmor, seccomp)                       │     │
│  │  • Connects interfaces                                         │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│  ┌───────────────────────────┼───────────────────────────────────┐      │
│  │           SYSTEM SNAPS    │                                   │      │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │      │
│  │  │    KERNEL    │  │    GADGET    │  │   CORE/BASE  │         │      │
│  │  │    snap      │  │    snap      │  │    snap      │         │      │
│  │  └──────────────┘  └──────────────┘  └──────────────┘         │      │
│  └───────────────────────────────────────────────────────────────┘      │
│                              │                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                     BOOTLOADER (GRUB/U-Boot)                   │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                          HARDWARE                              │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📦 What is a Snap?

A **snap** is a containerized software package that:
- Contains application + dependencies
- Is sandboxed using AppArmor and seccomp
- Updates automatically and atomically
- Can rollback on failure

### Snap File Structure

```
myapp.snap (SquashFS archive)
├── meta/
│   ├── snap.yaml           # Snap metadata
│   └── hooks/
│       ├── configure       # Configuration hook
│       ├── install         # Post-install hook
│       └── remove          # Pre-remove hook
├── bin/
│   └── myapp               # Application binary
├── lib/
│   └── *.so               # Bundled libraries
└── ...
```

### Snap vs Traditional Package

| Aspect | Snap | DEB/APT |
|--------|------|---------|
| **Dependencies** | Bundled | Shared system libs |
| **Isolation** | Sandboxed | Direct system access |
| **Updates** | Automatic, atomic | Manual, can break |
| **Rollback** | Built-in | Difficult |
| **Size** | Larger | Smaller |
| **Security** | AppArmor confined | Full access |
| **Cross-distro** | Yes | No |

---

## 🔧 System Snaps (The Core Components)

Ubuntu Core requires four essential system snaps:

### 1. **Core / Base Snap**

The base snap provides the minimal root filesystem.

| Base Snap | Ubuntu Version | Support |
|-----------|---------------|---------|
| `core24` | Ubuntu 24.04 | Current |
| `core22` | Ubuntu 22.04 | LTS |
| `core20` | Ubuntu 20.04 | LTS |
| `core18` | Ubuntu 18.04 | Extended |
| `core` | Ubuntu 16.04 | Legacy |

```yaml
# In snapcraft.yaml
base: core24
```

**What's inside core24:**
- glibc, systemd, dbus
- Basic utilities (coreutils, bash)
- SSL certificates
- Ubuntu base configuration

### 2. **Snapd Snap**

The snap daemon that manages everything.

```
snapd snap provides:
├── /usr/lib/snapd/snapd       # Main daemon
├── /usr/bin/snap              # CLI tool
├── /usr/lib/snapd/snap-*      # Helpers
└── D-Bus services
```

**Responsibilities:**
- Install/remove/update snaps
- Manage interfaces (permissions)
- Handle assertions (signatures)
- Communicate with Snap Store

### 3. **Kernel Snap**

Contains the Linux kernel and modules.

```
kernel snap structure:
├── kernel.img          # Compressed kernel (vmlinuz)
├── initrd.img          # Initial ramdisk
├── dtbs/              # Device tree blobs (ARM)
│   └── *.dtb
├── modules/           # Kernel modules
│   └── <version>/
└── firmware/          # Binary firmware files
```

**Key Points:**
- Built using Ubuntu kernel source
- Includes board-specific DTBs
- Can be customized per device
- Updates atomically with rollback

### 4. **Gadget Snap**

Board-specific bootloader and hardware configuration.

```
gadget snap structure:
├── meta/
│   ├── snap.yaml
│   └── gadget.yaml     # Crucial: partition/boot config
├── grub.conf           # or uboot.env/uboot.conf
├── splash.bmp          # Boot splash (optional)
└── assets/             # Board-specific files
```

#### gadget.yaml Example

```yaml
# gadget.yaml - Defines disk layout and boot
device-tree: myboard.dtb

defaults:
  system:
    timezone: UTC
    
volumes:
  mydevice:
    bootloader: u-boot
    schema: gpt
    structure:
      - name: ubuntu-seed
        role: system-seed
        filesystem: vfat
        type: EF,C12A7328-F81F-11D2-BA4B-00A0C93EC93B
        size: 1200M
        content:
          - source: grubx64.efi
            target: EFI/boot/grubx64.efi
            
      - name: ubuntu-boot
        role: system-boot
        filesystem: ext4
        type: 83,0FC63DAF-8483-4772-8E79-3D69D8477DE4
        size: 750M
        
      - name: ubuntu-save
        role: system-save
        filesystem: ext4
        type: 83,0FC63DAF-8483-4772-8E79-3D69D8477DE4
        size: 16M
        
      - name: ubuntu-data
        role: system-data
        filesystem: ext4
        type: 83,0FC63DAF-8483-4772-8E79-3D69D8477DE4
        size: 4G
```

---

## 🔒 Snap Confinement

### Confinement Levels

| Level | Description | Use Case |
|-------|-------------|----------|
| `strict` | Full sandbox (AppArmor + seccomp) | Production apps |
| `classic` | No sandbox, full system access | Developer tools |
| `devmode` | Sandbox violations logged, not blocked | Development |

### Interfaces (Permissions)

Interfaces control what a snap can access.

```
┌─────────────────┐         ┌─────────────────┐
│   App Snap      │         │   System/Snap   │
│  ┌───────────┐  │         │  ┌───────────┐  │
│  │   Plug    │──┼────────▶│  │   Slot    │  │
│  │ (request) │  │ connect │  │ (provide) │  │
│  └───────────┘  │         │  └───────────┘  │
└─────────────────┘         └─────────────────┘
```

**Common Interfaces:**

| Interface | Provides Access To |
|-----------|-------------------|
| `network` | Network connectivity |
| `network-bind` | Bind to ports |
| `home` | User home directory |
| `removable-media` | USB drives |
| `camera` | Camera devices |
| `audio-playback` | Sound output |
| `bluetooth-control` | Bluetooth |
| `serial-port` | Serial devices |
| `gpio` | GPIO pins |
| `i2c` | I2C bus |
| `spi` | SPI bus |

```bash
# List connections
snap connections myapp

# Connect interface
sudo snap connect myapp:serial-port core:serial-port

# Disconnect
sudo snap disconnect myapp:serial-port
```

---

## 📱 Ubuntu Core Channels

Snaps are published in channels following a track/risk format:

```
<track>/<risk>/<branch>

Examples:
- latest/stable
- 24/stable      
- 24/candidate
- 24/beta
- 24/edge
```

| Risk Level | Description |
|------------|-------------|
| `stable` | Production-ready |
| `candidate` | Release candidate testing |
| `beta` | Feature-complete testing |
| `edge` | Daily/CI builds |

```bash
# Install from specific channel
sudo snap install myapp --channel=24/stable

# Switch channel
sudo snap switch --channel=24/candidate myapp

# Refresh from new channel
sudo snap refresh myapp
```

---

## 🔄 Ubuntu Core Update Mechanism

### Transactional Updates

```
┌───────────────────────────────────────────────────────────────┐
│                   UPDATE PROCESS                               │
├───────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Download new snap to staging                               │
│     ┌──────────────┐                                           │
│     │ Download     │──▶ /var/lib/snapd/snaps/myapp_v2.snap    │
│     └──────────────┘                                           │
│                                                                │
│  2. Mount new version (parallel)                               │
│     ┌──────────────┐                                           │
│     │ Mount new    │──▶ /snap/myapp/v2 (alongside v1)         │
│     └──────────────┘                                           │
│                                                                │
│  3. Switch to new version atomically                           │
│     ┌──────────────┐                                           │
│     │ Update       │──▶ /snap/myapp/current → v2              │
│     │ symlink      │                                           │
│     └──────────────┘                                           │
│                                                                │
│  4. Restart services                                           │
│     ┌──────────────┐                                           │
│     │ Restart      │──▶ systemctl restart snap.myapp.*        │
│     └──────────────┘                                           │
│                                                                │
│  5. Keep old version for rollback                              │
│     ┌──────────────┐                                           │
│     │ Old version  │──▶ /snap/myapp/v1 (retained)             │
│     └──────────────┘                                           │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

### Rollback

```bash
# Revert to previous version
sudo snap revert myapp

# List revisions
snap list --all myapp
```

---

## 💻 Ubuntu Core CLI Commands

```bash
#============================================
# SNAP MANAGEMENT
#============================================
snap list                          # List installed snaps
snap info myapp                    # Snap details
snap find "search term"            # Search store

snap install myapp                 # Install
snap install myapp --channel=edge  # Install from channel
snap remove myapp                  # Remove
snap refresh myapp                 # Update single snap
snap refresh                       # Update all snaps

#============================================
# SYSTEM SNAPS
#============================================
snap list --all                    # Show all revisions
snap version                       # Show snapd version
snap changes                       # Show recent changes
snap tasks <change-id>             # Show change details

#============================================
# INTERFACES
#============================================
snap interfaces                    # List all interfaces
snap connections                   # Show all connections
snap connections myapp             # App connections
sudo snap connect myapp:plug provider:slot
sudo snap disconnect myapp:plug

#============================================
# CONFIGURATION
#============================================
snap get myapp                     # Get all config
snap get myapp key                 # Get specific key
sudo snap set myapp key=value      # Set config

#============================================
# SERVICES
#============================================
snap services                      # List services
snap services myapp                # App services
sudo snap start myapp.service
sudo snap stop myapp.service
sudo snap restart myapp.service
snap logs myapp                    # View logs
snap logs -f myapp                 # Follow logs

#============================================
# DEVELOPMENT
#============================================
snap download myapp                # Download without install
snap try ./myapp                   # Install from directory (dev)
snap run --shell myapp             # Shell in snap environment
```

---

## 🖥️ Ubuntu Core Images

### Creating an Ubuntu Core Image

```bash
# Install ubuntu-image tool
sudo snap install ubuntu-image --classic

# Create model assertion
# (Sign with your Canonical account key)

# Build image
ubuntu-image snap model.assertion -o mydevice.img

# Write to SD card
sudo dd if=mydevice.img of=/dev/sdX bs=32M status=progress
```

### Model Assertion Example

```json
{
    "type": "model",
    "series": "16",
    "brand-id": "my-brand-id",
    "model": "my-device",
    "architecture": "amd64",
    "base": "core24",
    "grade": "dangerous",
    "snaps": [
        {
            "name": "pc-kernel",
            "id": "pYVQrBcKmBa0mZ4CCN7ExT6jH8rY1hza",
            "type": "kernel",
            "default-channel": "24/stable"
        },
        {
            "name": "pc",
            "id": "UqFziVZDHLSyO3TqSWgNBoAdHbLI4dAH",
            "type": "gadget",
            "default-channel": "24/stable"
        },
        {
            "name": "snapd",
            "id": "PMrrV4ml8uWuEUDBT8dSGnKUYbevVhc4",
            "type": "snapd"
        },
        {
            "name": "myapp",
            "id": "abc123...",
            "type": "app",
            "default-channel": "latest/stable"
        }
    ],
    "timestamp": "2026-02-25T00:00:00Z"
}
```

---

## 🎯 Interview Questions

### Q1: What are the four essential system snaps in Ubuntu Core?
**A:** 
1. **Core/Base** (core24) - Minimal root filesystem
2. **Snapd** - Snap daemon managing lifecycle
3. **Kernel** - Linux kernel and modules
4. **Gadget** - Bootloader and partition layout

### Q2: What is the difference between core, core20, core22, and core24?
**A:** They are base snaps containing different Ubuntu versions:
- core = Ubuntu 16.04
- core20 = Ubuntu 20.04 LTS
- core22 = Ubuntu 22.04 LTS
- core24 = Ubuntu 24.04 LTS

Apps must specify their base, and the base provides the runtime libraries.

### Q3: Explain snap confinement levels.
**A:**
- **Strict**: Full sandbox, only access explicitly granted via interfaces
- **Classic**: No confinement, same as traditional packages (rare)
- **Devmode**: Sandbox violations logged but not blocked (development only)

### Q4: How do snap updates differ from APT updates?
**A:**
- Snaps update transactionally (atomic, all-or-nothing)
- Keep previous version for instant rollback
- Automatic delta updates (only changed parts)
- No dependency conflicts (self-contained)
- APT updates can leave system in inconsistent state

### Q5: What is the gadget snap responsible for?
**A:** The gadget snap defines:
- Disk partition layout (gadget.yaml)
- Bootloader configuration (GRUB/U-Boot)
- Boot assets (DTBs, splash screens)
- Default system configuration
- Hardware-specific setup

---

## 🔗 Resources

- [Ubuntu Core Documentation](https://ubuntu.com/core/docs)
- [Snapcraft Documentation](https://snapcraft.io/docs)
- [Snap Store](https://snapcraft.io/store)
- [Ubuntu Core Tutorial](https://ubuntu.com/core/docs/getting-started)

---

*Previous: [← Yocto Build Process](./02_Yocto_Build_Process.md) | Next: [Building Snaps →](./04_Building_Snaps.md)*

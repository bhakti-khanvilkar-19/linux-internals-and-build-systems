# Yocto Build Process - Deep Dive

## 📚 Overview

This document covers the complete Yocto build process including:
- Recipe anatomy and syntax
- Layer structure and priorities
- BitBake task workflow
- Configuration files
- Output artifacts

---

## 🍳 Recipe Anatomy

### Basic Recipe Structure

```bash
# myapp_1.0.bb - Recipe file naming: <name>_<version>.bb

#============================================
# METADATA
#============================================
SUMMARY = "My Application"
DESCRIPTION = "A detailed description of my application"
HOMEPAGE = "https://example.com/myapp"
AUTHOR = "Developer Name <dev@example.com>"

LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE;md5=abc123def456..."

#============================================
# SOURCE CONFIGURATION
#============================================
SRC_URI = "git://github.com/example/myapp.git;branch=main;protocol=https"
SRCREV = "abc123def456789..."   # Git commit hash

# OR for tarball:
# SRC_URI = "https://example.com/myapp-${PV}.tar.gz"
# SRC_URI[sha256sum] = "abc123..."

# Additional files
SRC_URI += "file://myconfig.cfg \
            file://fix-build.patch"

S = "${WORKDIR}/git"  # Source directory

#============================================
# DEPENDENCIES
#============================================
DEPENDS = "zlib openssl"         # Build-time dependencies
RDEPENDS:${PN} = "libcurl bash"  # Runtime dependencies
RRECOMMENDS:${PN} = "optional-pkg"

#============================================
# BUILD CONFIGURATION
#============================================
inherit cmake    # Use CMake class (autotools, meson, etc.)

EXTRA_OECMAKE = "-DENABLE_FEATURE=ON"

#============================================
# TASKS (override if needed)
#============================================
do_configure:prepend() {
    # Run before default configure
}

do_compile() {
    oe_runmake
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 ${B}/myapp ${D}${bindir}/
    
    install -d ${D}${sysconfdir}
    install -m 0644 ${WORKDIR}/myconfig.cfg ${D}${sysconfdir}/
}

#============================================
# PACKAGING
#============================================
FILES:${PN} = "${bindir}/myapp ${sysconfdir}/myconfig.cfg"
FILES:${PN}-dev = "${includedir}"
FILES:${PN}-doc = "${docdir}"
```

---

## 📝 Recipe Variables Reference

### Common Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `PN` | Package name | `myapp` |
| `PV` | Package version | `1.0` |
| `PR` | Package release | `r0` |
| `PF` | Full package name | `myapp-1.0-r0` |
| `S` | Source directory | `${WORKDIR}/git` |
| `B` | Build directory | `${WORKDIR}/build` |
| `D` | Destination (staging) | `${WORKDIR}/image` |
| `WORKDIR` | Recipe work directory | `/build/tmp/work/.../myapp/1.0-r0` |

### Path Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `${bindir}` | `/usr/bin` | User binaries |
| `${sbindir}` | `/usr/sbin` | System binaries |
| `${libdir}` | `/usr/lib` | Libraries |
| `${includedir}` | `/usr/include` | Headers |
| `${sysconfdir}` | `/etc` | Config files |
| `${datadir}` | `/usr/share` | Shared data |
| `${docdir}` | `/usr/share/doc` | Documentation |
| `${systemd_system_unitdir}` | `/lib/systemd/system` | systemd units |

---

## 🎨 Recipe Classes

Classes provide reusable build logic.

```bash
inherit <classname>
```

### Common Classes

| Class | Use Case |
|-------|----------|
| `autotools` | GNU Autoconf/Automake projects |
| `cmake` | CMake projects |
| `meson` | Meson build system |
| `setuptools3` | Python packages |
| `go` | Go applications |
| `rust` | Rust applications |
| `systemd` | Systemd service integration |
| `kernel` | Linux kernel |
| `image` | Root filesystem images |
| `populate_sdk` | SDK generation |

### Example: systemd Service

```bash
inherit systemd

SYSTEMD_SERVICE:${PN} = "myapp.service"
SYSTEMD_AUTO_ENABLE = "enable"

SRC_URI += "file://myapp.service"

do_install:append() {
    install -d ${D}${systemd_system_unitdir}
    install -m 0644 ${WORKDIR}/myapp.service ${D}${systemd_system_unitdir}/
}
```

---

## 📂 Layer Deep Dive

### Layer Structure

```
meta-mylayer/
├── conf/
│   ├── layer.conf              # Layer metadata
│   └── machine/
│       └── myboard.conf        # Machine definition
├── recipes-core/
│   └── images/
│       └── my-image.bb         # Custom image
├── recipes-apps/
│   └── myapp/
│       ├── myapp_1.0.bb        # Main recipe
│       ├── myapp_%.bbappend    # Append to any version
│       └── files/
│           ├── myapp.service
│           └── fix-bug.patch
├── recipes-kernel/
│   └── linux/
│       └── linux-yocto_%.bbappend
├── classes/
│   └── myclass.bbclass
└── README.md
```

### layer.conf

```bash
# conf/layer.conf
BBPATH .= ":${LAYERDIR}"

BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
            ${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "mylayer"
BBFILE_PATTERN_mylayer = "^${LAYERDIR}/"

LAYERDEPENDS_mylayer = "core openembedded-layer"
LAYERSERIES_COMPAT_mylayer = "kirkstone scarthgap"

# Priority (higher = takes precedence)
BBFILE_PRIORITY_mylayer = "6"
```

### Machine Definition

```bash
# conf/machine/myboard.conf
#@TYPE: Machine
#@NAME: My Custom Board
#@DESCRIPTION: Machine configuration for My Board

DEFAULTTUNE = "cortexa53"
require conf/machine/include/arm/armv8a/tune-cortexa53.inc

# Kernel
PREFERRED_PROVIDER_virtual/kernel = "linux-yocto"
KERNEL_IMAGETYPE = "Image"
KERNEL_DEVICETREE = "myvendor/myboard.dtb"

# Bootloader
UBOOT_MACHINE = "myboard_defconfig"
IMAGE_BOOT_FILES = "Image myboard.dtb"

# Serial console
SERIAL_CONSOLES = "115200;ttyS0"

# Image settings
IMAGE_FSTYPES = "wic.gz ext4 tar.gz"
WKS_FILE = "myboard.wks"

# Features
MACHINE_FEATURES = "usbhost wifi bluetooth"
```

---

## 🔄 BitBake Task Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         BITBAKE TASK SEQUENCE                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────┐                                                          │
│  │ do_fetch   │──→ Download source (git clone, wget, etc.)               │
│  └─────┬──────┘                                                          │
│        │                                                                 │
│        ▼                                                                 │
│  ┌────────────┐                                                          │
│  │ do_unpack  │──→ Extract source to WORKDIR                             │
│  └─────┬──────┘                                                          │
│        │                                                                 │
│        ▼                                                                 │
│  ┌────────────┐                                                          │
│  │ do_patch   │──→ Apply patches from SRC_URI                            │
│  └─────┬──────┘                                                          │
│        │                                                                 │
│        ▼                                                                 │
│  ┌─────────────────┐                                                     │
│  │ do_configure    │──→ Run ./configure, cmake, etc.                     │
│  └──────┬──────────┘                                                     │
│         │                                                                │
│         ▼                                                                │
│  ┌─────────────┐                                                         │
│  │ do_compile  │──→ Run make, ninja, etc.                                │
│  └──────┬──────┘                                                         │
│         │                                                                │
│         ▼                                                                │
│  ┌─────────────┐                                                         │
│  │ do_install  │──→ Install to staging (D=${WORKDIR}/image)              │
│  └──────┬──────┘                                                         │
│         │                                                                │
│         ▼                                                                │
│  ┌──────────────────┐                                                    │
│  │ do_package       │──→ Split into packages (${PN}, ${PN}-dev, etc.)    │
│  └───────┬──────────┘                                                    │
│          │                                                               │
│          ▼                                                               │
│  ┌───────────────────────┐                                               │
│  │ do_package_write_*    │──→ Create IPK/DEB/RPM packages                │
│  └───────────────────────┘                                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Task Modifiers

```bash
# Run before task
do_compile:prepend() {
    echo "Before compile"
}

# Run after task
do_compile:append() {
    echo "After compile"
}

# Completely replace task
do_compile() {
    # New implementation
}

# Add task dependency
do_mytask() {
    echo "Custom task"
}
addtask mytask after do_compile before do_install
```

---

## ⚙️ Configuration Files

### local.conf

```bash
# build/conf/local.conf

#============================================
# MACHINE SELECTION
#============================================
MACHINE ?= "qemux86-64"
# MACHINE ?= "raspberrypi4-64"
# MACHINE ?= "beaglebone-yocto"

#============================================
# DISTRIBUTION
#============================================
DISTRO ?= "poky"

#============================================
# PARALLEL BUILD
#============================================
BB_NUMBER_THREADS ?= "8"      # BitBake tasks
PARALLEL_MAKE ?= "-j 8"       # Make parallelism

#============================================
# DOWNLOAD/CACHE DIRECTORIES
#============================================
DL_DIR ?= "${TOPDIR}/../downloads"
SSTATE_DIR ?= "${TOPDIR}/../sstate-cache"

#============================================
# PACKAGE MANAGEMENT
#============================================
PACKAGE_CLASSES ?= "package_ipk"  # ipk, deb, or rpm

#============================================
# EXTRA IMAGE FEATURES
#============================================
EXTRA_IMAGE_FEATURES ?= "debug-tweaks ssh-server-openssh"

# debug-tweaks: root without password, debugfs
# ssh-server-openssh: SSH server

#============================================
# SDK
#============================================
SDKMACHINE ?= "x86_64"

#============================================
# LICENSE MANAGEMENT
#============================================
LICENSE_FLAGS_ACCEPTED = "commercial"

#============================================
# DISTRO FEATURES
#============================================
DISTRO_FEATURES:append = " systemd"
DISTRO_FEATURES:remove = "sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"

#============================================
# ACCEPT ONLY FREE LICENSES
#============================================
# INCOMPATIBLE_LICENSE = "GPL-3.0* LGPL-3.0*"
```

### bblayers.conf

```bash
# build/conf/bblayers.conf
BBLAYERS ?= " \
    /home/user/poky/meta \
    /home/user/poky/meta-poky \
    /home/user/poky/meta-yocto-bsp \
    /home/user/meta-openembedded/meta-oe \
    /home/user/meta-openembedded/meta-python \
    /home/user/meta-raspberrypi \
    /home/user/meta-mylayer \
"
```

---

## 🖼️ Creating Custom Images

### Method 1: Image Recipe

```bash
# recipes-core/images/my-image.bb
SUMMARY = "My Custom Image"

IMAGE_INSTALL = " \
    packagegroup-core-boot \
    ${CORE_IMAGE_EXTRA_INSTALL} \
"

# Add packages
IMAGE_INSTALL:append = " \
    openssh \
    python3 \
    myapp \
    htop \
    vim \
"

# Image features
IMAGE_FEATURES += "ssh-server-openssh"

# Root filesystem size (KB)
IMAGE_ROOTFS_SIZE = "1048576"

inherit core-image
```

### Method 2: Package Groups

```bash
# recipes-core/packagegroups/packagegroup-myproduct.bb
SUMMARY = "My Product Package Group"

PACKAGE_ARCH = "${MACHINE_ARCH}"

inherit packagegroup

RDEPENDS:${PN} = " \
    myapp \
    mylib \
    myconfig \
"
```

### Method 3: local.conf Addition

```bash
# In local.conf
IMAGE_INSTALL:append = " htop vim nano"
```

---

## 📦 Build Outputs

```
build/tmp/deploy/
├── images/
│   └── <machine>/
│       ├── core-image-minimal-<machine>.rootfs.wic.gz    # Complete image
│       ├── core-image-minimal-<machine>.rootfs.ext4      # Ext4 filesystem
│       ├── core-image-minimal-<machine>.rootfs.tar.gz    # Tarball
│       ├── Image                                          # Kernel
│       ├── <machine>.dtb                                  # Device tree
│       └── modules-<machine>.tgz                          # Kernel modules
├── ipk/                                                   # IPK packages
│   └── <arch>/
│       └── myapp_1.0-r0_<arch>.ipk
├── licenses/                                              # License info
└── sdk/                                                   # SDK installers
    └── poky-glibc-x86_64-core-image-minimal-cortexa53-<machine>-toolchain-4.0.sh
```

---

## 🛠️ Essential BitBake Commands

```bash
# Initialize environment
source oe-init-build-env build

# Build recipes
bitbake core-image-minimal         # Complete image
bitbake myapp                      # Single recipe
bitbake myapp -c clean             # Clean recipe
bitbake myapp -c cleansstate       # Clean including sstate
bitbake world                      # Build everything

# Task execution
bitbake myapp -c fetch             # Only fetch
bitbake myapp -c compile           # Only compile
bitbake myapp -c devshell          # Open dev shell
bitbake myapp -c menuconfig        # Kernel menuconfig

# Debugging
bitbake -e myapp                   # Show environment
bitbake -e myapp | grep ^SRC_URI   # Filter specific var
bitbake -g core-image-minimal      # Generate dependency graphs
bitbake -DDD myapp                 # Debug output

# Layer management
bitbake-layers show-layers         # List active layers
bitbake-layers show-recipes        # List recipes
bitbake-layers add-layer ../meta-mylayer
bitbake-layers remove-layer meta-mylayer

# Finding recipes
bitbake-layers show-recipes "*kernel*"
oe-pkgdata-util list-pkgs          # List packages

# SDK
bitbake core-image-minimal -c populate_sdk
bitbake core-image-minimal -c populate_sdk_ext  # Extensible SDK
```

---

## 🔧 Common Customizations

### 1. Append to Existing Recipe

```bash
# recipes-apps/myapp/myapp_%.bbappend
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"

SRC_URI += "file://custom.patch"

EXTRA_OECMAKE += "-DCUSTOM_OPTION=ON"
```

### 2. Kernel Configuration

```bash
# recipes-kernel/linux/linux-yocto_%.bbappend
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"

SRC_URI += "file://custom.cfg"  # Kernel config fragment

# Or use defconfig
SRC_URI += "file://defconfig"
```

### 3. Add Systemd Service

```bash
# Enable systemd in distro
DISTRO_FEATURES:append = " systemd"
INIT_MANAGER = "systemd"

# In recipe
inherit systemd
SYSTEMD_SERVICE:${PN} = "myservice.service"
```

---

## 🎯 Interview Questions

### Q1: What is the difference between DEPENDS and RDEPENDS?
**A:** 
- **DEPENDS**: Build-time dependencies (needed to compile)
- **RDEPENDS**: Runtime dependencies (needed to run)
- Example: A C app DEPENDS on gcc but RDEPENDS on libc

### Q2: Explain sstate-cache. Why is it important?
**A:** Shared State Cache (sstate) stores the output of build tasks. When rebuilding, BitBake checks if sstate exists for unchanged inputs and reuses results. This dramatically speeds up rebuilds (hours → minutes).

### Q3: How do you modify an existing recipe without editing the original?
**A:** Use a `.bbappend` file in your layer:
- Same name as original: `recipe_%.bbappend` (any version) or `recipe_1.0.bbappend` (specific)
- Can add patches, modify variables, extend tasks
- Original recipe remains unchanged

### Q4: What is the difference between IMAGE_INSTALL and RDEPENDS?
**A:**
- **IMAGE_INSTALL**: Packages to include in the root filesystem image
- **RDEPENDS**: Runtime dependencies of a specific package
- IMAGE_INSTALL is for images; RDEPENDS is for recipes

### Q5: How do you debug a failing recipe?
**A:**
1. `bitbake -DDD myapp` - Verbose output
2. `bitbake myapp -c devshell` - Interactive shell in work directory
3. Check `tmp/work/<arch>/myapp/1.0-r0/temp/log.do_compile`
4. `bitbake -e myapp | grep VARIABLE` - Check variable values

---

## 📚 Best Practices

1. **Use layers** - Never modify meta or meta-poky directly
2. **Version recipes** - Include version in filename
3. **Use sstate** - Share sstate-cache across builds
4. **Pin versions** - Use SRCREV for git repos
5. **Document** - README in every layer
6. **Test builds** - Use QEMU for quick validation
7. **Keep local.conf minimal** - Move settings to distro layer

---

*Previous: [← Yocto Introduction](./01_Yocto_Introduction.md) | Next: [Ubuntu Core & Snaps →](./03_Ubuntu_Core_Snaps.md)*

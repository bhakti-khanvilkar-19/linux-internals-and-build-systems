# Yocto Project - Introduction

## 📚 What is the Yocto Project?

The **Yocto Project** is an open-source collaboration project that provides:
- Templates, tools, and methods for creating custom Linux-based systems
- A complete embedded Linux development environment
- Hardware-agnostic build system

> **Key Point:** Yocto is NOT a Linux distribution itself - it's a tool to CREATE your own custom Linux distribution.

---

## 🏗️ Core Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                        YOCTO PROJECT ECOSYSTEM                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │    POKY      │  │ OpenEmbedded │  │      BitBake             │   │
│  │ (Reference   │  │   (Build     │  │  (Build Execution       │   │
│  │  Distro)     │  │   Framework) │  │      Engine)            │   │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘   │
│         │                 │                      │                   │
│         └────────────────┼──────────────────────┘                   │
│                          ▼                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    METADATA (Recipes + Layers)                │   │
│  │  • Recipes (.bb)     - How to build packages                  │   │
│  │  • Classes (.bbclass) - Shared build logic                    │   │
│  │  • Config (.conf)     - Machine/distro settings               │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Key Terminology

### 1. **Poky**
The reference distribution of the Yocto Project.

```
Poky = BitBake + OpenEmbedded-Core + meta-poky + meta-yocto-bsp
```

| Component | Description |
|-----------|-------------|
| BitBake | Task scheduler/executor (like make on steroids) |
| OpenEmbedded-Core | Core recipes (base system, gcc, glibc, etc.) |
| meta-poky | Poky-specific configuration |
| meta-yocto-bsp | Reference BSP layers for supported boards |

### 2. **BitBake**
The build engine that:
- Parses recipes and configuration
- Resolves dependencies
- Executes tasks (fetch, compile, package, image)
- Manages parallel builds

```bash
# BitBake commands
bitbake <recipe-name>        # Build a recipe
bitbake <image-name>         # Build complete image
bitbake -c <task> <recipe>   # Run specific task
bitbake -e <recipe>          # Show environment
```

### 3. **Recipe (.bb file)**
A recipe is the fundamental unit - it describes:
- Where to get source code
- How to configure it
- How to compile it
- How to package it

```bash
# Example: hello-world_1.0.bb
SUMMARY = "Hello World Application"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE;md5=abc123..."

SRC_URI = "git://github.com/example/hello.git;branch=main"
SRCREV = "a1b2c3d4..."

S = "${WORKDIR}/git"

inherit cmake

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}
}
```

### 4. **Layer**
A collection of related recipes and configuration.

```
meta-<layer-name>/
├── conf/
│   └── layer.conf          # Layer configuration
├── recipes-<category>/
│   └── <recipe-name>/
│       ├── <recipe>.bb     # Main recipe
│       └── files/          # Patches, configs
├── classes/                # Shared classes
└── README                  # Documentation
```

**Common Layers:**
| Layer | Purpose |
|-------|---------|
| meta | OpenEmbedded-Core (base) |
| meta-poky | Poky distro settings |
| meta-yocto-bsp | Reference BSP |
| meta-openembedded | Extra recipes (networking, python, etc.) |
| meta-raspberrypi | Raspberry Pi BSP |
| meta-intel | Intel BSP |
| meta-arm | ARM BSP |

### 5. **BSP (Board Support Package)**
Hardware-specific layer containing:
- Kernel configuration
- Bootloader settings
- Machine definitions
- Hardware-specific drivers

### 6. **Image**
The final output - a complete filesystem image.

```bash
# Common image targets
core-image-minimal          # Minimal bootable image (~8MB)
core-image-base             # Console image with hardware support
core-image-full-cmdline     # Full-featured console
core-image-sato             # Graphical image (Sato UI)
core-image-weston           # Wayland/Weston graphics
```

---

## 📁 Directory Structure

```
poky/                              # Yocto source directory
├── bitbake/                       # BitBake tool
├── meta/                          # OpenEmbedded-Core
├── meta-poky/                     # Poky distro layer
├── meta-yocto-bsp/               # Reference BSP
├── oe-init-build-env             # Environment setup script
└── scripts/                       # Helper scripts

build/                             # Build directory (generated)
├── conf/
│   ├── local.conf                # Local build configuration
│   └── bblayers.conf             # Layer configuration
├── tmp/                          # Build output
│   ├── deploy/
│   │   └── images/<machine>/     # Final images here!
│   ├── work/                     # Per-recipe work directories
│   └── sysroots/                 # SDK sysroots
└── downloads/                    # Downloaded source archives
```

---

## 🚀 Quick Start

### Prerequisites (Ubuntu 24.04)

```bash
# Install required packages
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential \
    chrpath socat cpio python3 python3-pip python3-pexpect xz-utils \
    debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa \
    libsdl1.2-dev pylint xterm zstd liblz4-tool

# Recommended: 100GB+ free disk, 8GB+ RAM
```

### Clone and Build

```bash
# Clone Poky (Scarthgap release - LTS)
git clone -b scarthgap git://git.yoctoproject.org/poky
cd poky

# Initialize build environment
source oe-init-build-env build

# Edit local.conf for your machine
# MACHINE = "qemux86-64"  (default)
# MACHINE = "raspberrypi4"
# MACHINE = "beaglebone-yocto"

# Build minimal image (first build takes 1-3 hours)
bitbake core-image-minimal

# Run in QEMU
runqemu qemux86-64
```

---

## 🔄 Release Cycle

| Release Name | Version | LTS | Support Until |
|--------------|---------|-----|---------------|
| Scarthgap | 5.0 | ✅ | April 2028 |
| Nanbield | 4.3 | ❌ | November 2024 |
| Mickledore | 4.2 | ❌ | May 2024 |
| Kirkstone | 4.0 | ✅ | April 2026 |
| Dunfell | 3.1 | ✅ | April 2024 |

> **Recommendation:** Use LTS releases (Scarthgap or Kirkstone) for production.

---

## 📊 Yocto vs Other Build Systems

| Feature | Yocto | Buildroot | OpenWrt |
|---------|-------|-----------|---------|
| **Complexity** | High | Low | Medium |
| **Flexibility** | Maximum | Limited | Network-focused |
| **Package Mgmt** | IPK/RPM/DEB | None | opkg |
| **Partial Builds** | Yes | Difficult | Yes |
| **SDK Generation** | Built-in | Limited | Yes |
| **Commercial Support** | Yes | Limited | Community |
| **Learning Curve** | Steep | Gradual | Moderate |

---

## 💡 Key Concepts Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOCTO KEY CONCEPTS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RECIPE (.bb)                                                    │
│  ├── What to build                                               │
│  ├── Where to get source                                         │
│  └── How to compile/install                                      │
│                                                                  │
│  LAYER (meta-*)                                                  │
│  ├── Collection of recipes                                       │
│  ├── Organized by purpose                                        │
│  └── Can override other layers                                   │
│                                                                  │
│  MACHINE                                                         │
│  ├── Target hardware definition                                  │
│  ├── Kernel/bootloader config                                    │
│  └── Hardware-specific settings                                  │
│                                                                  │
│  DISTRO                                                          │
│  ├── Distribution policy                                         │
│  ├── Init system (systemd/sysvinit)                              │
│  └── Default features                                            │
│                                                                  │
│  IMAGE                                                           │
│  ├── Collection of packages                                      │
│  ├── Root filesystem                                             │
│  └── Final deployable output                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Interview Questions

### Q1: What is the difference between Yocto and Poky?
**A:** Yocto Project is the umbrella project providing tools and methodology. Poky is the reference distribution that combines BitBake + OpenEmbedded-Core + example layers. You use Poky as a starting point to build your custom distribution.

### Q2: What is BitBake?
**A:** BitBake is the task execution engine (similar to make) that parses recipes, resolves dependencies, and executes build tasks. It's Python-based and handles parallel builds, caching, and shared state.

### Q3: What is a recipe and what does it contain?
**A:** A recipe (.bb file) is metadata describing how to build a software component. It contains:
- Source location (SRC_URI)
- Version and license info
- Dependencies (DEPENDS, RDEPENDS)
- Build instructions (do_compile, do_install)
- Package definitions

### Q4: What is a layer and why use layers?
**A:** A layer is a collection of related recipes. Benefits:
- Organization and modularity
- Reusability across projects
- Prioritization (higher priority layers override lower)
- Separation of concerns (BSP vs distro vs application)

### Q5: Explain the build workflow in Yocto.
**A:** 
1. **Parse** - BitBake reads recipes and configuration
2. **Fetch** - Download sources (do_fetch)
3. **Unpack** - Extract sources (do_unpack)
4. **Patch** - Apply patches (do_patch)
5. **Configure** - Run configure scripts (do_configure)
6. **Compile** - Build the software (do_compile)
7. **Install** - Install to staging (do_install)
8. **Package** - Create packages (do_package)
9. **Image** - Assemble final image (do_rootfs)

---

## 📚 Resources

- [Yocto Project Documentation](https://docs.yoctoproject.org/)
- [OpenEmbedded Layer Index](https://layers.openembedded.org/)
- [Yocto Project Quick Build](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html)

---

*Next: [Yocto Build Process →](./02_Yocto_Build_Process.md)*

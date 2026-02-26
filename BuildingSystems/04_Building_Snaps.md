# Building Snaps with Snapcraft

## 📚 Overview

**Snapcraft** is the tool for building snaps. This guide covers:
- snapcraft.yaml syntax
- Building different snap types
- Plugins and parts
- Publishing to the Snap Store

---

## 🛠️ Installing Snapcraft

```bash
# Install snapcraft
sudo snap install snapcraft --classic

# Verify installation
snapcraft --version

# For LXD-based builds (recommended)
sudo snap install lxd
sudo lxd init --auto
sudo usermod -aG lxd $USER
newgrp lxd
```

---

## 📝 snapcraft.yaml Structure

```yaml
# snapcraft.yaml - Complete reference

#============================================
# METADATA
#============================================
name: myapp                    # Snap name (store identifier)
version: '1.0.0'              # Version string
summary: My Application        # Short description (79 chars max)
description: |                 # Long description
  A longer description of my application
  that can span multiple lines.

#============================================
# BASE AND CONFINEMENT
#============================================
base: core24                   # Base snap (core24, core22, core20)
grade: stable                  # stable or devel
confinement: strict            # strict, devmode, or classic

#============================================
# ARCHITECTURES
#============================================
architectures:
  - build-on: amd64
    build-for: amd64
  - build-on: amd64
    build-for: arm64

#============================================
# APPS (EXECUTABLES/SERVICES)
#============================================
apps:
  myapp:                       # Command name (snap run myapp.myapp)
    command: bin/myapp         # Path within snap
    daemon: simple             # Optional: makes it a service
    restart-condition: always
    plugs:                     # Interfaces this app needs
      - network
      - home
    environment:
      LC_ALL: C.UTF-8

  helper:                      # Additional command
    command: bin/helper
    
#============================================
# PARTS (BUILD INSTRUCTIONS)
#============================================
parts:
  myapp:
    plugin: cmake              # Build plugin
    source: .                  # Source location
    source-type: local         # git, tar, local, etc.
    build-packages:            # Packages needed to build
      - build-essential
      - libssl-dev
    stage-packages:            # Packages needed to run
      - libssl3
    cmake-parameters:
      - -DCMAKE_BUILD_TYPE=Release
      
#============================================
# LAYOUT (FILESYSTEM REMAPPING)
#============================================
layout:
  /usr/share/myapp:
    bind: $SNAP/usr/share/myapp
    
#============================================
# PLUGS/SLOTS (INTERFACES)
#============================================
plugs:
  my-serial:
    interface: serial-port
    path: /dev/ttyUSB0
    
slots:
  my-dbus:
    interface: dbus
    bus: session
    name: com.example.myapp
```

---

## 🔌 Common Plugins

### Autotools Plugin

```yaml
parts:
  myapp:
    plugin: autotools
    source: https://example.com/myapp-1.0.tar.gz
    autotools-configure-parameters:
      - --prefix=/usr
      - --enable-feature
```

### CMake Plugin

```yaml
parts:
  myapp:
    plugin: cmake
    source: .
    cmake-parameters:
      - -DCMAKE_BUILD_TYPE=Release
      - -DCMAKE_INSTALL_PREFIX=/usr
```

### Meson Plugin

```yaml
parts:
  myapp:
    plugin: meson
    source: .
    meson-parameters:
      - --buildtype=release
```

### Python Plugin

```yaml
parts:
  myapp:
    plugin: python
    source: .
    python-packages:
      - requests
      - flask
```

### Go Plugin

```yaml
parts:
  myapp:
    plugin: go
    source: .
    build-snaps:
      - go/latest/stable
```

### Rust Plugin

```yaml
parts:
  myapp:
    plugin: rust
    source: .
    rust-channel: stable
```

### NPM Plugin

```yaml
parts:
  myapp:
    plugin: npm
    source: .
    npm-node-version: "18.17.0"
```

### Nil Plugin (Manual)

```yaml
parts:
  myapp:
    plugin: nil
    source: .
    override-build: |
      make
      make install DESTDIR=$SNAPCRAFT_PART_INSTALL
```

### Dump Plugin (Pre-built)

```yaml
parts:
  myapp:
    plugin: dump
    source: ./prebuilt/
    organize:
      'myapp': bin/myapp
```

---

## 🏗️ Example: Complete C Application

```yaml
name: hello-snap
version: '1.0'
summary: Hello World Snap
description: |
  A simple hello world application demonstrating
  snap packaging.

base: core24
grade: stable
confinement: strict

apps:
  hello:
    command: usr/bin/hello
    plugs:
      - network

parts:
  hello:
    plugin: cmake
    source: https://github.com/example/hello.git
    source-tag: v1.0.0
    build-packages:
      - build-essential
      - cmake
    cmake-parameters:
      - -DCMAKE_INSTALL_PREFIX=/usr
```

---

## 🐍 Example: Python Application

```yaml
name: mypy-app
version: '2.0'
summary: Python Application
description: A Python web application

base: core24
grade: stable
confinement: strict

apps:
  mypy-app:
    command: bin/myapp
    daemon: simple
    restart-condition: always
    plugs:
      - network
      - network-bind

parts:
  mypy-app:
    plugin: python
    source: .
    python-requirements:
      - requirements.txt
    stage-packages:
      - python3-venv
    
  # Additional part for static files
  static-files:
    plugin: dump
    source: static/
    organize:
      '*': share/myapp/static/
```

---

## ⚙️ Example: Daemon/Service

```yaml
name: my-daemon
version: '1.0'
summary: Background Service
description: A system daemon

base: core24
grade: stable
confinement: strict

apps:
  my-daemon:
    command: bin/my-daemon
    daemon: simple                    # simple, forking, oneshot, notify
    restart-condition: on-failure     # always, on-failure, on-success, never
    restart-delay: 10s
    stop-timeout: 30s
    start-timeout: 10s
    watchdog-timeout: 30s
    install-mode: enable              # enable or disable
    plugs:
      - network
      - network-bind
    
  ctl:                               # Control command
    command: bin/my-daemon-ctl
    plugs:
      - network

parts:
  my-daemon:
    plugin: make
    source: .
```

---

## 🔧 Build Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SNAPCRAFT BUILD LIFECYCLE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────┐                                                          │
│  │   PULL     │──▶ Download/fetch source code                            │
│  └─────┬──────┘    (snapcraft pull)                                      │
│        │                                                                 │
│        ▼                                                                 │
│  ┌────────────┐                                                          │
│  │   BUILD    │──▶ Compile source code                                   │
│  └─────┬──────┘    (snapcraft build)                                     │
│        │                                                                 │
│        ▼                                                                 │
│  ┌────────────┐                                                          │
│  │   STAGE    │──▶ Copy built artifacts to staging area                  │
│  └─────┬──────┘    (snapcraft stage)                                     │
│        │                                                                 │
│        ▼                                                                 │
│  ┌────────────┐                                                          │
│  │   PRIME    │──▶ Copy staged files to priming area                     │
│  └─────┬──────┘    (snapcraft prime)                                     │
│        │                                                                 │
│        ▼                                                                 │
│  ┌────────────┐                                                          │
│  │   SNAP     │──▶ Create final .snap file                               │
│  └────────────┘    (snapcraft snap / snapcraft)                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘

Working Directory Structure:
parts/
├── <part-name>/
│   ├── src/          # Source code (after pull)
│   ├── build/        # Build directory
│   └── install/      # Install directory (after build)
stage/                # Combined staging area (after stage)
prime/                # Final content (after prime)
*.snap                # Output snap file
```

---

## 🔀 Override Scripts

```yaml
parts:
  myapp:
    plugin: nil
    source: .
    
    override-pull: |
      snapcraftctl pull
      # Custom pull logic
      git submodule update --init
      
    override-build: |
      # Custom build
      ./configure --prefix=$SNAPCRAFT_PART_INSTALL/usr
      make -j$(nproc)
      make install
      
    override-stage: |
      snapcraftctl stage
      # Post-stage processing
      
    override-prime: |
      snapcraftctl prime
      # Strip debugging symbols
      find . -type f -executable -exec strip {} \;
```

---

## 🔐 Building System Snaps

### Kernel Snap

```yaml
name: mykernel
version: '5.15.0'
summary: Custom Linux Kernel
description: Custom kernel for my device
type: kernel

base: core24
grade: stable
build-base: core24

parts:
  kernel:
    plugin: kernel
    source: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
    source-tag: v5.15.100
    source-type: git
    kdefconfig:
      - defconfig
    kconfigs:
      - CONFIG_DEBUG_INFO=n
      - CONFIG_MODULES=y
    kernel-image-target: Image
    kernel-with-firmware: false
    
  firmware:
    plugin: nil
    source: linux-firmware
    override-build: |
      mkdir -p $SNAPCRAFT_PART_INSTALL/firmware
      cp -r * $SNAPCRAFT_PART_INSTALL/firmware/
```

### Gadget Snap

```yaml
name: mygadget
version: '1.0'
summary: Gadget snap for my device
description: Contains bootloader and device config
type: gadget

base: core24
grade: stable

parts:
  gadget:
    plugin: nil
    source: .
    override-build: |
      # Copy gadget.yaml
      cp gadget.yaml $SNAPCRAFT_PART_INSTALL/
      # Copy boot assets
      cp -r boot/* $SNAPCRAFT_PART_INSTALL/
```

**gadget.yaml for x86:**

```yaml
volumes:
  pc:
    bootloader: grub
    structure:
      - name: mbr
        type: mbr
        size: 440
        content:
          - image: pc-boot.img
      - name: BIOS Boot
        type: DA,21686148-6449-6E6F-744E-656564454649
        size: 1M
        offset: 1M
        offset-write: mbr+92
        content:
          - image: pc-core.img
      - name: EFI System
        type: EF,C12A7328-F81F-11D2-BA4B-00A0C93EC93B
        filesystem: vfat
        filesystem-label: ubuntu-seed
        size: 1200M
        role: system-seed
      - name: ubuntu-boot
        type: 83,0FC63DAF-8483-4772-8E79-3D69D8477DE4
        filesystem: ext4
        filesystem-label: ubuntu-boot
        size: 750M
        role: system-boot
      - name: ubuntu-save
        type: 83,0FC63DAF-8483-4772-8E79-3D69D8477DE4
        filesystem: ext4
        filesystem-label: ubuntu-save
        size: 16M
        role: system-save
      - name: ubuntu-data
        type: 83,0FC63DAF-8483-4772-8E79-3D69D8477DE4
        filesystem: ext4
        filesystem-label: ubuntu-data
        role: system-data
```

---

## 🚀 Build Commands

```bash
#============================================
# BUILDING
#============================================
snapcraft                          # Full build
snapcraft --debug                  # Debug mode (shell on failure)
snapcraft --verbose                # Verbose output
snapcraft --use-lxd                # Build in LXD container
snapcraft --destructive-mode       # Build on host (careful!)

#============================================
# STEP-BY-STEP
#============================================
snapcraft pull                     # Pull sources only
snapcraft build                    # Build only
snapcraft stage                    # Stage only
snapcraft prime                    # Prime only
snapcraft snap                     # Create snap only

#============================================
# CLEANING
#============================================
snapcraft clean                    # Clean all
snapcraft clean mypart             # Clean specific part
snapcraft clean --step=build       # Clean from build step

#============================================
# TESTING
#============================================
sudo snap install myapp.snap --dangerous --devmode
snap run myapp
snap run --shell myapp             # Debug shell

#============================================
# REMOTE BUILD
#============================================
snapcraft remote-build             # Build on Launchpad
```

---

## 📤 Publishing to Snap Store

### 1. Create Account & Register

```bash
# Login to Snapcraft
snapcraft login

# Register snap name
snapcraft register myapp

# Check registered names
snapcraft list-registered
```

### 2. Upload and Release

```bash
# Upload snap
snapcraft upload myapp_1.0_amd64.snap

# Release to channel
snapcraft release myapp 1 stable  # revision 1 to stable

# Or upload and release together
snapcraft upload myapp_1.0_amd64.snap --release=stable
```

### 3. Manage Releases

```bash
# List revisions
snapcraft revisions myapp

# Release to multiple channels
snapcraft release myapp 5 stable,candidate

# Close channel
snapcraft close myapp beta
```

---

## 🔒 Confinement & Interfaces

### Requesting Interfaces

```yaml
# In snapcraft.yaml
apps:
  myapp:
    plugs:
      - network
      - network-bind
      - serial-port
      - gpio
      
plugs:
  serial-port:
    interface: serial-port
    path: /dev/ttyUSB0
```

### Auto-Connect

Some interfaces auto-connect, others need manual approval:

| Auto-Connect | Manual Request |
|--------------|----------------|
| network | serial-port |
| network-bind | raw-usb |
| home (read) | gpio |
| opengl | i2c |
| audio-playback | spi |

```bash
# Request auto-connect in store
# Submit request via snapcraft.io dashboard
```

---

## 📊 Common Patterns

### Multi-Part Builds

```yaml
parts:
  library:
    plugin: cmake
    source: lib/
    
  application:
    plugin: cmake
    source: app/
    after:
      - library    # Build after library
    build-environment:
      - CFLAGS: "-I$SNAPCRAFT_STAGE/usr/include"
      - LDFLAGS: "-L$SNAPCRAFT_STAGE/usr/lib"
```

### Organizing Files

```yaml
parts:
  myapp:
    plugin: dump
    source: ./dist/
    organize:
      'linux-amd64/myapp': usr/bin/myapp
      'config.yaml': etc/myapp/config.yaml
    stage:
      - usr/bin/myapp
      - etc/myapp/
    prime:
      - usr/bin/myapp
      - etc/myapp/
```

### Environment Variables

```yaml
apps:
  myapp:
    command: bin/myapp
    environment:
      PATH: $SNAP/usr/bin:$PATH
      LD_LIBRARY_PATH: $SNAP/usr/lib:$LD_LIBRARY_PATH
      MY_CONFIG: $SNAP/etc/myapp/config.yaml
```

---

## 🎯 Interview Questions

### Q1: What is the difference between build-packages and stage-packages?
**A:**
- **build-packages**: APT packages needed at *build time* (compilers, dev headers). Not included in final snap.
- **stage-packages**: APT packages needed at *runtime*. Included in the snap.

### Q2: Explain the snap build lifecycle stages.
**A:**
1. **Pull**: Download source code
2. **Build**: Compile/build the software
3. **Stage**: Copy build artifacts to staging area (combine all parts)
4. **Prime**: Copy to final area (filter what goes into snap)
5. **Snap**: Package into .snap file

### Q3: What is a base snap and why is it important?
**A:** The base snap (core24, core22, etc.) provides the runtime environment - glibc, basic utilities, and libraries. Apps depend on these shared libraries. Using the right base ensures compatibility and reduces snap size (libraries aren't duplicated).

### Q4: How do you debug a snap that's not working?
**A:**
1. Install with `--devmode` for relaxed confinement
2. `snap run --shell myapp` for interactive shell
3. Check `snap logs myapp`
4. `snapcraft --debug` for build issues
5. Check `/var/snap/myapp/` for runtime data

### Q5: What's the difference between strict and classic confinement?
**A:**
- **Strict**: Sandboxed, can only access resources via interfaces. Default and recommended.
- **Classic**: No sandbox, full system access like traditional packages. Requires store approval. Used for development tools (IDEs, compilers).

---

## 📚 Resources

- [Snapcraft Documentation](https://snapcraft.io/docs)
- [Snapcraft Reference](https://snapcraft.io/docs/snapcraft-yaml-reference)
- [Snapcraft Plugins](https://snapcraft.io/docs/snapcraft-plugins)
- [Snap Store Dashboard](https://snapcraft.io/snaps)

---

*Previous: [← Ubuntu Core & Snaps](./03_Ubuntu_Core_Snaps.md)*

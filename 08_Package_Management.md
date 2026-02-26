# Linux Package Management — APT, dpkg, Snap & More

## Objective
Master software installation on Ubuntu: understand package formats, repositories, and tools like APT, dpkg, Snap, and Flatpak.

> **Interactive Diagram**: See [diagrams/package_management_flow.drawio](diagrams/package_management_flow.drawio) for APT workflow visualization.

---

## 1) Package Management Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      UBUNTU PACKAGE MANAGEMENT STACK                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌─────────────────────────────────────────────────────────────────────────────┐   │
│   │                           APT (apt, apt-get)                                │   │
│   │                  High-level: Dependencies, Repositories                     │   │
│   └─────────────────────────────────────┬───────────────────────────────────────┘   │
│                                         │                                           │
│                                         ▼                                           │
│   ┌─────────────────────────────────────────────────────────────────────────────┐   │
│   │                              dpkg                                           │   │
│   │                   Low-level: Install/Remove .deb files                      │   │
│   └─────────────────────────────────────┬───────────────────────────────────────┘   │
│                                         │                                           │
│                                         ▼                                           │
│   ┌─────────────────────────────────────────────────────────────────────────────┐   │
│   │                          .deb Package Files                                 │   │
│   │                  (archives with binaries + metadata)                        │   │
│   └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
│   ALSO AVAILABLE:                                                                   │
│   • Snap - Universal, sandboxed packages (snapd)                                    │
│   • Flatpak - Universal, sandboxed (mainly GUI apps)                                │
│   • AppImage - Single portable executable                                           │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) APT — The Primary Package Manager

### Update Package Lists

```bash
# ALWAYS run this first!
sudo apt update                      # Refresh package lists from repos
                                     # Does NOT install anything
```

### Installing Packages

```bash
# Install single package
sudo apt install nginx

# Install multiple packages
sudo apt install nginx php mysql-server

# Install without prompts
sudo apt install -y nginx

# Install specific version
sudo apt install nginx=1.18.0-0ubuntu1

# Install .deb file (apt handles dependencies!)
sudo apt install ./package.deb

# Reinstall package
sudo apt reinstall nginx

# Install recommended packages too
sudo apt install --install-recommends nginx

# Only download, don't install
sudo apt install --download-only nginx
```

### Upgrading Packages

```bash
# Upgrade all packages (safe)
sudo apt upgrade

# Upgrade with removals if needed
sudo apt full-upgrade              # May remove packages for upgrade

# Upgrade single package
sudo apt install --only-upgrade nginx

# See what would be upgraded
apt list --upgradable
```

### Removing Packages

```bash
# Remove package (keep config files)
sudo apt remove nginx

# Remove package AND config files
sudo apt purge nginx

# Remove unused dependencies
sudo apt autoremove

# Clean package cache
sudo apt clean                     # Remove all cached packages
sudo apt autoclean                 # Remove old cached packages
```

### Searching & Information

```bash
# Search for packages
apt search nginx
apt search "web server"

# Show package details
apt show nginx

# List installed packages
apt list --installed
apt list --installed | grep nginx

# List all versions available
apt list -a nginx

# Check if package is installed
dpkg -l | grep nginx

# Show package files
dpkg -L nginx                      # List files in package
dpkg -S /etc/nginx/nginx.conf      # Which package owns file?
```

---

## 3) dpkg — Low-Level Tool

dpkg works directly with .deb files but doesn't resolve dependencies.

```bash
# Install .deb file
sudo dpkg -i package.deb

# Fix broken dependencies after dpkg
sudo apt install -f              # or: sudo apt --fix-broken install

# Remove package
sudo dpkg -r package-name        # Keep config
sudo dpkg -P package-name        # Purge config too

# List installed packages
dpkg -l
dpkg -l | grep nginx

# Package status
dpkg -s nginx

# List package contents
dpkg -L nginx

# Find package owning a file
dpkg -S /bin/ls

# Extract .deb without installing
dpkg -x package.deb /target/dir

# Show .deb contents
dpkg -c package.deb
```

---

## 4) Repositories Configuration

### Repository Files

```bash
# Main sources list
cat /etc/apt/sources.list

# Additional sources (modern way)
ls /etc/apt/sources.list.d/

# Example Ubuntu 24.04 entry:
# deb http://archive.ubuntu.com/ubuntu/ noble main restricted universe multiverse
# │   │                                  │     │
# │   │                                  │     └── Components (main, restricted, etc.)
# │   │                                  └── Distribution codename
# │   └── Repository URL
# └── Type (deb for binaries, deb-src for source)
```

### Ubuntu Components

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        UBUNTU REPOSITORY COMPONENTS                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Component       │ Free? │ Supported? │ Contents                                   │
│   ────────────────┼───────┼────────────┼──────────────────────────────────────      │
│   main            │ Yes   │ Yes        │ Core Ubuntu packages                       │
│   restricted      │ No    │ Yes        │ Proprietary drivers (nvidia, etc.)         │
│   universe        │ Yes   │ Community  │ Community-maintained free software         │
│   multiverse      │ No    │ No         │ Non-free software                          │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Pocket          │ Purpose                                                         │
│   ────────────────┼──────────────────────────────────────────────────────           │
│   noble           │ Main release packages                                           │
│   noble-updates   │ Recommended updates                                             │
│   noble-security  │ Security updates (critical!)                                    │
│   noble-backports │ Newer versions backported                                       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Adding Repositories

```bash
# Add PPA (Personal Package Archive)
sudo add-apt-repository ppa:user/ppa-name
sudo apt update

# Remove PPA
sudo add-apt-repository --remove ppa:user/ppa-name

# Add third-party repository
# Example: Adding Docker repository

# 1. Add GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 2. Add repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 3. Update and install
sudo apt update
sudo apt install docker-ce
```

---

## 5) Snap Packages

Snap packages are self-contained, sandboxed, and auto-updating.

```bash
# Snap basics
snap find nginx                  # Search
snap info nginx                  # Details
sudo snap install nginx          # Install
sudo snap remove nginx           # Remove

# List installed snaps
snap list

# Snap channels (versions)
sudo snap install firefox --channel=esr/stable
sudo snap refresh firefox --channel=latest/stable

# Snap updates
sudo snap refresh                # Update all snaps
sudo snap refresh firefox        # Update specific snap

# Revert to previous version
sudo snap revert firefox

# Snap services
sudo snap services
sudo snap start nginx
sudo snap stop nginx
sudo snap restart nginx

# Disable snap auto-updates (temporary)
sudo snap refresh --hold         # Hold all updates
sudo snap refresh --unhold       # Resume updates

# Connect/disconnect interfaces
snap connections firefox
snap connect firefox:camera
snap disconnect firefox:camera
```

### Snap vs APT Comparison

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           SNAP vs APT (deb)                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Feature            │ APT (.deb)              │ Snap                               │
│   ───────────────────┼─────────────────────────┼────────────────────────────        │
│   Dependencies       │ Shared libraries        │ Bundled (larger size)              │
│   Updates            │ Manual (apt upgrade)    │ Automatic (background)             │
│   Rollback           │ Complex                 │ Easy (snap revert)                 │
│   Sandboxing         │ No                      │ Yes (AppArmor)                     │
│   Startup time       │ Fast                    │ Slower (first launch)              │
│   Disk space         │ Efficient               │ Uses more space                    │
│   Multiple versions  │ No                      │ Yes (parallel install)             │
│   System integration │ Deep                    │ Limited by confinement             │
│                                                                                     │
│   Use APT for: System tools, servers, when disk space matters                       │
│   Use Snap for: Desktop apps, when you want auto-updates and sandboxing             │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6) Flatpak (Alternative)

Flatpak is popular for desktop applications (not pre-installed on Ubuntu).

```bash
# Install Flatpak
sudo apt install flatpak
sudo apt install gnome-software-plugin-flatpak

# Add Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Restart system, then:

# Search and install
flatpak search gimp
flatpak install flathub org.gimp.GIMP

# Run
flatpak run org.gimp.GIMP

# List installed
flatpak list

# Update
flatpak update

# Remove
flatpak uninstall org.gimp.GIMP

# Remove unused runtimes
flatpak uninstall --unused
```

---

## 7) AppImage

Portable applications that don't need installation.

```bash
# Download AppImage file
wget https://example.com/app.AppImage

# Make executable
chmod +x app.AppImage

# Run
./app.AppImage

# Optional: Move to PATH
sudo mv app.AppImage /usr/local/bin/app
```

---

## 8) Package Pinning & Holding

### Hold Packages (Prevent Updates)

```bash
# Hold package at current version
sudo apt-mark hold nginx

# Show held packages
apt-mark showhold

# Unhold package
sudo apt-mark unhold nginx
```

### Pinning (Priority)

```bash
# Create pin file
sudo nano /etc/apt/preferences.d/pin-nginx

# Content:
Package: nginx
Pin: version 1.18*
Pin-Priority: 1001
```

---

## 9) Troubleshooting

### Common Issues

```bash
# Fix broken packages
sudo apt --fix-broken install

# Reconfigure package
sudo dpkg-reconfigure nginx

# Force remove broken package
sudo dpkg --remove --force-remove-reinstreq package-name

# Clear apt cache completely
sudo rm -rf /var/lib/apt/lists/*
sudo apt update

# Fix "Could not get lock" error
# (Another apt process running)
sudo killall apt apt-get
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/apt/lists/lock
sudo dpkg --configure -a

# Fix interrupted dpkg
sudo dpkg --configure -a

# Check package integrity
sudo debsums -c package-name
```

### Logs

```bash
# APT history
cat /var/log/apt/history.log

# dpkg log
cat /var/log/dpkg.log

# Recent installations
grep " install " /var/log/dpkg.log | tail -20
```

---

## 10) Building from Source

When packages aren't available:

```bash
# Install build tools
sudo apt install build-essential
sudo apt install autoconf automake libtool

# Typical build process
wget https://example.com/software.tar.gz
tar xzf software.tar.gz
cd software

# Using autotools
./configure --prefix=/usr/local
make
sudo make install

# Using CMake
mkdir build && cd build
cmake ..
make
sudo make install

# Using Meson
meson setup build
cd build
ninja
sudo ninja install

# Remove (if supported)
sudo make uninstall
```

---

## 11) Managing Package Caches

```bash
# View cache size
du -sh /var/cache/apt/archives/

# Clean all cached packages
sudo apt clean

# Clean only outdated packages
sudo apt autoclean

# Remove orphaned packages
sudo apt autoremove

# Snap cache
du -sh /var/lib/snapd/cache/
sudo rm -rf /var/lib/snapd/cache/*
```

---

## 12) Practical Exercises

### Exercise 1: Package Workflow
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Search for and install a package
apt search htop
apt show htop
sudo apt install htop

# Verify installation
which htop
dpkg -L htop | head
```

### Exercise 2: Install from External Repo
```bash
# Add VS Code repository
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list

sudo apt update
sudo apt install code
```

### Exercise 3: Snap Experience
```bash
# Install VLC via snap
sudo snap install vlc

# Check confinement
snap info vlc | grep confinement

# View connections
snap connections vlc
```

---

## 13) Quick Reference

| Task | Command |
|------|---------|
| Update package list | `sudo apt update` |
| Upgrade all packages | `sudo apt upgrade` |
| Install package | `sudo apt install pkg` |
| Remove package | `sudo apt remove pkg` |
| Remove + config | `sudo apt purge pkg` |
| Search packages | `apt search keyword` |
| Package info | `apt show pkg` |
| List installed | `apt list --installed` |
| Fix broken | `sudo apt --fix-broken install` |
| Clean cache | `sudo apt clean` |
| Remove orphans | `sudo apt autoremove` |
| Hold package | `sudo apt-mark hold pkg` |
| Install snap | `sudo snap install pkg` |
| Update snaps | `sudo snap refresh` |

---

## 14) Interview Questions

**Q1: What's the difference between apt and dpkg?**
> dpkg is low-level and handles individual .deb files without dependency resolution. apt is high-level, uses dpkg underneath, and handles dependencies and repositories.

**Q2: What's the difference between `apt remove` and `apt purge`?**
> `remove` deletes the package but keeps configuration files. `purge` removes both the package and its configuration files.

**Q3: How do you add a third-party repository securely?**
> Import the GPG key, add the repository with signed-by pointing to the key, update apt, then install packages.

**Q4: What is a PPA?**
> Personal Package Archive - a repository hosted on Launchpad containing packages built by individuals/teams, commonly used for newer software versions.

**Q5: When would you use Snap vs APT?**
> APT for system packages, server software, and when disk space/startup time matters. Snap for desktop apps that need sandboxing, auto-updates, or when you want the latest version.

---

*Next: [09_Processes.md](09_Processes.md) — Process management, monitoring, and control.*

# Linux Learning Notes - Ubuntu 24.04

## 📚 Study Guide

This repository contains systematic Linux notes from beginner to advanced level.

---

## 🗂️ Topics Index

| # | Topic | Description | Status |
|---|-------|-------------|--------|
| 1 | [Fundamentals](./01_Fundamentals.md) | What is Linux, distributions, architecture | ✅ |
| 2 | [Boot Process](./02_Boot_Process.md) | Detailed x86 & embedded boot, secure boot | ✅ |
| 3 | [Distributions & Packages](./03_Distributions_PackageManagement.md) | Distros, dpkg vs apt, repos, Snap/Flatpak | ✅ |
| 4 | [RootFS In-Depth](./04_RootFS_In_Depth.md) | Root filesystem structure, FHS, virtual FS, embedded vs desktop | ✅ |
| 5 | [File System](./05_File_System.md) | Directory structure, paths, navigation | ✅ |
| 6 | [Basic Commands](./06_Basic_Commands.md) | Essential terminal commands | ✅ |
| 7 | [Users & Permissions](./07_Users_Permissions.md) | Users, groups, chmod, chown | ✅ |
| 8 | [Package Management](./08_Package_Management.md) | APT, dpkg, snap | ✅ |
| 9 | [Processes](./09_Processes.md) | ps, top, kill, background jobs | ✅ |
| 10 | [Services & Systemd](./10_Services_Systemd.md) | Managing services, systemctl | ✅ |
| 11 | [Networking](./11_Networking.md) | IP, ports, firewall, SSH | ✅ |
| 12 | [Storage & Disks](./12_Storage_Disks.md) | Partitions, mounting, LVM | ✅ |
| 13 | [Shell Scripting](./13_Shell_Scripting.md) | Bash scripts, automation | ✅ |
| 14 | [Text Processing](./14_Text_Processing.md) | grep, sed, awk, pipes | ✅ |
| 15 | [System Administration](./15_System_Admin.md) | Logs, cron, backup, security | ✅ |

### 📊 Diagrams

| Diagram | Description |
|---------|-------------|
| [Embedded Boot Cycle](./diagrams/embedded_boot_cycle.drawio) | Interactive diagram showing embedded Linux boot with secure boot chain |
| [Distros & Packages](./diagrams/distros_and_packages.drawio) | Distro family tree, dpkg vs apt flow, package install steps |
| [RootFS Structure](./diagrams/rootfs_structure.drawio) | Root filesystem hierarchy, FHS directories, virtual filesystems, storage types |
| [Distro Family Tree](./diagrams/distro_family_tree.drawio) | Distribution relationships, package managers, comparison table |
| [Processes](./diagrams/processes.drawio) | Process hierarchy, states, signals, job control |
| [Systemd Architecture](./diagrams/systemd_architecture.drawio) | systemd as PID 1, unit types, boot targets, systemctl commands |
| [Networking Overview](./diagrams/networking_overview.drawio) | OSI/TCP-IP models, ports, IP addressing, UFW firewall, SSH |
| [LVM Architecture](./diagrams/lvm_architecture.drawio) | LVM layers (PV→VG→LV→FS), commands, benefits |
| [Text Processing](./diagrams/text_processing.drawio) | grep/sed/awk tools, pipelines, regex reference |
| [Bash Scripting](./diagrams/bash_scripting.drawio) | Script structure, variables, loops, functions, error handling |
| [System Administration](./diagrams/system_admin.drawio) | Logs, cron, security, backup, monitoring, troubleshooting |

> Open `.drawio` files in VS Code (with Draw.io extension) or at [app.diagrams.net](https://app.diagrams.net)

---

## 🚀 Quick Reference

### Most Used Commands
```bash
# Navigation
pwd                 # Print working directory
cd /path            # Change directory
ls -la              # List files (detailed)

# File Operations
cp src dest         # Copy
mv src dest         # Move/rename
rm file             # Remove
mkdir dir           # Create directory

# System Info
uname -a            # Kernel info
lsb_release -a      # Ubuntu version
df -h               # Disk space
free -h             # Memory usage

# Package Management (Ubuntu)
sudo apt update     # Update package lists
sudo apt upgrade    # Upgrade packages
sudo apt install X  # Install package

# Help
man command         # Manual page
command --help      # Quick help
```

---

## 📝 Notes Format

Each topic file follows this structure:
1. **Concept** - What it is
2. **Why** - Real-world importance  
3. **How** - Technical explanation
4. **Examples** - Practical commands
5. **Exercises** - Practice tasks
6. **Interview Q&A** - Common questions

---

## 🎯 Learning Path

**Week 1-2:** Topics 1-4 (Foundations)
**Week 3-4:** Topics 5-7 (System basics)
**Week 5-6:** Topics 8-10 (Networking & Scripting)
**Week 7-8:** Topics 11-12 (Advanced)

---
👩‍💻 Maintainer
Bhakti Khanvilkar
Linux Systems | Embedded Linux | Platform Engineering

📧 bhakti19khanvilkar@gmail.com

---
*Last Updated: February 2026*

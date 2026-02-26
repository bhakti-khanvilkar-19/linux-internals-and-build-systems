# Building Linux Images

## 📚 Overview

This section covers two major approaches to building custom Linux images for embedded and IoT devices:

1. **Yocto Project** - Industry-standard build system for custom embedded Linux
2. **Ubuntu Core & Snaps** - Canonical's approach using containerized, transactional updates

---

## 🗂️ Contents

| # | Topic | Description |
|---|-------|-------------|
| 1 | [Yocto Introduction](./01_Yocto_Introduction.md) | What is Yocto, OpenEmbedded, Poky, terminology |
| 2 | [Yocto Build Process](./02_Yocto_Build_Process.md) | Recipes, layers, BitBake, build workflow |
| 3 | [Ubuntu Core & Snaps](./03_Ubuntu_Core_Snaps.md) | Ubuntu Core, snap types, system snaps |
| 4 | [Building Snaps](./04_Building_Snaps.md) | Snapcraft, snapcraft.yaml, building & publishing |

---

## 📊 Diagrams

| Diagram | Description |
|---------|-------------|
| [Yocto Architecture](./diagrams/yocto_architecture.drawio) | Layers, recipes, BitBake workflow, output images |
| [Snap Architecture](./diagrams/snap_architecture.drawio) | Snap structure, system snaps, Ubuntu Core stack |
| [Build Comparison](./diagrams/build_comparison.drawio) | Yocto vs Ubuntu Core comparison |

---

## 🎯 When to Use What?

### Use Yocto When:
- Need complete control over every component
- Building for resource-constrained devices
- Require specific kernel/driver customizations
- Long-term product support (10+ years)
- Working with custom hardware

### Use Ubuntu Core When:
- Need transactional, atomic updates
- OTA (Over-The-Air) updates are critical
- Want application sandboxing
- Faster time-to-market
- IoT/Edge devices with network connectivity

---

## 🔗 Quick Comparison

| Aspect | Yocto | Ubuntu Core |
|--------|-------|-------------|
| **Build System** | BitBake | Snapcraft |
| **Package Format** | IPK/RPM/DEB | Snaps |
| **Update Method** | Manual/Mender/RAUC | Transactional (built-in) |
| **Image Size** | Highly customizable | Larger (snap overhead) |
| **Learning Curve** | Steep | Moderate |
| **Flexibility** | Maximum | Structured |
| **Security** | Manual hardening | AppArmor sandboxing |

---

*Last Updated: February 2026*

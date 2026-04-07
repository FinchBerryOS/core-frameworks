# FinchBerryOS Frameworks
<p align="center">
  <img src="https://raw.githubusercontent.com/FinchBerryOS/.github/refs/heads/main/profile/assets/Frameworks_github.png" alt="FinchBerryOS Logo" width="400">
</p>

This repository contains the high-level C-based frameworks for **FinchBerryOS**. These frameworks provide the primary APIs for applications (`.appd`) and system services (`.serviced`), abstracting the underlying Linux kernel and core daemons into a clean, macOS-inspired interface.

## 🏗 Architecture

The framework layer sits between the low-level `core-services` (daemons like `syscored`, `kmodsysd`, etc.) and the user-facing applications. Every framework is packaged as a **`.frameworkb`** (Framework Directory) and links against **`libfinch.so`** for system-wide XPC and C-library access.

---

## 📦 The Frameworks

#### GNUCore
**The Headless Foundation.** GNUCore acts as the essential bridge to the Linux ecosystem, forming the system's minimal "survival capsule." It contains the core libraries and binaries strictly required for the boot process, network management (via NetKit), and the hardware registry (via IOKit/kmodsysd). A system equipped only with this framework operates as a sovereign headless server.

* **Bundle Name:** `GNUCore.frameworkb`
* **Included Libraries:**
    * **Hardware & System:** `libudev.so`, `libkmod.so`, `libblkid.so`, `libuuid.so`
    * **Security & Utility:** `libssl.so`, `libcrypto.so`, `libz.so`, `libexpat.so`, `libffi.so`
* **Internal Helpers (CLI Tools):**
    * Disk & Partitioning: `sgdisk`, `growpart`, `fdisk`, `lsblk`, `wipefs`
    * Filesystem: `e2fsck`, `resize2fs`, `tune2fs`, `mkfs.ext4`, `mkfs.vfat`, `dosfsck`
    * Kernel & Hardware: `kmod`, `udevadm`, `hwclock`

#### GNUCoreExtensions
**The Multimedia & Interaction Layer.**

* **Bundle Name:** `GNUCoreExtensions.frameworkb`
* **Dependency:** Requires `GNUCore.frameworkb`
* **Included Libraries:**
    * Graphics: `libdrm.so`, `libwayland-*`, `libgbm.so`, `libEGL.so`
    * Audio/Input: `libasound.so`, `libinput.so`, `libxkbcommon.so`

### ConfigKit
* **Bundle Name:** `ConfigKit.frameworkb`
* System registry & configuration
* Change notifications
* Backend: `configd`

### StorageKit
* **Bundle Name:** `StorageKit.frameworkb`
* Volume, partition, fs management
* Scope: no file-level API
* Backend: GNUCore helpers

### SecurityKit
* **Bundle Name:** `SecurityKit.frameworkb`
* Keychain, TLS, TPM
* Access control & policy
* Backend: `securityd`

### IdentityKit
* **Bundle Name:** `IdentityKit.frameworkb`
* Users, groups, authentication
* Session lifecycle
* Backend: `identityd`, `authd`

### ContainerKit
* **Bundle Name:** `ContainerKit.frameworkb`
* Namespaces, cgroups, isolation
* Execution inside containers
* Backend: `syscored`

### IOKit
* **Bundle Name:** `IOKit.frameworkb`
* Hardware registry & events
* Backend: `kmodsysd`

### CoreSystem
* **Bundle Name:** `CoreSystem.frameworkb`
* Logging, lifecycle, errors
* Process management
* Time & scheduling
* Bundles (.appb/.frameworkb/.pluginb)
* Backend: `logd`, `syscored`

### NetKit
* **Bundle Name:** `NetKit.frameworkb`
* Connections, streams, sessions
* HTTP/1–3, QUIC
* libp2p
* Unified VPN (WireGuard/OpenVPN/IPsec)
* DNS, mDNS
* Backend: `networkd`, `dnsd`

### BitcoinEngine
* **Bundle Name:** `BitcoinEngine.frameworkb`
* Bitcoin protocol & sync
* Backend: `bitcoind` or native service

### PythonKit
* **Bundle Name:** `PythonKit.frameworkb`
* Python bindings for system frameworks

### NodeKit
* **Bundle Name:** `NodeKit.frameworkb`
* Node.js bindings

### JavaKit
* **Bundle Name:** `JavaKit.frameworkb`
* JVM bindings

### CoreGraphics
* **Bundle Name:** `CoreGraphics.frameworkb`

### Collaboration
* **Bundle Name:** `Collaboration.frameworkb`

---

## 📁 Framework Bundle Structure

Name.frameworkb/
├── Name
├── Headers/
├── Resources/
└── Info.json

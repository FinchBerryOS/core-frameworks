# FinchBerryOS Frameworks
[English](README.md) | [Deutsch](README.de.md)

This repository contains the high-level C-based frameworks for **FinchBerryOS**. These frameworks provide the primary APIs for applications (`.appd`) and system services (`.serviced`), abstracting the underlying Linux kernel and core daemons into a clean, macOS-inspired interface.

## 🏗 Architecture

The framework layer sits between the low-level `core-services` (daemons like `syscored`, `kmodsysd`, etc.) and the user-facing applications. Every framework is packaged as a **`.frameworkb`** (Framework Directory) and links against **`libfinch.so`** for system-wide XPC and C-library access.

---

## 📦 The Frameworks

#### GNUCore
**The Headless Foundation.** GNUCore acts as the essential bridge to the Linux ecosystem, forming the system's minimal "survival capsule." It contains the core libraries and binaries strictly required for the boot process, network management (via NetKit), and the hardware registry (via IOKit/kmodsysd). A system equipped only with this framework operates as a sovereign headless server.

* **Bundle Name:** `GNUCore.frameworkb`
* **Included Libraries:**
    * **Core Runtime:** `libc.so`, `libm.so`, `libdl.so`, `libpthread.so`
    * **Hardware & System:** `libudev.so`, `libkmod.so`, `libblkid.so`, `libuuid.so`
    * **Security & Utility:** `libssl.so`, `libcrypto.so`, `libz.so`, `libexpat.so`, `libffi.so`
* **Internal Helpers (CLI Tools):** These binaries are stored within `Helpers/` and are isolated from the global `$PATH`. They are invoked via `libfinch` or system daemons for low-level tasks:
    * **Disk & Partitioning:** `sgdisk`, `growpart`, `fdisk`, `lsblk`, `wipefs`
    * **Filesystem Management:** `e2fsck`, `resize2fs`, `tune2fs`, `mkfs.ext4`, `mkfs.vfat`, `dosfsck`
    * **Kernel & Hardware:** `kmod` (modprobe/insmod), `udevadm`, `hwclock`

#### GNUCoreExtensions
**The Multimedia & Interaction Layer.** This framework builds directly upon `GNUCore` and extends the system with graphics, audio, and input capabilities. It is required as soon as a graphical user interface (WindowServer/CoreGraphics) or media output is active. This separation ensures that the attack surface on headless systems remains minimal.

* **Bundle Name:** `GNUCoreExtensions.frameworkb`
* **Dependency:** Requires `GNUCore.frameworkb` to be installed.
* **Included Libraries:**
    * **Graphics & Display:** `libdrm.so`, `libwayland-client.so`, `libwayland-server.so`, `libgbm.so`, `libpixman-1.so`, `libEGL.so`, `libGLESv2.so`
    * **Audio & Input:** `libasound.so`, `libinput.so`, `libxkbcommon.so`

### Foundation
**The Logical Base.** Centralizes system state and configuration. It is the "Single Source of Truth" for hostnames, service settings, and global parameters.
* **Bundle Name:** `Foundation.frameworkb`
* **Functionality:** System registry access, key-value configuration, service state management.
* **Backend:** Interfaces with `configd`.

### StorageKit
**Volume Management.** Abstrahierst physical drives into logical units. Manages mount points, partition tables, and the integrity of A/B system images.
* **Bundle Name:** `StorageKit.frameworkb`
* **Functionality:** Mounting/Unmounting, partitioning, filesystem integrity (fsck).
* **Backend:** Utilizes `GNUCore` helpers (fdisk, mkfs).

### SecurityKit
**Identity & Encryption.** Manages cryptographic secrets and identities.
* **Bundle Name:** `SecurityKit.frameworkb`
* **Functionality:** System-wide Keychain, SSL/TLS certificate management, TPM interface.
* **Backend:** Interfaces with `securityd`.

### IdentityKit
**User Identity & Authentication.**
Manages system access, user profiles, and permission levels. It serves as the central authority for the login process and session management.
* **Bundle Name:** `IdentityKit.frameworkb`
* **Functionality:** User login (authentication), management of user groups, namespace assignment, and session control.
* **Backend:** Interfaces with `identityd` and `authd`.

### IOKit
**The Hardware Registry.** Provides an object-oriented view of the system's hardware.
* **Bundle Name:** `IOKit.frameworkb`
* **Backend:** Communicates via XPC with `kmodsysd`.

### CoreSystem
**The OS Foundation.** Provides core primitives used by almost every binary.
* **Bundle Name:** `CoreSystem.frameworkb`
* **Functionality:** Unified Logging (`cs_log`), memory helpers, notifications.

### NetKit
**Networking & Decentralization.** High-performance stack for standard and sovereign connectivity (WireGuard, Bitcoin P2P, HTTP/3).
* **Bundle Name:** `NetKit.frameworkb`

### CoreGraphics
**The Rendering Engine.** Primary 2D drawing API for FinchBerryOS.
* **Bundle Name:** `CoreGraphics.frameworkb`

### Collaboration
**Data Exchange.** Handles data sharing and user identity management (Pasteboard, Share Sheets).
* **Bundle Name:** `Collaboration.frameworkb`

---

## 📁 Framework Bundle Structure (.frameworkb)

```text
Name.frameworkb/
├── Name                 # Shared Object / Umbrella Library
├── Headers/             # C header files
├── Resources/           # Localization, assets, and icons
└── Info.json            # Metadata, versioning, and dependencies
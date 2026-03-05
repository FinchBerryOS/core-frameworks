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
**The Hardware Registry.**
Inspired by Darwin’s I/O Kit, this framework provides an object-oriented, hierarchical view of the system's hardware. It abstracts the complexity of kernel events and driver states into a stable API for applications and system services.

* **Bundle Name:** `IOKit.frameworkb`
* **Key Functionalities:**
    * **I/O Registry:** A dynamic tree structure of all detected hardware components (CPU, PCI, USB, NVMe). Enables precise device matching via classes and properties.
    * **Hardware Events:** An asynchronous notification system for hot-plugging events (e.g., "Monitor connected," "USB drive removed").
    * **Power Management:** Centralized control of power states (Sleep, Wake, Idle) for individual hardware groups.
    * **Property Tables:** Direct access to hardware metadata such as serial numbers, revisions, and supported features (e.g., display resolutions via EDID).
* **Backend:** Interfaces via XPC with the hardware daemon `kmodsysd` and utilizes `libudev` (via GNUCore) to monitor kernel states.

### CoreSystem
**The OS Foundation.**
CoreSystem defines the fundamental programming models and primitive data types for FinchBerryOS. It acts as the bridge between the raw C world of `GNUCore` and the higher-level object-oriented framework architecture. Every binary in the system links against CoreSystem to ensure consistent behavior across the OS.

* **Bundle Name:** `CoreSystem.frameworkb`
* **Key Functionalities:**
    * **Unified Logging:** `cs_log` provides a high-performance, structured logging system with granular log levels (Fault, Error, Info, Debug).
    * **Object Lifecycle:** Implements reference counting and memory management primitives for framework objects.
    * **Notification Center:** A system-wide observer pattern for inter-process events and state changes.
    * **Dispatch Queues:** High-level abstractions for asynchronous threading and task management.
* **Backend:** Interfaces with `logd` for persistent log storage and `syscored` for process metrics and health monitoring.

### NetKit
**Networking & Decentralization.**
A high-performance networking stack providing a unified API for standard communication and sovereign, decentralized connectivity. NetKit abstracts the complexity of the Linux kernel into an object-oriented interface for modern web and P2P protocols.

* **Bundle Name:** `NetKit.frameworkb`
* **Key Features:**
    * **Modern IP Stack:** Full abstraction of IPv4/IPv6 addressing, TCP/UDP streams, and native support for **QUIC** (UDP-based).
    * **High-Level HTTP Engine:** Native support for **HTTP/1.1, HTTP/2, and HTTP/3**. Includes an intelligent connection pooler and automatic response caching.
    * **Sovereign Connectivity:** Native integration of the **Bitcoin P2P protocol**. Enables handshakes, peer discovery, and Merkle block filtering directly at the framework level (SPV-ready).
    * **Secure VPN (WireGuard):** A dedicated API to control and monitor native Linux kernel WireGuard interfaces (key exchange, peer management).
    * **DNS Ecosystem:** Integrated stub resolver with support for **DNS-over-HTTPS (DoH)** and **DNS-over-TLS (DoT)** to bypass censorship and tracking.
    * **Service Discovery:** Native implementation of **mDNS (Bonjour)** and DNS-SD for automatic device and service discovery in local networks.
* **Backend:** Interfaces via XPC with `networkd` (connectivity), `dnsd` (caching/privacy), and directly with the Linux kernel for WireGuard operations.

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
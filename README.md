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

### ConfigKit
**The Logical Base.** Centralizes system state and configuration. It is the "Single Source of Truth" for hostnames, service settings, and global parameters.
* **Bundle Name:** `ConfigKit.frameworkb`
* **Functionality:** System registry access, key-value configuration, service state management.
* **Additional Features:** Change notification system for observing configuration updates.
* **Backend:** Interfaces with `configd`.

### StorageKit
**Volume Management.** Abstrahierst physical drives into logical units. Manages mount points, partition tables, and the integrity of A/B system images.
* **Bundle Name:** `StorageKit.frameworkb`
* **Functionality:** Mounting/Unmounting, partitioning, filesystem integrity (fsck).
* **Scope:** Focused on volumes and filesystems, not file-level operations (handled by libc/POSIX).
* **Backend:** Utilizes `GNUCore` helpers (fdisk, mkfs).

### SecurityKit
**Identity & Encryption.** Manages cryptographic secrets and identities.
* **Bundle Name:** `SecurityKit.frameworkb`
* **Functionality:** System-wide Keychain, SSL/TLS certificate management, TPM interface.
* **Additional Features:** Access control and trust policy evaluation for secure system interactions.
* **Backend:** Interfaces with `securityd`.

### IdentityKit
**User Identity & Authentication.**
Manages system access, user profiles, and permission levels. It serves as the central authority for the login process and session management.
* **Bundle Name:** `IdentityKit.frameworkb`
* **Functionality:** User login (authentication), management of user groups, namespace assignment.
* **Additional Features:** Session lifecycle management across the system.
* **Backend:** Interfaces with `identityd` and `authd`.

### ContainerKit
**Process Isolation & Resource Management.**
ContainerKit is the primary framework for creating and managing isolated execution environments. It abstracts the complexity of Linux namespaces and control groups (cgroups) to securely separate processes, filesystems, and network stacks from each other and the host system.

* **Bundle Name:** `ContainerKit.frameworkb`
* **Key Features:**
    * **Namespace Abstraction:** Simplified creation of isolated environments for mounts, process IDs (PID), networks, and user IDs (User).
    * **Resource Quotas:** Precise control over CPU cycles, memory limits, and I/O priorities via cgroups.
    * **Filesystem Jailing:** Integration of overlay mounts and read-only layers for volatile or protected container environments.
    * **Network Virtualization:** Creation of virtual interfaces (veth) for isolated network stacks within a container.
    * **Execution Context:** Launch and manage processes inside isolated container environments.
* **Backend:** Interfaces via LXPC with `syscored` to handle kernel-level configuration for isolated process contexts.

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
    * **Error Handling:** Standardized error representation and propagation across all frameworks.
    * **Process Management:** Creation, execution, and lifecycle control of system processes.
    * **Time & Scheduling:** System time access, timers, and scheduling primitives.
    * **Notification Center:** A system-wide observer pattern for inter-process events and state changes.
    * **Dispatch Queues:** High-level abstractions for asynchronous threading and task management.
    * **Bundle Model:** Canonical runtime representation of `.appb`, `.frameworkb`, and `.pluginb` bundles.
    * **Resource Resolution:** Lookup of bundle-local resources, manifests, embedded frameworks, and helper binaries.
    * **Code Location:** Resolution of primary binaries, framework entry points, and plugin executables.
    * **Helper Integration:** Discovery and controlled invocation of bundle-local helpers.
* **Backend:** Interfaces with `logd` for persistent log storage and `syscored` for process metrics and health monitoring.

### NetKit
**Networking & Decentralization.**
A high-performance networking stack providing a unified API for standard communication and sovereign, decentralized connectivity. NetKit abstracts the complexity of the Linux kernel into an object-oriented interface for modern web and P2P protocols.

* **Bundle Name:** `NetKit.frameworkb`
* **Key Features:**
    * **Modern IP Stack:** Abstraction of IPv4/IPv6, TCP/UDP streams, and native support for **QUIC** (UDP-based).
    * **Connection Model:** Unified abstraction for connections, listeners, streams, and sessions across all supported protocols.
    * **High-Level HTTP Engine:** Native support for **HTTP/1.1, HTTP/2, and HTTP/3** (via QUIC).
    * **P2P Foundations (libp2p):** Native support for peer identities, secure channels, peer discovery, and multiplexed stream-based communication for decentralized protocols.
    * **Unified VPN Platform:** Provides a backend-agnostic API for establishing and monitoring secure VPN tunnels using **WireGuard**, **OpenVPN**, or **IPsec**. Supports both **system-wide routing** and **isolated container-backed VPN contexts** with standardized proxy endpoints for application traffic.
    * **DNS Ecosystem:** Integrated stub resolver with support for **DNS-over-HTTPS (DoH)** and **DNS-over-TLS (DoT)**.
    * **Service Discovery:** Native implementation of **mDNS (Bonjour)** and DNS-SD.
* **Backend:** Interfaces via LXPC with `networkd` (connectivity) and `dnsd` (privacy/caching).

### BitcoinEngine
**Sovereign Bitcoin Node Engine.**
Provides the native Bitcoin protocol and engine layer for FinchBerryOS. BitcoinEngine builds on NetKit’s transport and decentralized networking foundations to manage peer sessions, block and transaction exchange, and node synchronization.

* **Bundle Name:** `BitcoinEngine.frameworkb`
* **Key Features:**
    * **Bitcoin P2P Protocol:** Native implementation of version handshakes, peer messaging, and inventory exchange.
    * **Chain Synchronization:** Download and processing of headers, blocks, and transaction announcements.
    * **Peer Management:** Maintains Bitcoin peer sessions and policy-aware connection behavior on top of NetKit.
* **Backend:** Interfaces with `bitcoind` or a native FinchBerryOS Bitcoin service for validation, storage, and policy control.

### PythonKit
**Native Python Runtime Integration.**
Provides the default FinchBerryOS system API bridge for Python. PythonKit exposes the system frameworks through a native Python interface and is loaded by the system Python runtime to provide first-class access to FinchBerryOS services.

* **Bundle Name:** `PythonKit.frameworkb`
* **Functionality:** Native Python bindings for `CoreSystem`, `ConfigKit`, `NetKit`, `StorageKit`, `SecurityKit`, `IdentityKit`, and `IOKit`.
* **Backend:** Bridges the Python runtime to FinchBerryOS frameworks through the native C framework layer.

### NodeKit
**Native Node.js Runtime Integration.**
Provides the default FinchBerryOS system API bridge for Node.js. NodeKit exposes the system frameworks through native Node.js modules and gives JavaScript applications direct access to FinchBerryOS services.

* **Bundle Name:** `NodeKit.frameworkb`
* **Functionality:** Native Node.js bindings for `CoreSystem`, `ConfigKit`, `NetKit`, `StorageKit`, `SecurityKit`, `IdentityKit`, and `IOKit`.
* **Backend:** Bridges the Node.js runtime to FinchBerryOS frameworks through the native C framework layer.

### JavaKit
**Native JVM Runtime Integration.**
Provides the default FinchBerryOS system API bridge for JVM languages. JavaKit exposes the system frameworks to Java and Kotlin applications through a native integration layer for first-class OS access.

* **Bundle Name:** `JavaKit.frameworkb`
* **Functionality:** Native JVM bindings for `CoreSystem`, `ConfigKit`, `NetKit`, `StorageKit`, `SecurityKit`, `IdentityKit`, and `IOKit`.
* **Backend:** Bridges the JVM runtime to FinchBerryOS frameworks through the native C framework layer.

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
```
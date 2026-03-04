# FinchBerryOS Frameworks

This repository contains the high-level C-based frameworks for **FinchBerryOS**. These frameworks provide the primary APIs for applications (`.appd`) and system services (`.serviced`), abstracting the underlying Linux kernel and core daemons into a clean, macOS-inspired interface.

## 🏗 Architecture

The framework layer sits between the low-level `core-services` (daemons like `syscored`, `kmodsysd`, etc.) and the user-facing applications. Every framework is packaged as a **`.frameworkd`** (Framework Bundle) and links against **`libfinch.so`** for system-wide XPC and C-library access.

---

## 📦 The Frameworks

### GNUCore
**The Vendor Layer.**
GNUCore acts as the bridge to the Linux world. It collects and re-exports essential GNU/Linux libraries needed by other frameworks, ensuring binary stability across the system.
* **Bundle Name:** `GNUCore.frameworkd`
* **Included Libraries:**
    * **Hardware & System:** `libudev.so`, `libkmod.so`, `libblkid.so`, `libuuid.so`
    * **Graphics & Display:** `libdrm.so`, `libwayland-client.so`, `libwayland-server.so`, `libgbm.so`, `libpixman-1.so`, `libEGL.so`, `libGLESv2.so`
    * **Audio & Input:** `libasound.so`, `libinput.so`, `libxkbcommon.so`
    * **Utility:** `libffi.so`, `libexpat.so`, `libz.so`
* **Purpose:** Provides a centralized, stable vendor-layer for all low-level Linux dependencies.

### IOKit
**The Hardware Registry.**
Inspired by Darwin’s I/O Kit, this framework provides an object-oriented view of the system's hardware.
* **Bundle Name:** `IOKit.frameworkd`
* **Functionality:** Accessing the I/O Registry, matching devices (USB, PCI), and managing power states.
* **Backend:** Communicates via XPC with `kmodsysd` to retrieve udev-parsed hardware data.

### CoreSystem
**The OS Foundation.**
Provides core primitives and essential system utilities used by almost every binary.
* **Bundle Name:** `CoreSystem.frameworkd`
* **Functionality:** Unified Logging (`cs_log`), memory management helpers, and system-wide notification observers.
* **Backend:** Interfaces with `logd` and `configd` to provide a real-time view of the system state.

### Collaboration
**Inter-App Data Exchange.**
Handles data sharing and user identity management between applications.
* **Bundle Name:** `Collaboration.frameworkd`
* **Functionality:** Global pasteboard (clipboard), "Share Sheet" logic, and cross-application collaboration tools.
* **Backend:** Works in tandem with the `sharingd` daemon.

### NetKit
**Networking & Decentralization.**
A high-performance networking stack that goes beyond standard sockets.
* **Bundle Name:** `NetKit.frameworkd`
* **Functionality:** WireGuard tunnel management, DNS-over-HTTPS, and native support for the Bitcoin P2P protocol.
* **Backend:** Leverages `networkd` and `configd` for IP and connectivity management.

### CoreGraphics
**The Rendering Engine.**
The primary 2D drawing API for FinchBerryOS.
* **Bundle Name:** `CoreGraphics.frameworkd`
* **Functionality:** Path-based drawing, anti-aliased text rendering, and affine transformations.
* **Backend:** Uses `libdrm` and `pixman` (via GNUCore) to render into shared memory buffers managed by the WindowServer.

---

## 📁 Framework Bundle Structure (.frameworkd)

Each framework follows a strict structure to support versioning and modularity:

```text
Name.frameworkd/
├── Name                 # Shared Object / Umbrella Library
├── Headers/             # C header files
├── Resources/           # Localization, assets, and icons
└── Info.json            # Metadata, versioning, and dependencies
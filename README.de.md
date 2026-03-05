# FinchBerryOS Frameworks
[English](README.md) | [Deutsch](README.de.md)

Dieses Repository enthält die High-Level C-basierten Frameworks für **FinchBerryOS**. Diese Frameworks bieten die primären APIs für Anwendungen (`.appd`) und Systemdienste (`.serviced`) und abstrahieren den zugrunde liegenden Linux-Kernel sowie die Core-Daemons in eine saubere, von macOS inspirierte Schnittstelle.

## 🏗 Architektur

Die Framework-Schicht sitzt zwischen den Low-Level `core-services` (Daemons wie `syscored`, `kmodsysd`, etc.) und den benutzerorientierten Anwendungen. Jedes Framework ist als **`.frameworkb`** (Framework-Verzeichnis) paketiert und linkt gegen **`libfinch.so`** für systemweiten XPC- und C-Library-Zugriff.

---

## 📦 Die Frameworks: Vendor & Utility Layer

FinchBerryOS nutzt eine modulare Architektur, um die Linux-Kompatibilität sicherzustellen. Die Hardware-Abstraktion und System-Basis sind in zwei spezialisierte Layer unterteilt, um maximale Sicherheit und Effizienz (Headless vs. Desktop) zu gewährleisten.

#### GNUCore
**Die Headless-Basis.** GNUCore fungiert als essentielle Brücke zum Linux-Ökosystem und bildet die minimale "Überlebenskapsel" des Systems. Sie enthält die Kern-Bibliotheken und Binaries, die für den Boot-Vorgang, das Netzwerk-Management (via NetKit) und die Hardware-Registry (via IOKit/kmodsysd) zwingend erforderlich sind. Ein System, das nur mit diesem Framework ausgestattet ist, operiert als souveräner Headless-Server.

* **Bundle-Name:** `GNUCore.frameworkb`
* **Enthaltene Bibliotheken:**
    * **Core Runtime:** `libc.so`, `libm.so`, `libdl.so`, `libpthread.so`
    * **Hardware & System:** `libudev.so`, `libkmod.so`, `libblkid.so`, `libuuid.so`
    * **Sicherheit & Utilities:** `libssl.so`, `libcrypto.so`, `libz.so`, `libexpat.so`, `libffi.so`
* **Interne Helfer (CLI-Tools):** Diese Binaries werden in `Helpers/` gespeichert und sind vom globalen `$PATH` isoliert. Sie werden über `libfinch` oder System-Daemons für Low-Level-Aufgaben aufgerufen:
    * **Disk & Partitionierung:** `sgdisk`, `growpart`, `fdisk`, `lsblk`, `wipefs`
    * **Dateisystem-Verwaltung:** `e2fsck`, `resize2fs`, `tune2fs`, `mkfs.ext4`, `mkfs.vfat`, `dosfsck`
    * **Kernel & Hardware:** `kmod` (modprobe/insmod), `udevadm`, `hwclock`

#### GNUCoreExtensions
**Der Multimedia & Interaktions-Layer.** Dieses Framework baut direkt auf `GNUCore` auf und erweitert das System um Grafik-, Audio- und Eingabefähigkeiten. Es wird benötigt, sobald eine grafische Benutzeroberfläche (WindowServer/CoreGraphics) oder eine Medienausgabe aktiv ist. Diese Trennung stellt sicher, dass die Angriffsfläche auf Headless-Systemen minimal bleibt.

* **Bundle-Name:** `GNUCoreExtensions.frameworkb`
* **Abhängigkeit:** Erfordert ein installiertes `GNUCore.frameworkb`.
* **Enthaltene Bibliotheken:**
    * **Grafik & Display:** `libdrm.so`, `libwayland-client.so`, `libwayland-server.so`, `libgbm.so`, `libpixman-1.so`, `libEGL.so`, `libGLESv2.so`
    * **Audio & Eingabe:** `libasound.so`, `libinput.so`, `libxkbcommon.so`

---

### IOKit
**Die Hardware-Registrierung.**
Inspiriert von Darwins I/O Kit bietet dieses Framework eine objektorientierte Sicht auf die Hardware des Systems.
* **Bundle-Name:** `IOKit.frameworkb`
* **Funktionalität:** Zugriff auf die I/O-Registry, Matching von Geräten (USB, PCI) und Verwaltung von Energiezuständen.
* **Backend:** Kommuniziert via XPC mit `kmodsysd`.

### CoreSystem
**Das OS-Fundament.**
Bietet Kern-Primitiven und essentielle System-Utilities, die von fast jeder Binary verwendet werden.
* **Bundle-Name:** `CoreSystem.frameworkb`
* **Funktionalität:** Einheitliche Protokollierung (`cs_log`), Helfer für die Speicherverwaltung und systemweite Benachrichtigungs-Observer.
* **Backend:** Schnittstelle zu `logd` und `configd`.

### Collaboration
**Anwendungsübergreifender Datenaustausch.**
Verwaltet die gemeinsame Datennutzung und Benutzeridentitäten zwischen Anwendungen.
* **Bundle-Name:** `Collaboration.frameworkb`
* **Funktionalität:** Globale Zwischenablage (Pasteboard), "Share Sheet"-Logik und Tools für die Zusammenarbeit zwischen Anwendungen.
* **Backend:** Arbeitet zusammen mit dem `sharingd`-Daemon.

### NetKit
**Networking & Dezentralisierung.**
Ein leistungsstarker Netzwerk-Stack, der eine einheitliche API für Standard-, fortgeschrittene und souveräne Konnektivität bietet.
* **Bundle-Name:** `NetKit.frameworkb`
* **Unterstützte Protokolle & Features:**
    * **Standard-Stack:** IPv4, IPv6, TCP, UDP, **QUIC**, ICMP.
    * **HTTP-Dienste:** High-Level **HTTP-Client-API** mit nativer Unterstützung für HTTP/1.1, HTTP/2 und **HTTP/3 (via QUIC)**.
    * **DNS-Dienste:** High-Level **Client-DNS-API** (Stub-Resolver) für A-, AAAA-, SRV- und TXT-Abfragen. Unterstützt moderne verschlüsselte Standards (DoH/DoT).
    * **WireGuard API:** Control-Plane-API zur Konfiguration und Verwaltung nativer Linux-Kernel-WireGuard-Schnittstellen.
    * **Dezentralisierung (Souveränität):** Native Unterstützung für das **Bitcoin-P2P-Protokoll** (Handshakes, Peer-Discovery, Merkle-Block-Filtering).
    * **Diensterkennung:** mDNS (Bonjour-kompatibel) und DNS-SD.
* **Backend:** Schnittstelle zu `networkd` (Konnektivität), `configd` (Status) und **`dnsd`** (zentraler rekursiver/caching Resolver-Dienst).

### CoreGraphics
**Die Rendering-Engine.**
Die primäre 2D-Zeichnungs-API für FinchBerryOS.
* **Bundle-Name:** `CoreGraphics.frameworkb`
* **Funktionalität:** Pfadbasiertes Zeichnen, geglättete Textdarstellung (Anti-Aliasing) und affine Transformationen.
* **Backend:** Nutzt `libdrm` und `pixman` (via GNUCoreExtensions), um in Shared-Memory-Buffer zu rendern, die vom WindowServer verwaltet werden.

---

## 📁 Framework-Bundle-Struktur (.frameworkb)

Jedes Framework folgt einer strengen Struktur, um Versionierung und Modularität zu unterstützen:

```text
Name.frameworkb/
├── Name                 # Shared Object / Umbrella Library
├── Headers/             # C-Header-Dateien
├── Resources/           # Lokalisierung, Assets und Icons
└── Info.json            # Metadaten, Versionierung und Abhängigkeiten
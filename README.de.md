# FinchBerryOS Frameworks
[English](README.md) | [Deutsch](README.de.md)

Dieses Repository enthält die High-Level C-basierten Frameworks für **FinchBerryOS**. Diese Frameworks bieten die primären APIs für Anwendungen (`.appd`) und Systemdienste (`.serviced`) und abstrahieren den zugrunde liegenden Linux-Kernel sowie die Core-Daemons in eine saubere, von macOS inspirierte Schnittstelle.

## 🏗 Architektur

Die Framework-Schicht sitzt zwischen den Low-Level `core-services` (Daemons wie `syscored`, `kmodsysd`, etc.) und den benutzerorientierten Anwendungen. Jedes Framework ist als **`.frameworkb`** (Framework-Verzeichnis) paketiert und linkt gegen **`libfinch.so`** für systemweiten XPC- und C-Library-Zugriff.

---

## 📦 Die Frameworks

#### GNUCore
**Die Headless-Basis.** GNUCore fungiert als essentielle Brücke zum Linux-Ökosystem und bildet die minimale "Überlebenskapsel" des Systems. Sie enthält die Kern-Bibliotheken und Binaries, die für den Boot-Vorgang, das Netzwerk-Management (via NetKit) und die Hardware-Registry (via IOKit/kmodsysd) zwingend erforderlich sind. Ein System mit nur diesem Framework operiert als souveräner Headless-Server.

* **Bundle-Name:** `GNUCore.frameworkb`
* **Enthaltene Bibliotheken:**
    * **Core Runtime:** `libc.so`, `libm.so`, `libdl.so`, `libpthread.so`
    * **Hardware & System:** `libudev.so`, `libkmod.so`, `libblkid.so`, `libuuid.so`
    * **Sicherheit & Utilities:** `libssl.so`, `libcrypto.so`, `libz.so`, `libexpat.so`, `libffi.so`
* **Interne Helfer (CLI-Tools):** Diese Binaries werden in `Helpers/` gespeichert und sind vom globalen `$PATH` isoliert. Sie werden über `libfinch` oder System-Daemons aufgerufen:
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

### Foundation
**Das logische Fundament.** Zentralisiert den Systemstatus und die Konfiguration. Es ist die "Single Source of Truth" für Hostnamen, Dienst-Einstellungen und globale Parameter.
* **Bundle-Name:** `Foundation.frameworkb`
* **Funktionalität:** Zugriff auf die System-Registry, Key-Value-Konfiguration, Dienst-Status.
* **Backend:** Kommuniziert mit `configd`.

### StorageKit
**Die Volumen-Verwaltung.** Abstrahiert physische Datenträger in logische Einheiten. Verwaltet Mount-Points, Partitionstabellen und die Integrität der A/B-System-Images.
* **Bundle-Name:** `StorageKit.frameworkb`
* **Funktionalität:** Mounten/Unmounten, Partitionierung, Dateisystem-Checks (fsck).
* **Backend:** Nutzt `GNUCore` Helfer (fdisk, mkfs).

### SecurityKit
**Identität & Verschlüsselung.** Verwaltet kryptografische Geheimnisse und Identitäten.
* **Bundle-Name:** `SecurityKit.frameworkb`
* **Funktionalität:** System-Schlüsselbund (Keychain), SSL/TLS-Zertifikatsverwaltung, TPM-Schnittstelle.
* **Backend:** Schnittstelle zu `securityd`.

### IdentityKit
**Benutzeridentität & Authentifizierung.**
Regelt den Zugriff auf das System und verwaltet Benutzerprofile sowie Berechtigungsstufen. Es ist die zentrale Instanz für den Login-Prozess und die Sitzungsverwaltung.
* **Bundle-Name:** `IdentityKit.frameworkb`
* **Funktionalität:** Benutzer-Login (Authentifizierung), Verwaltung von Benutzergruppen, Namespace-Zuweisung und Sitzungskontrolle.
* **Backend:** Schnittstelle zu `identityd` und `authd`.

### IOKit
**Die Hardware-Registry.**
Inspiriert von Darwins I/O Kit, bietet dieses Framework eine objektorientierte, hierarchische Sicht auf die gesamte Hardware des Systems. Es abstrahiert die Komplexität von Kernel-Events und Treiber-Zuständen in eine stabile API für Anwendungen und Systemdienste.

* **Bundle-Name:** `IOKit.frameworkb`
* **Kern-Funktionalitäten:**
    * **I/O Registry:** Eine dynamische Baumstruktur aller erkannten Hardware-Komponenten (CPU, PCI, USB, NVMe). Ermöglicht präzises Device-Matching über Klassen und Eigenschaften.
    * **Hardware-Events:** Ein asynchrones Benachrichtigungssystem für Hot-Plugging (z. B. "Monitor angeschlossen", "USB-Stick entfernt").
    * **Power Management:** Zentrale Steuerung von Energiezuständen (Sleep, Wake, Idle) für einzelne Hardware-Gruppen.
    * **Property Tables:** Direkter Zugriff auf Hardware-Metadaten wie Seriennummern, Revisionen und unterstützte Features (z. B. Display-Auflösungen via EDID).
* **Backend:** Kommuniziert via XPC mit dem Hardware-Daemon `kmodsysd` und nutzt `libudev` (via GNUCore) zur Überwachung des Kernels.

### CoreSystem
**Das Betriebssystem-Fundament.**
CoreSystem definiert die grundlegenden Programmiermodelle und primitiven Datentypen für FinchBerryOS. Es ist das Bindeglied zwischen der reinen C-Welt der `GNUCore` und der objektorientierten Framework-Architektur. Jede Binary im System linkt gegen CoreSystem, um einheitliche Verhaltensweisen sicherzustellen.

* **Bundle-Name:** `CoreSystem.frameworkb`
* **Zentrale Funktionen:**
    * **Unified Logging:** `cs_log` bietet ein hochperformantes, strukturiertes Logging-System mit einstellbaren Log-Leveln (Fault, Error, Info, Debug).
    * **Objekt-Lebenszyklus:** Implementiert Referenzzählung (Reference Counting) und Speicherverwaltungs-Primitives für Framework-Objekte.
    * **Notification Center:** Ein systemweites Beobachter-Modell (Observer Pattern) für inter-prozessuale Ereignisse.
    * **Dispatch Queues:** Abstraktion für asynchrones Threading und Task-Management.
* **Backend:** Kommuniziert mit `logd` für die persistente Speicherung und `syscored` für Prozess-Metriken.

### NetKit
**Networking & Dezentralisierung.**
Ein hochperformanter Netzwerk-Stack, der eine einheitliche API für Standard-Kommunikation sowie souveräne, dezentrale Konnektivität bietet. NetKit abstrahiert die Komplexität des Linux-Kernels in eine objektorientierte Schnittstelle für moderne Web- und P2P-Protokolle.

* **Bundle-Name:** `NetKit.frameworkb`
* **Kern-Features:**
    * **Moderner IP-Stack:** Vollständige Abstraktion von IPv4/IPv6-Adressierung, TCP/UDP-Streams und nativer Support für **QUIC** (UDP-basiert).
    * **High-Level HTTP Engine:** Unterstützung für **HTTP/1.1, HTTP/2 und HTTP/3**. Beinhaltet einen intelligenten Connection-Pooler und automatisches Caching.
    * **Souveräne Konnektivität:** Native Integration des **Bitcoin P2P-Protokolls**. Ermöglicht Handshakes, Peer-Discovery und Merkle-Block-Filtering direkt auf Framework-Ebene (SPV-fähig).
    * **Sicheres VPN (WireGuard):** Eine dedizierte API zur Steuerung und Überwachung von nativen WireGuard-Schnittstellen im Kernel (Schlüsselaustausch, Peer-Management).
    * **DNS-Ökosystem:** Integrierter Stub-Resolver mit Support für **DNS-over-HTTPS (DoH)** und **DNS-over-TLS (DoT)** zur Umgehung von Zensur und Tracking.
    * **Service Discovery:** Native Implementierung von **mDNS (Bonjour)** und DNS-SD zur automatischen Erkennung von Geräten und Diensten im lokalen Netzwerk.
* **Backend:** Kommuniziert über XPC mit `networkd` (Konnektivität), `dnsd` (Caching/Privacy) und direkt mit dem Linux-Kernel für WireGuard-Operationen.

### CoreGraphics
**Die Rendering-Engine.** Primäre 2D-Zeichnungs-API für FinchBerryOS.
* **Bundle-Name:** `CoreGraphics.frameworkb`

### Collaboration
**Datenaustausch.** Regelt das Teilen von Daten und Identitäten zwischen Apps (Pasteboard, Share Sheets).
* **Bundle-Name:** `Collaboration.frameworkb`

---

## 📁 Framework-Bundle-Struktur (.frameworkb)

```text
Name.frameworkb/
├── Name                 # Shared Object / Umbrella Library
├── Headers/             # C-Header-Dateien
├── Resources/           # Lokalisierung, Assets und Icons
└── Info.json            # Metadaten, Versionierung und Abhängigkeiten
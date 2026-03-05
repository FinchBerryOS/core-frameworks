# FinchBerryOS Frameworks
[English](README.md) | [Deutsch](README.de.md)

Dieses Repository enthält die High-Level C-basierten Frameworks für **FinchBerryOS**. Diese Frameworks bieten die primären APIs für Anwendungen (`.appd`) und Systemdienste (`.serviced`) und abstrahieren den zugrunde liegenden Linux-Kernel sowie die Core-Daemons in eine saubere, von macOS inspirierte Schnittstelle.

## 🏗 Architektur

Die Framework-Schicht sitzt zwischen den Low-Level `core-services` (Daemons wie `syscored`, `kmodsysd`, etc.) und den benutzerorientierten Anwendungen. Jedes Framework ist als **`.frameworkb`** (Framework-Verzeichnis) paketiert und linkt gegen **`libfinch.so`** für systemweiten XPC- und C-Library-Zugriff.

---

## 📦 Die Frameworks

#### GNUCore
**Die Headless-Basis.** Fungiert als essentielle Brücke zum Linux-Ökosystem. Sie enthält Kern-Bibliotheken für den Boot-Vorgang, Netzwerk-Management und die Hardware-Registry.
* **Bundle-Name:** `GNUCore.frameworkb`
* **Libraries:** `libc`, `libudev`, `libkmod`, `libssl`, `libz`, etc.

#### GNUCoreExtensions
**Der Multimedia-Layer.** Erweitert `GNUCore` um Grafik-, Audio- und Eingabefähigkeiten. Erforderlich für GUIs.
* **Bundle-Name:** `GNUCoreExtensions.frameworkb`
* **Abhängigkeit:** Erfordert `GNUCore.frameworkb`.

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
**Die Hardware-Registrierung.** Bietet eine objektorientierte Sicht auf die Hardware des Systems.
* **Bundle-Name:** `IOKit.frameworkb`
* **Backend:** Kommuniziert mit `kmodsysd`.

### CoreSystem
**Die OS-Basis.** Bietet grundlegende Primitiven für fast jede Binary.
* **Bundle-Name:** `CoreSystem.frameworkb`
* **Funktionalität:** Unified Logging (`cs_log`), Speicher-Helfer, Notifizierungen.

### NetKit
**Netzwerk & Dezentralisierung.** High-Performance Stack für Standard- und souveräne Verbindungen (WireGuard, Bitcoin P2P, HTTP/3).
* **Bundle-Name:** `NetKit.frameworkb`

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
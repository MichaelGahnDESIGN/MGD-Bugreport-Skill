# 08. Plattformen

Jede Plattform hat eigene Formate, Sicherheitsanforderungen und Eigenheiten. Dieser Abschnitt gibt einen Überblick.

---

## macOS

### Formate

| Format | Beschreibung |
|--------|-------------|
| `.dmg` | Disk-Image, Standard für macOS-Apps |
| `.pkg` | Installer-Paket, für Apps mit System-Installation |
| `.zip` | Signiertes ZIP für einfache Apps |

### Sicherheit

**Code-Signierung ist Pflicht.**

Ohne gültiges Apple Developer Certificate:
- Gatekeeper blockiert den Start
- Nutzer sehen "nicht vertrauenswürdige App"

**Notarisierung** (notarytool) ist für öffentliche Releases notwendig.

**Apple Silicon und Intel:**
Universal-Binary (.universal2) oder getrennte Builds empfohlen.

### Update-Pfad

```text
Signiertes .dmg bauen
↓
Notarisieren (apple notarytool)
↓
Auf Update-Server hochladen
↓
SHA-256 berechnen
↓
Manifest aktualisieren
```

→ Checkliste: [`checklists/macos.md`](../checklists/macos.md)

---

## Windows

### Formate

| Format | Beschreibung |
|--------|-------------|
| `.exe` | Setup-Exe (z.B. mit NSIS, Inno Setup) |
| `.msi` | Windows Installer, für Enterprise |
| `MSIX` | Moderne Paketformat, Store-kompatibel |

### Sicherheit

**Code-Signierung empfohlen.**

Ohne Signierung:
- SmartScreen-Warnung beim ersten Start
- Nutzer müssen explizit "Trotzdem ausführen" wählen

EV (Extended Validation) Zertifikate bauen schneller Vertrauen auf als normale OV-Zertifikate.

**UAC (User Account Control):**
Installer die in `Program Files` installieren, brauchen Admin-Rechte. Apps die nur in `%LOCALAPPDATA%` installieren, vermeiden UAC.

### Update-Pfad

```text
Signiertes .exe oder .msi bauen
↓
Auf Update-Server hochladen
↓
SHA-256 berechnen
↓
Manifest aktualisieren
```

→ Checkliste: [`checklists/windows.md`](../checklists/windows.md)

---

## Linux

### Formate

| Format | Beschreibung |
|--------|-------------|
| `.AppImage` | Portabel, kein root, läuft auf fast allen Distros |
| `.deb` | Debian/Ubuntu-Pakete |
| `.rpm` | Fedora/RHEL-Pakete |
| `Flatpak` | Sandboxed, distributionsunabhängig |
| `Snap` | Ubuntu-zentriert, Canonical-kontrolliert |

**Empfehlung für maximale Kompatibilität:** AppImage als primäres Format, optional Flatpak.

### Eigenheiten

- Keine einheitliche Installer-Infrastruktur
- Berechtigungen und Desktop-Integration variieren je nach Distro
- Auto-Updater: Flatpak und Snap haben eingebaute Update-Mechanismen

→ Checkliste: [`checklists/linux.md`](../checklists/linux.md)

---

## Mobile — iOS & Android

Binär-Updates bei mobilen Apps gehen fast immer über die App Stores.

Das eigene Backend kann trotzdem für folgendes genutzt werden:
- Mindestversions-Prüfung (Nutzer zur Store-Seite leiten)
- Content-Updates (Daten, Konfiguration, Feature-Flags)
- API-Kompatibilitätsprüfungen

**Wichtig:** Remote-Code-Loading ist in den App Stores meist verboten. Nur zulässige Content-Update-Mechanismen nutzen.

---

## Flutter Desktop

Flutter Desktop unterstützt macOS, Windows und Linux aus einer Codebasis.

Besonderheiten:
- Kein eingebauter Auto-Updater in Flutter
- `pub.dev`-Pakete verfügbar (z.B. `upgrader`)
- Phase 1 und 2 gut manuell umsetzbar

→ Beispiel: [`examples/flutter-desktop.md`](../examples/flutter-desktop.md)

---

## Electron

Electron hat `autoUpdater` eingebaut (basiert auf Squirrel).

Besonderheiten:
- Squirrel erfordert Code-Signierung auf macOS und Windows
- `electron-updater` aus `electron-builder` ist die häufig genutzte Alternative
- Release-Kanäle (stable, beta) werden gut unterstützt

→ Beispiel: [`examples/electron.md`](../examples/electron.md)

---

## Tauri

Tauri hat einen eingebauten Updater.

Besonderheiten:
- Updater-Endpunkt muss signiert sein
- Tauri verwendet Squirrel auf Windows und macOS-Mechanismen
- Konfiguration in `tauri.conf.json`

→ Beispiel: [`examples/tauri.md`](../examples/tauri.md)

---

## Nächster Schritt

→ Weiter mit [`09-API-Clients.md`](09-API-Clients.md)

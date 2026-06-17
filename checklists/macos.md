# macOS-Checkliste

## Format wählen

- [ ] `.dmg` — Standard für die meisten macOS-Apps
- [ ] `.pkg` — wenn System-weite Installation nötig ist
- [ ] `.zip` (signiert) — für einfache, portable Apps

## Signierung

- [ ] Apple Developer Account vorhanden und aktiv
- [ ] Developer ID Application Zertifikat vorhanden
- [ ] App-Bundle ist signiert (`codesign`)
- [ ] Alle Frameworks und Bibliotheken sind ebenfalls signiert

## Notarisierung

- [ ] App wird an Apple zur Notarisierung eingereicht (`notarytool`)
- [ ] Notarisierungsticket wird in die App eingebunden (`staple`)
- [ ] Gatekeeper-Verhalten ist lokal getestet

## Architekturen

- [ ] Apple Silicon (arm64) wird unterstützt
- [ ] Intel (x86_64) wird unterstützt falls noch relevant
- [ ] Universal Binary (.universal2) wenn beide Architekturen aus einem Build

## Download und Installation

- [ ] HTTPS für Manifest und Download
- [ ] SHA-256 ist nach dem Build berechnet
- [ ] Download-Test auf sauberem macOS-System
- [ ] Installer funktioniert ohne vorherige Entwickler-Tools

## Release

- [ ] Manifest (`latest.json`) nach dem Upload aktualisiert
- [ ] Download-URL getestet (echter Download)
- [ ] Rollback-Version ist verfügbar

# Windows-Checkliste

## Format wählen

- [ ] `.exe` — Klassischer Setup-Installer (NSIS, Inno Setup, WiX)
- [ ] `.msi` — Windows Installer, für Enterprise-Umgebungen
- [ ] `MSIX` — Modernes Format, Microsoft Store kompatibel

## Code-Signierung

- [ ] Code-Signing-Zertifikat vorhanden (OV oder EV)
- [ ] Installer ist signiert (`signtool`)
- [ ] SmartScreen-Verhalten ist getestet
- [ ] Hinweis: EV-Zertifikate bauen schneller Vertrauen auf

## Installation

- [ ] Installer wurde auf sauberem Windows-System getestet
- [ ] Verhalten ohne Admin-Rechte ist definiert
- [ ] UAC-Anforderung ist nur vorhanden wenn wirklich nötig
- [ ] Deinstallation funktioniert sauber

## Kompatibilität

- [ ] Getestete Windows-Versionen sind dokumentiert (Win 10, Win 11, ...)
- [ ] 32-Bit / 64-Bit-Variante ist definiert
- [ ] Verhalten auf ARM Windows ist bekannt (falls relevant)

## Download und Release

- [ ] HTTPS für Manifest und Download
- [ ] SHA-256 ist nach dem Build berechnet
- [ ] Download-URL getestet (echter Download)
- [ ] Manifest (`latest.json`) nach Upload aktualisiert

# Linux-Checkliste

## Format wählen

- [ ] `.AppImage` — Portabel, kein root, läuft auf fast allen Distributionen
- [ ] `.deb` — Debian/Ubuntu/Mint
- [ ] `.rpm` — Fedora/RHEL/openSUSE
- [ ] `Flatpak` — Distributionsunabhängig, Sandbox-isoliert
- [ ] `Snap` — Ubuntu-zentriert

**Empfehlung für maximale Reichweite:** AppImage als primäres Format.

## Unterstützte Distributionen

- [ ] Unterstützte Distributionen und Versionen sind dokumentiert
- [ ] Tests auf mindestens Ubuntu LTS und Fedora aktuell
- [ ] Glibc-Version Anforderung ist bekannt und dokumentiert

## AppImage

- [ ] Ausführbar-Bit gesetzt (`chmod +x`)
- [ ] Desktop-Integration getestet (`.desktop`-Datei, Icon)
- [ ] AppImageTool oder linuxdeploy verwendet

## Paket-Signierung

- [ ] `.deb`: GPG-Signatur vorhanden (optional, aber empfohlen)
- [ ] Flatpak: Signierung im Flatpak-System konfiguriert

## Download und Release

- [ ] HTTPS für Manifest und Download
- [ ] SHA-256 ist nach dem Build berechnet
- [ ] Download-URL getestet (echter Download)
- [ ] Dateiberechtigungen nach dem Download korrekt
- [ ] Manifest (`latest.json`) nach Upload aktualisiert

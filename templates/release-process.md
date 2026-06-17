# Release-Prozess Vorlage

Dieses Template dokumentiert den Ablauf für ein neues Release. Für jedes Release kopieren und ausfüllen.

---

## Release-Informationen

- **App:** 
- **Version:** 
- **Plattform(en):** 
- **Kanal:** stable / beta / nightly
- **Release-Datum:** 
- **Veröffentlicht von:** 

---

## Checkliste

### Vorbereitung

- [ ] Version in der App aktualisiert
- [ ] CHANGELOG aktualisiert (Datum und Beschreibung)
- [ ] Tests lokal durchgeführt und bestanden
- [ ] Alle offenen Blocker-Issues geschlossen

### Build

- [ ] Sauberer Build ohne Warnungen
- [ ] Build auf Zielplattform(en) getestet
- [ ] macOS: Code-signiert und notarisiert
- [ ] Windows: Installer signiert
- [ ] Linux: Paket erstellt und getestet

### Upload

- [ ] Installer-Datei(en) auf Update-Server hochgeladen
- [ ] SHA-256 für jede Datei berechnet
- [ ] SHA-256 notiert für Manifest-Eintrag

### Manifest aktualisieren

- [ ] `latestVersion` aktualisiert
- [ ] `downloadUrl` für jede Plattform aktualisiert
- [ ] `sha256` für jede Plattform eingetragen
- [ ] `changelog` aktualisiert
- [ ] `forceUpdate` korrekt gesetzt
- [ ] `publishedAt` gesetzt
- [ ] `minimumVersion` geprüft (Anpassung nötig?)

### Testen

- [ ] Manifest-URL gibt korrekte JSON-Antwort zurück
- [ ] Update-Dialog erscheint in der App (mit alter Version testen)
- [ ] Download-Link funktioniert
- [ ] Installer installiert erfolgreich
- [ ] App startet nach Installation korrekt
- [ ] Update-Dialog erscheint nicht mehr nach der Installation

### Abschluss

- [ ] Git-Tag für diese Version gesetzt (z.B. `v1.0.1`)
- [ ] Rollback-Version ist verfügbar falls nötig
- [ ] Team informiert

---

## SHA-256 Werte (für Manifest)

| Plattform | SHA-256 |
|-----------|---------|
| macOS | |
| Windows | |
| Linux | |

---

## SHA-256 berechnen

```bash
# macOS / Linux
shasum -a 256 example-app-1.0.1-macos.dmg

# Windows (PowerShell)
Get-FileHash example-app-1.0.1-windows.exe -Algorithm SHA256
```

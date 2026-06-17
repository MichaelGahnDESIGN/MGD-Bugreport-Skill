# 05. Phase 2 — Geführte Downloads

Phase 2 lässt die App den Installer selbst herunterladen — ohne dass der Nutzer einen Browser öffnen muss. Die Installation bleibt aber noch manuell und nachvollziehbar.

Phase 2 erst implementieren wenn Phase 1 stabil läuft und der Release-Prozess dokumentiert ist.

---

## Ablauf

```text
Nutzer klickt "Update herunterladen"
↓
App zeigt Fortschrittsbalken
↓
Datei wird in sicheres Temp-Verzeichnis heruntergeladen
↓
Checksumme wird geprüft
↓
Dialog: "Download abgeschlossen — Installer öffnen?"
↓
Nutzer klickt "Öffnen"
↓
Installer startet
↓
App informiert den Nutzer was als nächstes passiert
```

---

## Neue Felder im Manifest

```json
{
  "app": "example-app",
  "platform": "windows",
  "latestVersion": "1.1.0",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.1.0-windows.exe",
  "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
  "fileSize": 45234176,
  "changelog": [
    "Automatischer Download-Dialog",
    "Verbesserte Fehlerbehandlung"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Checksummen-Verifikation

**Warum Checksummen?**
Ein manipulierter oder beschädigter Download wird erkannt, bevor der Nutzer ihn ausführt.

**Empfohlen:** SHA-256

```text
Erwarteter Hash aus Manifest: sha256-Feld
Tatsächlicher Hash der heruntergeladenen Datei: lokal berechnen
Vergleich → stimmt nicht → Download verwerfen, Fehler anzeigen
```

**Wichtig:** Checksummen schützen vor Beschädigung und einfacher Manipulation. Für vollständigen Schutz ist zusätzlich Code-Signierung nötig (Phase 3+).

---

## Download-Ort

Sichere temporäre Verzeichnisse:

| Plattform | Empfohlen |
|-----------|-----------|
| macOS | `~/Downloads/` oder `NSTemporaryDirectory()` |
| Windows | `%TEMP%\` oder `%LOCALAPPDATA%\Temp\` |
| Linux | `/tmp/` |

Niemals in App-Verzeichnisse schreiben. Keine Schreibrechte voraussetzen die der Nutzer nicht hat.

---

## Fehlerbehandlung

| Fehler | Reaktion |
|--------|----------|
| Netzwerkfehler | "Download fehlgeschlagen. Erneut versuchen?" |
| Checksumme falsch | "Download beschädigt. Erneut versuchen?" |
| Nicht genug Speicher | Frühzeitig prüfen, verständliche Meldung |
| Download abgebrochen | Temp-Datei löschen |
| Timeout | Nach 30–60 Sekunden Inaktivität |

---

## Was der Nutzer sehen soll

- Dateiname und -größe vor dem Download
- Fortschrittsbalken mit Prozent und verbleibender Zeit
- Möglichkeit den Download abzubrechen
- Klare Aussage was nach dem Download passiert
- Klare Fehlermeldung wenn etwas schiefläuft

---

## Checkliste Phase 2

→ Vollständige Checkliste: [`checklists/phase-2.md`](../checklists/phase-2.md)

---

## Nächster Schritt

→ Weiter mit [`06-Phase-3-Auto-Updater.md`](06-Phase-3-Auto-Updater.md)

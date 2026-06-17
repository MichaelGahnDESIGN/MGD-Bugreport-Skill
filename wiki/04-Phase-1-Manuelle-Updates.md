# 04. Phase 1 — Manuelle Updates

Phase 1 ist der empfohlene Einstieg für die meisten Projekte.

Die App prüft ob eine neue Version existiert und zeigt dem Nutzer einen Download-Link. Der Nutzer installiert das Update selbst.

---

## Ablauf

```text
Nutzer startet App
↓
App prüft latest.json
↓
Neue Version gefunden
↓
Dialog zeigt Changelog
↓
Nutzer klickt "Jetzt herunterladen"
↓
Browser öffnet Installer-URL
↓
Nutzer installiert manuell
```

---

## Minimales Manifest

```json
{
  "app": "example-app",
  "platform": "macos",
  "latestVersion": "1.0.1",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.0.1-macos.dmg",
  "changelog": [
    "Update-Prüfung hinzugefügt",
    "Einstellungen verbessert",
    "Startproblem behoben"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Was die App implementieren muss

**Minimale Implementierung:**

1. Eigene installierte Version aus der App-Konfiguration lesen
2. Aktuelle Plattform erkennen (macOS / Windows / Linux)
3. Manifest-URL per HTTPS laden
4. JSON parsen
5. Versionen vergleichen
6. Bei Update: Dialog anzeigen mit Versionsnummer und Changelog
7. Bei Klick: `downloadUrl` im Standard-Browser öffnen
8. Bei Netzwerkfehler: still scheitern, keine Abstürze

**Pflicht-Update (forceUpdate):**

Wenn `forceUpdate: true` oder die installierte Version unter `minimumVersion` liegt:
- Kein "Später" anbieten
- Update klar als notwendig markieren
- App-Start ggf. blockieren

---

## Versionsnummern vergleichen

Semver-Vergleich (`1.2.3` vs `1.2.4`):

```text
Split by "."
Vergleiche MAJOR → MINOR → PATCH
Erste Differenz entscheidet
```

Niemals als String vergleichen: `"1.10.0" < "1.9.0"` wäre falsch.

---

## Häufige Fehler vermeiden

| Fehler | Richtig |
|--------|---------|
| Update-Prüfung blockiert App-Start | Im Hintergrund prüfen |
| Absturz bei fehlendem Netzwerk | Fehler still behandeln |
| Versionsvergleich als String | Numerisch vergleichen |
| Kein Timeout für HTTPS-Request | Timeout von 10–30 Sekunden |
| Update-Dialog beim ersten Start | Erste Prüfung verzögern |

---

## Update-Dialog Empfehlungen

Ein guter Update-Dialog zeigt:
- Aktuelle Version
- Neue Version
- Changelog (sortiert, übersichtlich)
- Button "Jetzt herunterladen"
- Button "Später" (außer bei Pflicht-Updates)
- Optional: "Diese Version überspringen"

---

## Checkliste Phase 1

→ Vollständige Checkliste: [`checklists/phase-1.md`](../checklists/phase-1.md)

---

## Nächster Schritt

→ Weiter mit [`05-Phase-2-Gefuehrte-Downloads.md`](05-Phase-2-Gefuehrte-Downloads.md)

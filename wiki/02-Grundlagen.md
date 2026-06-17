# 02. Grundlagen

---

## Die zwei Update-Typen

### App-Updates

App-Updates ersetzen das Programm selbst.

Beispiele: `.dmg`, `.exe`, `.msi`, `.AppImage`, mobile App-Releases oder Game-Client-Builds.

### Content-Updates

Content-Updates ändern Daten die von der App geladen werden — nicht die App selbst.

Beispiele: Spiellevel, Quests, Übersetzungen, Templates, Remote-Konfiguration, Feature-Flags, API-Kompatibilitäts-Metadaten.

**Faustregel:** App-Updates und Content-Updates wann immer möglich trennen.

---

## Versionierung

Eine klare Versionsstrategie ist die Grundlage jedes Update-Systems.

Empfohlen: Semantic Versioning — `MAJOR.MINOR.PATCH`

```text
1.0.0  — Erstveröffentlichung
1.0.1  — Bugfix
1.1.0  — Neue Funktion (rückwärtskompatibel)
2.0.0  — Breaking Change
```

Ein nützliches Manifest enthält:
- `latestVersion` — aktuellste verfügbare Version
- `minimumVersion` — älteste noch unterstützte Version

---

## Das Update-Manifest

Das Manifest ist eine JSON-Datei auf einem öffentlichen Server. Die App lädt sie und entscheidet ob ein Update nötig ist.

```json
{
  "app": "example-app",
  "platform": "macos",
  "latestVersion": "1.1.0",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.1.0-macos.dmg",
  "changelog": [
    "Neue Exportfunktion",
    "Verbesserte Ladezeit",
    "Bugfix beim Speichern"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Statisches JSON oder API?

**Statisches JSON** reicht für viele frühe Projekte.

Vorteile:
- Kein Server-Code nötig
- Auf jedem Webspace oder CDN hostbar
- Einfach zu aktualisieren

**Eine API** ist sinnvoll wenn Updates abhängen von:
- Lizenzstatus
- Kundengruppen
- Rollout-Kanälen (stable, beta, nightly)
- Sprachen oder Regionen
- Plattformspezifischen Regeln

**Empfehlung:** Mit statischem JSON starten. API erst wenn wirklich nötig.

---

## Update-Kanäle

Update-Kanäle erlauben verschiedene Versionen für verschiedene Nutzergruppen:

| Kanal | Beschreibung |
|-------|-------------|
| `stable` | Getestete Releases für alle Nutzer |
| `beta` | Vorabtests für interessierte Nutzer |
| `nightly` | Tagesaktuelle Builds für Entwickler |

Kanäle sind optional. Die meisten Projekte brauchen anfangs nur `stable`.

---

## Nächster Schritt

→ Weiter mit [`03-Architektur.md`](03-Architektur.md)

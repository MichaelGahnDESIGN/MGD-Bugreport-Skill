# MGD App Updater Skill

**Ein universeller Skill für KI-Agenten zur Planung sicherer Update-Systeme.**

Für Flutter, Electron, Tauri, native Desktop-Apps, Mobile Apps, Spiele, API-Clients, SaaS-Clients und Self-Hosted-Software.

---

## Was ist dieser Skill?

Der MGD App Updater Skill ist eine Planungs- und Dokumentationsstruktur für Software-Update-Systeme.

Er ist keine fertige Bibliothek. Er ist ein Skill, der KI-Agenten zwingt, **zuerst zu planen und dann erst zu implementieren**.

Update-Systeme ersetzen Software auf dem Computer des Nutzers. Das macht sie sicherheitskritisch. Ein schlecht geplanter Updater kann zur Angriffsfläche werden. Dieser Skill verhindert das durch strukturiertes Vorgehen.

---

## Für wen ist dieser Skill?

**Menschen:**
- Entwickler die ein Update-System für ihre App einbauen wollen
- Open-Source-Maintainer die professionelle Releases anbieten wollen
- Indie-Entwickler ohne Update-Erfahrung

**KI-Agenten:**
- ChatGPT Codex
- Claude Code
- Cursor
- Windsurf
- Gemini CLI
- Andere Coding-Agenten

---

## Unterstützte Plattformen

| Kategorie | Plattformen |
|-----------|-------------|
| Desktop-Apps | Flutter Desktop, Electron, Tauri, native macOS, native Windows, native Linux |
| Mobile Apps | Flutter Mobile, native iOS, native Android |
| Spiele | Game-Clients, Content-Updates, Patchsysteme |
| API-Clients | Wetter-Apps, Finanz-Apps, KI-Clients, SaaS-Clients |
| Server-Software | Self-Hosted-Apps, Open-Source-Projekte, interne Tools |

---

## Warum existiert dieser Skill?

Viele Apps brauchen irgendwann Updates. Das klingt simpel, wirft aber schnell wichtige Fragen auf:

- Woher weiß die App, dass eine neue Version existiert?
- Wo werden Installer gespeichert?
- Was passiert wenn eine alte Version nicht mehr mit der API kompatibel ist?
- Wie werden Downloads verifiziert?
- Sind Updates optional oder verpflichtend?

Dieser Skill beantwortet diese Fragen Schritt für Schritt — bevor Code geschrieben wird.

---

## Philosophie

> Planen vor Implementieren.  
> Sicherheit vor Komfort.  
> Einfach starten, sicher wachsen.

Die meisten Projekte sollten nicht mit einem komplexen Auto-Updater beginnen. Ein gutes erstes Update-System prüft einfach eine JSON-Datei, vergleicht Versionen, zeigt ein Changelog und öffnet einen Download-Link.

---

## Update-Reifegrade

| Stufe | Name | Beschreibung |
|-------|------|-------------|
| 1 | Manueller Update-Hinweis | App prüft eine Versionsdatei und öffnet einen Download-Link |
| 2 | Geführter Download | App zeigt Changelog, lädt den Installer herunter und verifiziert ihn |
| 3 | Automatischer Updater | App lädt herunter, verifiziert und installiert Updates sicher |
| 4 | Sicheres Release-System | Signierung, Checksummen, erzwungene Updates, Rollback |
| 5 | Kommerzielle Distribution | Lizenzprüfung, Staged Rollout, Enterprise-Kanäle |

**Die meisten Projekte starten bei Stufe 1.**

---

## Schnellstart

### Schritt 1 — Skill-Datei lesen

```text
skill/SKILL.md
```

### Schritt 2 — Agenten-Prompt

Für ChatGPT Codex oder Claude Code:

```text
Verwende den MGD App Updater Skill. Analysiere mein Projekt und erstelle eine Phase-1-Roadmap.
Schreibe noch keinen Code. Erkläre zuerst Architektur, Risiken, Checkliste und offene Fragen.
```

### Schritt 3 — Minimalstruktur

Die einfachste Update-Architektur:

```text
App
↓ lädt
https://updates.example.com/example-app/platform/latest.json
↓ vergleicht Version
Update-Dialog
↓ öffnet
Download-URL im Browser
```

### Schritt 4 — Minimales Update-Manifest

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

## Skill-Befehle

```text
/updateservice analyse          — Projekt analysieren
/updateservice roadmap          — Update-Roadmap erstellen
/updateservice architecture     — Architektur planen
/updateservice phase1           — Phase 1 umsetzen
/updateservice phase2           — Phase 2 umsetzen
/updateservice phase3           — Phase 3 umsetzen
/updateservice security         — Sicherheitsanalyse
/updateservice flutter          — Flutter-spezifische Planung
/updateservice electron         — Electron-spezifische Planung
/updateservice tauri            — Tauri-spezifische Planung
/updateservice game             — Spiel-Client-Planung
/updateservice mobile           — Mobile-App-Planung
/updateservice api-client       — API-Client-Planung
/updateservice weather-app      — Wetter-App-Beispiel
/updateservice selfhosted       — Self-Hosted-Planung
/updateservice github           — GitHub-Releases-Workflow
/updateservice checklist        — Passende Checkliste
```

---

## Beispiele

| Beispiel | Datei |
|----------|-------|
| Flutter Desktop | [`examples/flutter-desktop.md`](examples/flutter-desktop.md) |
| Electron | [`examples/electron.md`](examples/electron.md) |
| Tauri | [`examples/tauri.md`](examples/tauri.md) |
| Spiel-Client | [`examples/game-client.md`](examples/game-client.md) |
| Wetter-App | [`examples/weather-app.md`](examples/weather-app.md) |
| API-Client | [`examples/api-client.md`](examples/api-client.md) |
| PHP Update-API | [`examples/php-update-api.md`](examples/php-update-api.md) |

---

## Projektstruktur

```text
README.md                          — Diese Datei
LICENSE                            — MIT-Lizenz
IMPRESSUM.md                       — Impressum (§ 5 TMG)
CHANGELOG.md                       — Versionshistorie
CONTRIBUTING.md                    — Beitrag leisten

skill/
  SKILL.md                         — Kern-Skill für KI-Agenten

wiki/
  01-Einfuehrung.md
  02-Grundlagen.md
  03-Architektur.md
  04-Phase-1-Manuelle-Updates.md
  05-Phase-2-Gefuehrte-Downloads.md
  06-Phase-3-Auto-Updater.md
  07-Sicherheit.md
  08-Plattformen.md
  09-API-Clients.md
  10-Spiele-und-Content-Updates.md
  11-GitHub-Releases.md
  12-Self-Hosted-Updates.md
  13-Lizenzserver.md
  14-FAQ.md
  15-Glossar.md

examples/
  flutter-desktop.md
  electron.md
  tauri.md
  game-client.md
  weather-app.md
  api-client.md
  php-update-api.md

templates/
  latest.json
  version-api-response.json
  remote-config.json
  release-process.md

checklists/
  phase-1.md
  phase-2.md
  phase-3.md
  macos.md
  windows.md
  linux.md
```

---

## Sicherheit

Update-Systeme sind sicherheitskritisch. Die wichtigsten Regeln:

- Niemals private Tokens, API-Keys oder Signing-Zertifikate in eine ausgelieferte App einbetten
- Niemals Remote-Skripte ohne Verifikation ausführen
- Niemals Binärdateien ohne Integritätsprüfung ersetzen
- Immer HTTPS für Manifeste und Downloads
- Checksummen für Phase 2, Signaturen für Phase 3+

Alles was in eine App ausgeliefert wird, muss als extrahierbar betrachtet werden.

Mehr dazu: [`wiki/07-Sicherheit.md`](wiki/07-Sicherheit.md)

---

## Öffentlichkeitsprinzip

Dieses Repository ist öffentlich. Daher gilt:

- Keine privaten Repository-URLs
- Keine Kundenprojekte, keine NDA-Inhalte
- Keine internen Server oder private Infrastruktur
- Im Zweifel: nicht erwähnen

Beispiele verwenden immer generische Namen wie `example-app`, `updates.example.com` und `api.example.com`.

---

## Wiki

Das vollständige Handbuch beginnt hier:

**→ [`wiki/01-Einfuehrung.md`](wiki/01-Einfuehrung.md)**

---

## Lizenz

MIT License. Siehe [`LICENSE`](LICENSE).

---

## Impressum

Siehe [`IMPRESSUM.md`](IMPRESSUM.md).

---

*MGD App Updater Skill — von [Michael Gahn DESIGN](https://michael-gahn.de)*

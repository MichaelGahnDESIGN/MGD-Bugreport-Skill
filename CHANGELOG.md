# Changelog

## [1.0.0] - 2026-06-17

### Erste stabile Version

- SKILL.md auf Version 1.0 gesetzt — produktionsreif und vollständig
- README.md: Abschnitt "Was enthält v1.0?" ergänzt
- GitHub Topics gesetzt für maximale Auffindbarkeit
- Alle Inhalte final geprüft und freigegeben

---

## [0.3.0] - 2026-06-17

### Neu

- **Wiki erweitert auf 17 Dateien**
  - `wiki/09-Swift-und-SwiftUI.md` — App Store vs. Direktdownload, Swift UpdateService mit async/await, SwiftUI-Dialog
  - `wiki/10-PHP-Backends.md` — Plain-PHP-API, Flight-PHP-Variante, Sicherheitsregeln-Tabelle
  - Bestehende Dateien 09–15 verschoben nach 11–17 (neue Nummerierung)
- **15 Beispiele** (8 neue)
  - `examples/flutter-mobile.md` — Mindestversion-Prüfung, App Store / Play Store Redirect
  - `examples/swift-macos.md` — Swift UpdateService (async/await), SwiftUI-Alert, @main-Integration
  - `examples/swift-ios.md` — App-Store-Regeln, UIAlertController, Weiterleitung
  - `examples/unity.md` — C# UpdateManifest, UnityWebRequest, Addressables Content-Updates
  - `examples/godot.md` — GDScript Content-Updater, PCK-Dateien laden, Sicherheitshinweis
  - `examples/laravel-update-api.md` — Laravel-Controller mit Lizenzvalidierung
  - `examples/symfony-update-api.md` — Symfony-Controller mit Route-Annotations, LicenseService
  - `examples/nodejs-update-api.md` — Express.js, Path-Traversal-Schutz, Rate-Limiting
- **8 Checklisten** (2 neue)
  - `checklists/ios.md` — App-Store-Regeln, TestFlight, Datenschutz
  - `checklists/android.md` — Play Core API, APK-Signierung, Berechtigungen

### Geändert

- **SKILL.md komplett überarbeitet**
  - Technologie-Neutralitätsprinzip: Plattform- und Technologie-Erkennung zuerst
  - Agenten-Regeln für Claude Code, ChatGPT Codex, Cursor, Windsurf, Gemini CLI
  - Verteilungsmodell-Tabelle (App Store, Direct, Enterprise, Self-Hosted)
  - 15 Analyse-Fragen vor jeder Implementierung
  - Mermaid-Flussdiagramm für Phase-1-Architektur
- **README.md vollständig erneuert**
  - Untertitel: "Das deutschsprachige Open-Source-Handbuch für sichere Software-Update-Systeme"
  - Technologie-Tabellen für Desktop, Mobile, Spiele, Backend, Deployment
  - Mermaid-Diagramm für alle 5 Update-Reifegrade
  - 15 Beispiele mit Links, vollständige Projektstruktur
- **`wiki/08-Plattformen.md`** — Mermaid-Diagramm, Swift/SwiftUI, React Native ergänzt

---

## [0.2.0] - 2026-06-17

### Geändert

- Vollständiger Neuaufbau der Dokumentation auf Deutsch
- README komplett neu geschrieben — professioneller, ausführlicher, strukturierter
- SKILL.md komplett auf Deutsch übersetzt und erweitert
- CONTRIBUTING.md auf Deutsch
- Wiki-Dateien umbenannt auf deutsche Dateinamen
- Alle Wiki-Inhalte auf Deutsch übersetzt und ausgebaut
- Alle Beispiele auf Deutsch und inhaltlich erweitert
- Alle Checklisten auf Deutsch
- Öffentlichkeitsregel in README, SKILL, CONTRIBUTING und Wiki verankert

### Hinzugefügt

- `wiki/12-Self-Hosted-Updates.md`
- `wiki/13-Lizenzserver.md`
- `wiki/14-FAQ.md` (aus `12-faq.md` verschoben und erweitert)
- `wiki/15-Glossar.md`

### Entfernt

- Englische Wiki-Dateinamen (`01-introduction.md` usw.)

---

## [0.1.0] - 2026-06-17

### Hinzugefügt

- Erste öffentliche Projektstruktur
- Universeller KI-Agenten-Updater-Skill
- README, Lizenz, Impressum und Beitragsleitfaden
- Wiki-Dokumentation
- Phasen-Checklisten
- Plattform-Checklisten
- Beispiele für API-Clients, Wetter-Apps, Game-Clients, Flutter, Electron und Tauri
- Templates für Update-Manifeste, Remote-Konfiguration und Release-Prozesse

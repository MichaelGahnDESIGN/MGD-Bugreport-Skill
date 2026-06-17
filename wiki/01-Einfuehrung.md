## Einführung: MGD Bugreport Skill

Der MGD Bugreport Skill ist eine Planungsstruktur für KI-Agenten und Entwickler, die professionelle Bug-Report-, Feedback- und QA-Systeme in ihre Anwendungen integrieren möchten. Er ist kein Framework, keine Bibliothek und kein fertiges Produkt — er ist eine strukturierte Anleitung, die KI-Agenten wie Claude Code, Codex oder Cursor befähigt, die richtige Implementierung für einen konkreten Anwendungsfall zu planen und umzusetzen.

### Was ist der MGD Bugreport Skill?

Der Skill definiert, **wie** ein Feedback-System aufgebaut sein soll, **welche Daten** erfasst werden dürfen, und **welche Entscheidungen** vor der Implementierung getroffen werden müssen. Er beantwortet Fragen wie:

- Welche Feedback-Typen soll die Anwendung unterstützen?
- Wie werden Screenshots datenschutzkonform verarbeitet?
- Welche technischen Daten dürfen automatisch erfasst werden?
- Wie sieht der Datenfluss vom Nutzer bis zum Entwickler-Dashboard aus?

Der Skill gibt keine Antworten vor — er stellt sicher, dass die richtigen Fragen gestellt werden, bevor Code geschrieben wird.

### Warum ist Planung wichtig?

Feedback-Systeme berühren sensible Bereiche:

**Datenschutz:** Screenshots können persönliche Daten enthalten. Systemdaten können zur Profilbildung genutzt werden. In Europa gilt die DSGVO (Datenschutz-Grundverordnung), die eine informierte Einwilligung verlangt.

**Screenshot-Sensibilität:** Ein Screenshot, der automatisch beim Klick auf "Fehler melden" aufgenommen wird, kann Passwörter, private Nachrichten oder sensible Geschäftsdaten enthalten. Ohne Vorschau und Opt-out ist das ein Datenschutzproblem.

**Nutzererfahrung:** Ein zu komplexes Formular wird nicht ausgefüllt. Ein zu einfaches liefert keine verwertbaren Informationen. Die Balance ist entscheidend.

**Plattformunterschiede:** Screenshot-APIs, Systemdaten und Dateizugriff funktionieren auf macOS, Windows, Linux, iOS, Android und im Web grundlegend unterschiedlich.

### Struktur des Skills

| Verzeichnis | Inhalt |
|---|---|
| `skill/` | Die eigentliche Skill-Datei für KI-Agenten (YAML/Markdown) |
| `wiki/` | Diese Dokumentation — vollständige Erklärung aller Konzepte |
| `examples/` | Beispiel-Implementierungen für verschiedene Plattformen und Frameworks |
| `checklists/` | Abhaklisten für Entwickler vor dem Launch |
| `templates/` | Wiederverwendbare Formulare, JSON-Schemata und UI-Komponenten |

### Für wen ist dieser Skill?

**KI-Agenten:**
- Claude Code (Anthropic)
- Codex / ChatGPT (OpenAI)
- Cursor
- Windsurf
- Gemini CLI (Google)

Der Skill wurde so formuliert, dass er von KI-Agenten direkt als Planungsgrundlage genutzt werden kann. Ein Agent, der diesen Skill geladen hat, kann einen Entwickler durch den gesamten Implementierungsprozess führen — von der Konzeptentscheidung bis zum ersten funktionierenden Formular.

**Entwickler:**
- Indie-Entwickler, die ein einfaches Feedback-Widget für ihre App benötigen
- Open-Source-Maintainer, die Bug-Reports strukturieren möchten
- Teams, die ein internes QA-System aufbauen
- Agenturen, die für Kunden ein Feedback-System konzipieren

### Prinzipien des Skills

1. **Privacy by Default** — Kein Screenshot ohne Vorschau, keine Systemdaten ohne Opt-out
2. **Modularität** — Jeder Feedback-Typ ist unabhängig aktivierbar
3. **Plattformneutralität** — Die Konzepte gelten für Web, Desktop und Mobile
4. **KI-Kompatibilität** — Alle Entscheidungen sind als Fragen formuliert, die ein Agent stellen kann

→ Weiter mit [wiki/02-Feedback-Hub.md](02-Feedback-Hub.md)

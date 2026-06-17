## Backend und Admin-Bereich

Ab Phase 2 des Projekts wird ein eigenes Backend empfohlen. Dieses ermöglicht eine strukturierte Verwaltung aller eingegangenen Reports, Ideen, Feedbacks und Hilfeanfragen über eine Admin-Oberfläche.

---

## Bugs — Admin-Bereich

### Listenansicht
- Alle eingegangenen Bug-Reports in chronologischer Reihenfolge
- **Suchfunktion:** Volltext-Suche über Titel und Beschreibung
- **Filter:** Schweregrad, Status, Datum (von/bis), Bearbeiter, App-Version

### Detailansicht pro Bug
- Titel und Beschreibung
- Schweregrad: `low` / `medium` / `high` / `critical`
- **Status:** Neu → In Bearbeitung → Behoben → Abgelehnt
- Bearbeiter zuweisen (aus dem Team)
- Interne Kommentare
- Angehängte Screenshots und Dateien anzeigen
- Technische Daten (OS, Gerät, App-Version)
- GitHub-Sync-Status: `Nicht verknüpft` / `Issue #123` / `Geschlossen`

### Aktionen
- Status ändern
- Priorität setzen
- GitHub Issue erstellen (manuell oder automatisch)
- Bug als Duplikat markieren und mit bestehendem verknüpfen

---

## Ideen — Admin-Bereich

### Listenansicht
- Alle eingegangenen Ideen mit Bewertung/Upvote-Anzahl
- **Suchfunktion** und **Filter:** Status, Datum, Kategorie

### Status-Workflow
`Neu` → `Geplant` → `Umgesetzt` → `Abgelehnt`

### Detailansicht
- Titel und Beschreibung der Idee
- Anzahl der Upvotes (falls Voting aktiviert)
- Status und Priorität
- Interne Kommentare
- GitHub-Sync-Status

### Aktionen
- Status ändern
- Idee in Roadmap aufnehmen
- Ähnliche Ideen zusammenführen

---

## Feedback — Admin-Bereich

### Listenansicht
- Allgemeine Rückmeldungen der Nutzer
- **Filter:** Status, Bewertung (Sterne), Datum

### Status-Workflow
`Neu` → `Gelesen` → `Archiviert`

### Detailansicht
- Feedback-Text
- Bewertung (z. B. 1–5 Sterne)
- Kontaktdaten des Nutzers (falls angegeben)
- Interne Kommentare

### Auswertung
- Durchschnittliche Bewertung über alle Feedbacks
- Bewertungsverteilung als Diagramm
- Exportfunktion (CSV)

---

## Hilfe-Anfragen — Admin-Bereich

### Listenansicht
- Alle offenen und bearbeiteten Hilfe-Anfragen
- **Filter:** Status, Dringlichkeit, Datum, Bearbeiter

### Status-Workflow
`Offen` → `In Bearbeitung` → `Geschlossen`

### Detailansicht
- Beschreibung des Problems
- Bearbeiter zuweisen
- **Antwort verfassen** (wird per E-Mail an den Nutzer geschickt, falls Kontaktdaten vorhanden)
- Gesprächsverlauf / Antwort-Thread

---

## Empfohlener Backend-Stack (Phase 2)

| Komponente | Option A | Option B | Option C |
|-----------|---------|---------|---------|
| **Sprache/Framework** | PHP / Laravel | Node.js / Express | Python / FastAPI |
| **Datenbank** | MySQL / MariaDB | PostgreSQL | SQLite (klein) |
| **Auth** | Laravel Breeze | JWT | Session-basiert |
| **Datei-Storage** | Lokales Filesystem | S3-kompatibel | Cloudflare R2 |

### Minimale API-Endpunkte

```
POST   /api/feedback/bug          → Bug-Report einreichen
POST   /api/feedback/idea         → Idee einreichen
POST   /api/feedback/general      → Allgemeines Feedback einreichen
POST   /api/feedback/help         → Hilfe anfordern
GET    /api/admin/bugs            → Bugs auflisten (Auth erforderlich)
PATCH  /api/admin/bugs/:id        → Bug-Status aktualisieren
DELETE /api/admin/bugs/:id        → Bug löschen
```

---

→ Weiter mit [wiki/12-GitHub-Integration.md](12-GitHub-Integration.md)

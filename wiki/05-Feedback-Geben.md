## Feedback geben

Das Feedback-Formular ist der niedrigschwelligste Feedback-Kanal. Es dient für freie Rückmeldungen aller Art — Lob, Kritik, Verbesserungsvorschläge oder Bedienungsprobleme, die sich nicht klar als Bug oder Idee einordnen lassen.

### Zweck

Freies Feedback fängt all das auf, was zwischen den Stühlen liegt:

- **Lob:** "Die neue Kalenderansicht ist viel übersichtlicher als vorher."
- **Kritik:** "Die Einstellungen sind sehr schwer zu finden."
- **Verbesserungswunsch:** "Der Onboarding-Prozess könnte kürzer sein."
- **Bedienungsproblem:** "Ich habe 10 Minuten gesucht, bis ich die Export-Funktion gefunden habe."

Bedienungsprobleme sind besonders wertvoll: Sie zeigen, wo die UX verbessert werden muss, ohne dass etwas technisch kaputt ist.

### Designprinzip: So einfach wie möglich

Das Feedback-Formular soll die geringste Hürde aller Formulare haben. Nur ein Pflichtfeld. Keine Kategorie-Zwangswahl. Kein Konto erforderlich.

**Faustregel:** Wenn der Nutzer in unter 30 Sekunden Feedback absenden kann, wird er es tun. Jedes zusätzliche Pflichtfeld halbiert die Wahrscheinlichkeit, dass das Formular abgesendet wird.

### Formularfelder

| Feld | Typ | Pflicht | Zeichenlimit | Hinweise |
|---|---|---|---|---|
| Freitext | Textarea | Ja | 1000 Zeichen | Die eigentliche Rückmeldung |
| Bewertung | 1–5 Sterne | Nein | — | Optionale Gesamtbewertung |
| Kategorie | Select | Nein | — | Lob / Kritik / Vorschlag / Sonstiges |

**Standardmäßig anonym:** Kein E-Mail-Feld. Wenn der Nutzer Kontakt wünscht, kann er eine Kontaktadresse am Ende des Freitexts eintragen — das ist bewusst nicht strukturiert.

### Platzhaltertext-Beispiele

Guter Platzhaltertext leitet ohne zu zwingen:

```
"Was gefällt dir? Was könnte besser sein? Wie war dein Erlebnis heute?"
```

Schlechter Platzhaltertext formuliert Erwartungen, die Nutzer hemmen:

```
"Bitte beschreibe dein Feedback detailliert und vollständig."
```

### JSON-Datenmodell

```json
{
  "type": "free_feedback",
  "id": "fb_20240315_c1d4",
  "timestamp": "2024-03-15T16:45:00Z",
  "form": {
    "message": "Die neue Kalenderansicht ist super übersichtlich. Danke! Nur die Schrift im Dark-Mode ist etwas schwer zu lesen.",
    "rating": 4,
    "category": "Lob"
  },
  "system": {
    "app_version": "2.1.3",
    "language": "de-DE",
    "route": "/calendar",
    "opt_out_system_data": false
  }
}
```

### Bewertungs-System (Sterne)

Die 1–5-Sterne-Bewertung ist optional und gibt dem Feedback eine quantitative Dimension. Über viele Einträge hinweg lässt sich daraus eine Gesamtzufriedenheit ableiten.

**Empfehlungen:**
- Sterne nicht als Pflichtfeld setzen — erzwungene Bewertungen verfälschen Ergebnisse
- Halbe Sterne unnötig — 5 Stufen reichen für Nutzerfeedback
- Labels unter den Sternen helfen bei der Einordnung: 1 = Sehr schlecht, 5 = Ausgezeichnet
- Im Dashboard: Durchschnittswert über Zeit darstellen, um Trends zu erkennen

### Kategorien

| Kategorie | Beschreibung |
|---|---|
| Lob | Positives Feedback, Bestätigung |
| Kritik | Negatives Feedback, Unzufriedenheit |
| Vorschlag | Idee für Verbesserung (ohne konkrete Feature-Request-Struktur) |
| Sonstiges | Passt in keine andere Kategorie |

Die Kategorie ist optional. Wenn sie genutzt wird, ermöglicht sie eine schnelle Filterung im Dashboard — z. B. "Alle kritischen Feedbacks dieser Woche".

### Verarbeitung im Dashboard

Freies Feedback ist oft unstrukturiert. Empfehlungen für die Dashboard-Seite:

- **Sentiment-Analyse** (optional): Automatische Einstufung als positiv/negativ/neutral
- **Tagging** (manuell): Entwickler können Einträge mit Schlagwörtern versehen
- **Status**: Ungelesen / Gelesen / Archiviert

Freies Feedback sollte **nicht** als ToDo betrachtet werden — es muss nicht immer beantwortet werden. Es dient als Stimmungsbild und Inspirationsquelle.

### UX-Tipps

- Formular beim Öffnen direkt mit Fokus auf das Textarea starten
- Submit-Button erst aktivieren, wenn mindestens 10 Zeichen eingetippt wurden
- Nach dem Absenden: kurze Bestätigung, kein langer Dialog
- Empfohlene Bestätigungstext: "Danke für dein Feedback!" — nicht länger

→ Weiter mit [wiki/06-Hilfe-Anfordern.md](06-Hilfe-Anfordern.md)

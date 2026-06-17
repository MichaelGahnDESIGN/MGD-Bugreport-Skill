## Ideen einreichen

Das Ideen-Formular gibt Nutzern die Möglichkeit, Verbesserungsvorschläge und Feature-Wünsche strukturiert einzureichen. Im Gegensatz zu freiem Feedback hat eine Idee immer eine klare Richtung: der Nutzer wünscht sich eine konkrete Änderung oder Erweiterung.

### Zweck und Abgrenzung

Eine Idee ist kein Bug. Der Unterschied:
- **Bug:** "Der Export-Button funktioniert nicht." (Etwas ist kaputt)
- **Idee:** "Es wäre toll, wenn man auch als CSV exportieren könnte." (Etwas fehlt oder könnte besser sein)

Das Formular hilft Entwicklern, Feature-Requests von Fehlermeldungen zu trennen und priorisiert zu bearbeiten.

### Formularfelder

| Feld | Typ | Pflicht | Zeichenlimit | Hinweise |
|---|---|---|---|---|
| Titel | Text | Ja | 80 Zeichen | Kurze, klare Zusammenfassung der Idee |
| Beschreibung | Textarea | Ja | 500 Zeichen | Warum wäre das nützlich? Für wen? |
| Kategorie | Select | Nein | — | UI, Performance, Feature, Sonstiges |
| Priorität | Select | Nein | — | niedrig / mittel / hoch |
| Screenshot / Mockup | Datei-Upload | Nein | 5 MB | Skizze oder Referenz-Screenshot |

**Zeichenzähler:** Titel und Beschreibung sollten einen Live-Zeichenzähler anzeigen:
```
Titel: [_________________________________] 42/80
```

Ein Zeichenlimit für den Titel (80) zwingt Nutzer zu einer präzisen Formulierung. Lange, vage Titel wie "Alles verbessern und die ganze App neu designen" liefern keinen Mehrwert.

### Kategorie-System

| Kategorie | Beschreibung | Beispiel |
|---|---|---|
| UI | Visuelle oder Interaktions-Verbesserung | "Dunkel-Modus hinzufügen" |
| Performance | Geschwindigkeit, Ressourcenverbrauch | "Ladezeit der Startseite reduzieren" |
| Feature | Neue Funktionalität | "PDF-Export ermöglichen" |
| Sonstiges | Passt in keine andere Kategorie | "Dokumentation verbessern" |

Die Kategorie ist optional, hilft aber beim Filtern und Priorisieren im Dashboard. Wenn nur wenige Ideen zu erwarten sind, kann die Kategorie weggelassen werden.

### Priorität

Die Priorität ist eine **Selbsteinschätzung des Nutzers**, keine verbindliche Aussage für den Entwickler:

| Wert | Bedeutung |
|---|---|
| niedrig | Wäre nett, aber nicht dringend |
| mittel | Würde meine Arbeit spürbar verbessern |
| hoch | Ich brauche das dringend |

Der Entwickler kann die Priorität im Dashboard überschreiben.

### JSON-Datenmodell

```json
{
  "type": "idea_submission",
  "id": "idea_20240315_b7c2",
  "timestamp": "2024-03-15T09:15:00Z",
  "form": {
    "title": "CSV-Export für Berichte",
    "description": "Aktuell kann man Berichte nur als PDF exportieren. Ein CSV-Export würde die Weiterverarbeitung in Excel deutlich erleichtern.",
    "category": "Feature",
    "priority": "hoch"
  },
  "attachments": {
    "screenshot": null,
    "files": []
  },
  "votes": {
    "count": 0,
    "user_voted": false
  },
  "system": {
    "app_version": "2.1.3",
    "language": "de-DE"
  }
}
```

### Abstimmungs-System (optional)

Wenn mehrere Nutzer die gleiche Idee einreichen, können sie die Idee anderer **upvoten** statt einen Duplikat-Report zu erstellen. Das Abstimmungs-System ist optional und erfordert:

- Eine Deduplizierungslogik (ähnliche Ideen zusammenführen)
- Eine Nutzeridentifikation (anonym via Session-ID oder mit Account)
- Ein Backend, das Stimmen speichert

**Empfehlung:** Das Abstimmungs-System erst nach der ersten Produktivphase hinzufügen. In der Frühphase ist die Menge der Ideen überschaubar; Deduplizierung kann manuell erfolgen.

### UX-Tipps

- Platzhaltertext Titel: *"Fasse deine Idee in einem Satz zusammen"*
- Platzhaltertext Beschreibung: *"Beschreibe, welches Problem diese Idee löst und für wen sie besonders nützlich wäre"*
- Zeige nach dem Absenden: "Danke für deine Idee! Wir prüfen alle Vorschläge regelmäßig."
- Vermeide das Wort "Feature-Request" im UI — Nutzer fühlen sich damit wohler, wenn sie eine "Idee einreichen" statt einen "Feature-Request stellen"
- Live-Vorschau der Zeichenanzahl verhindern, dass Nutzer die Beschreibung abbrechen lassen müssen

→ Weiter mit [wiki/05-Feedback-Geben.md](05-Feedback-Geben.md)

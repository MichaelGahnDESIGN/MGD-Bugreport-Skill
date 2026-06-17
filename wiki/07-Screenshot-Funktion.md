## Screenshot-Funktion

Screenshots sind das wertvollste Element eines Bug-Reports — sie zeigen auf einen Blick, was der Nutzer sieht. Gleichzeitig sind sie datenschutzrechtlich das sensibelste Element. Die Implementierung hängt stark von der Zielplattform ab.

### Wichtig: Rückfrage vor der Implementierung

Ein KI-Agent, der den Screenshot-Teil implementieren soll, **muss zuerst folgende Fragen stellen**:

1. Welche Plattform(en) soll die App unterstützen?
2. Soll der Screenshot automatisch aufgenommen werden oder nur auf expliziten Nutzerwunsch?
3. Muss der Nutzer eine Vorschau sehen und bestätigen, bevor der Screenshot gesendet wird?
4. Soll der Nutzer Teile des Screenshots unkenntlich machen können?
5. Reicht ein Screenshot der eigenen App, oder soll auch der Desktop erfasst werden?

Ohne diese Antworten ist keine korrekte Implementierung möglich — die verfügbaren APIs unterscheiden sich grundlegend.

### Plattform-Übersicht

| Plattform | Verfügbare API | Einschränkungen | Besonderheiten |
|---|---|---|---|
| macOS | `NSScreen`, `CGWindowListCreateImage` | Keine Systemeinschränkungen | Ab macOS 12.3: Bildschirmaufnahme-Berechtigung erforderlich |
| Windows | `PrintScreen` / GDI+ / `BitBlt` | Keine Systemeinschränkungen | DRM-geschützte Inhalte werden schwarz dargestellt |
| Linux | X11 (`XGetImage`) / Wayland (`wlr-screencopy`) | Stark abhängig vom Compositor | Wayland: nicht alle Compositor unterstützen screencopy |
| iOS | `UIGraphicsGetCurrentContext` | Nur die eigene App, kein System-UI | Keine Darstellung anderer Apps möglich |
| Android | `MediaProjection` | Erfordert explizite Nutzererlaubnis | Dialog erscheint systemseitig, nicht unterdrückbar |
| Web | `html2canvas` / `dom-to-image` | Nur die eigene Seite, kein Browser-UI | Cross-Origin-Inhalte werden nicht erfasst |
| Unity | `ScreenCapture.CaptureScreenshot` / `RenderTexture` | Nur das eigene Spielfenster | Einfach implementierbar |
| Godot | `Viewport.get_texture()` | Nur das eigene Fenster | Als `Image` exportierbar |

### Screenshot-Umfang: Drei Optionen

| Option | Beschreibung | Empfehlung |
|---|---|---|
| Vollbild | Gesamter Bildschirm inkl. anderer Apps | Nur wenn explizit gewünscht, mit Warnung |
| App-only | Nur das eigene App-Fenster | Standard-Empfehlung |
| Benutzerauswahl | Nutzer zieht einen Rahmen um den relevanten Bereich | Ideal für komplexe UIs |

**Empfehlung:** App-only als Standard. Vollbild nur auf expliziten Nutzerwunsch mit deutlichem Hinweis: "Ein Vollbild-Screenshot kann andere Anwendungen und persönliche Daten enthalten."

### Pflichtschritt: Vorschau vor dem Senden

Kein Screenshot darf ohne vorherige Nutzerbestätigung gesendet werden. Der Vorschau-Dialog muss:

1. Den Screenshot vollständig anzeigen (zoombar)
2. Einen "Senden"-Button haben
3. Einen "Verwerfen"-Button haben
4. Optional: Schwärzungs-/Unkenntlichmachungs-Werkzeug

```
┌─────────────────────────────────────┐
│  Screenshot-Vorschau                │
│  ┌─────────────────────────────┐    │
│  │  [Screenshot-Inhalt]        │    │
│  └─────────────────────────────┘    │
│                                     │
│  ⚠ Prüfe den Screenshot auf        │
│    persönliche Daten, bevor         │
│    du ihn sendest.                  │
│                                     │
│  [✏ Schwärzen]  [Verwerfen]  [✓ Senden] │
└─────────────────────────────────────┘
```

### Annotations-Werkzeuge (zukünftig)

Für eine spätere Ausbaustufe empfohlene Werkzeuge:

- **Schwärzen (Redact):** Schwarzer Balken über sensiblen Bereichen
- **Pfeil / Markierung:** Nutzer kann den relevanten Bereich markieren
- **Freihandzeichnung:** Einfache Annotation direkt auf dem Screenshot
- **Text-Label:** Kurze Beschriftung ("Hier ist der Fehler")

Diese Werkzeuge sind für die erste Version nicht notwendig, verbessern aber die Qualität der Bug-Reports erheblich.

### Datenschutz-Warnungen

Auf allen Plattformen gilt: **Immer warnen, bevor ein Screenshot aufgenommen wird.**

Empfohlener Hinweistext:
> "Screenshots können persönliche Daten enthalten. Bitte prüfe die Vorschau, bevor du den Screenshot sendest."

Zusätzliche Plattform-spezifische Hinweise:

- **macOS/Windows:** "Der Screenshot erfasst das gesamte App-Fenster, einschließlich aller geöffneten Daten."
- **Android:** "Du wirst von Android nach der Berechtigung gefragt, bevor der Screenshot erstellt wird."
- **Web:** "Der Screenshot erfasst nur diese Seite — nicht andere Browser-Tabs oder Fenster."

### Opt-out

Nutzer müssen Screenshots vollständig ablehnen können:

```
○ Screenshot automatisch hinzufügen
● Kein Screenshot (ich teile manuell, wenn gewünscht)
```

Wenn Opt-out gewählt wird, wird kein Screenshot aufgenommen — auch nicht im Hintergrund.

### Dateiformat und Komprimierung

- Format: PNG für Schärfe, JPEG (80% Qualität) für Dateigröße-Optimierung
- Maximale Dateigröße vor Upload: 5 MB
- Bei Überschreitung: automatische Komprimierung oder Hinweis an Nutzer
- Dateiname: keine identifizierbaren Informationen (z. B. `screenshot_a3f9.png`, nicht `Max_Muster_Desktop.png`)

→ Weiter mit [wiki/08-Technische-Daten.md](08-Technische-Daten.md)

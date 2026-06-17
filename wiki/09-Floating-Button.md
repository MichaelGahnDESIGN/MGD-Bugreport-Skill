## Der Floating Button

Der Floating Button ist der zentrale Einstiegspunkt für den Feedback-Hub. Er schwebt dauerhaft über der Anwendungsoberfläche und ist jederzeit erreichbar — unabhängig vom aktuellen Screen oder der Navigation.

### Standardempfehlung

- **Farbe:** Grün (signalisiert Hilfsbereitschaft, kein Alarm)
- **Symbol:** 🐞 (Käfer/Bug-Symbol) oder alternativ ein Sprechblasen-Icon
- **Position:** Unten rechts (Standard)
- **Z-Index:** Höchster Wert im gesamten UI — niemals von anderen Elementen überlagert
- **Verhalten:** Immer sichtbar, unabhängig von Navigation und Modals

---

### Fragen, die der Agent stellen MUSS

Bevor der Floating Button implementiert wird, muss der Agent folgende Fragen klären:

1. Soll der Button **immer sichtbar** sein oder nur im Debug-Modus?
2. Soll der Button nur für **bestimmte Nutzer** (Tester, Admins) erscheinen?
3. **Position:** Unten rechts (Standard) oder konfigurierbar?
4. Soll der Button **minimierbar** sein (z. B. als kleines Tab am Rand)?
5. Soll es einen **Keyboard-Shortcut** als Alternative geben?

---

### Verhalten je nach Kontext

Der Button sollte sich je nach Betriebsmodus unterschiedlich verhalten:

| Modus | Sichtbarkeit | Zielgruppe |
|-------|-------------|------------|
| **Debug-Modus** | Immer sichtbar, volle Größe | Entwickler und Tester |
| **Tester-Modus** | Nur sichtbar wenn Flag/Setting aktiv | Ausgewählte Tester |
| **Produktions-Modus** | Standardmäßig ausgeblendet, aktivierbar über Einstellungen | Alle Nutzer optional |

---

### Positionsoptionen

| Option | Beschreibung | Empfehlung |
|--------|-------------|------------|
| **Unten rechts** | Standard, weit verbreitet | ✅ Empfohlen |
| **Unten links** | Alternative für Apps mit rechter Navigation | Gut |
| **Oben rechts** | Seltener, aber möglich | Nur wenn notwendig |
| **Frei positionierbar** | Nutzer kann Position selbst wählen | Power-User-Feature |

---

### Implementierungshinweise

Der Button muss:

- **Außerhalb des normalen Widget-Baums** platziert werden (z. B. als Overlay oder Stack-Element auf Root-Ebene)
- **Nicht von Tastatur-Overlays** überdeckt werden (wichtig auf Mobile)
- **Safe Areas** respektieren (iOS Notch, Android Navigation Bar)
- Bei Modals und Dialogen **weiterhin erreichbar** bleiben

```dart
// Flutter-Beispiel: Floating Button als Overlay
Overlay.of(context)?.insert(
  OverlayEntry(
    builder: (context) => Positioned(
      bottom: 80,
      right: 16,
      child: FeedbackHubButton(),
    ),
  ),
);
```

---

### Barrierefreiheit

Neben dem visuellen Button sollte immer ein **Keyboard-Shortcut** verfügbar sein:

- **Standard-Shortcut:** `Ctrl + Shift + F` (für Feedback)
- Auf macOS: `Cmd + Shift + F`
- Der Shortcut sollte in den Einstellungen anpassbar sein

```json
{
  "feedbackShortcut": {
    "windows": "Ctrl+Shift+F",
    "macos": "Cmd+Shift+F",
    "linux": "Ctrl+Shift+F"
  }
}
```

Der Button muss zudem:

- Ein aussagekräftiges **Accessibility-Label** haben (`"Feedback melden"`)
- Per **Screen Reader** erreichbar sein
- Einen ausreichenden **Kontrast** zum Hintergrund aufweisen (WCAG 2.1 AA)

---

→ Weiter mit [wiki/10-Einstellungen.md](10-Einstellungen.md)

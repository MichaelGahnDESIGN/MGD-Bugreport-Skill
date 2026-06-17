## Einstellungen-Integration

Der Feedback-Hub sollte nahtlos in die Einstellungen der Anwendung integriert werden. Nutzer erhalten so die Kontrolle darüber, welche Funktionen aktiv sind und welche Daten gesendet werden.

### Empfohlene Einstellungskategorie

Erstelle eine eigene Kategorie in den App-Einstellungen:

- **Empfohlener Name:** `Debug & Ideen` oder `Feedback`
- **Icon:** 🐞 oder Sprechblasen-Symbol
- **Platzierung:** Am Ende der Einstellungsliste (nicht unter „Allgemein")

---

### Einstellungsoptionen im Überblick

| Einstellung | Typ | Standard | Beschreibung |
|-------------|-----|---------|--------------|
| **Feedback-Hub anzeigen** | Toggle | `true` | Blendet den Floating Button ein oder aus |
| **Fehler melden aktivieren** | Toggle | `true` | Aktiviert den Bug-Report-Eintrag im Hub |
| **Ideen-Funktion aktivieren** | Toggle | `true` | Aktiviert den Ideen-Eintrag im Hub |
| **Feedback-Funktion aktivieren** | Toggle | `true` | Aktiviert den allgemeinen Feedback-Eintrag |
| **Hilfe anfordern aktivieren** | Toggle | `true` | Aktiviert den Hilfe-Eintrag im Hub |
| **Technische Daten mitsenden** | Toggle | `true` | Sendet Gerät, OS, App-Version mit dem Report |
| **Screenshots automatisch erstellen** | Toggle | `false` | Erstellt beim Öffnen des Hubs automatisch einen Screenshot |
| **Kontaktdaten speichern** | Toggle | `false` | Merkt sich die E-Mail-Adresse für zukünftige Reports |

---

### Erweiterte Einstellungen (optional)

Für fortgeschrittene Nutzer oder Self-Hosted-Setups:

| Einstellung | Typ | Beschreibung |
|-------------|-----|--------------|
| **API-Endpunkt** | Texteingabe | Eigener Backend-Endpunkt (z. B. `https://api.example.com/feedback`) |
| **Debug-Modus** | Toggle | Zeigt zusätzliche Diagnoseinformationen im Hub an |
| **Gespeicherte Kontaktdaten löschen** | Aktion | Löscht die gespeicherte E-Mail-Adresse sofort |

---

### Einstellungsmodell (Code-Beispiel)

```json
{
  "feedbackHub": {
    "enabled": true,
    "modules": {
      "bugReport": true,
      "ideas": true,
      "feedback": true,
      "helpRequest": true
    },
    "privacy": {
      "sendTechnicalData": true,
      "autoScreenshot": false,
      "saveContactInfo": false
    },
    "advanced": {
      "apiEndpoint": "https://api.example.com/feedback",
      "debugMode": false
    }
  }
}
```

```dart
// Dart/Flutter-Beispiel: Settings-Klasse
class FeedbackHubSettings {
  bool enabled;
  bool bugReportEnabled;
  bool ideasEnabled;
  bool feedbackEnabled;
  bool helpRequestEnabled;
  bool sendTechnicalData;
  bool autoScreenshot;
  bool saveContactInfo;
  String? apiEndpoint;
  bool debugMode;

  FeedbackHubSettings({
    this.enabled = true,
    this.bugReportEnabled = true,
    this.ideasEnabled = true,
    this.feedbackEnabled = true,
    this.helpRequestEnabled = true,
    this.sendTechnicalData = true,
    this.autoScreenshot = false,
    this.saveContactInfo = false,
    this.apiEndpoint,
    this.debugMode = false,
  });

  factory FeedbackHubSettings.fromJson(Map<String, dynamic> json) {
    // Deserialisierung hier implementieren
    return FeedbackHubSettings();
  }
}
```

---

### Hinweise zur Implementierung

- Einstellungen sollten **persistent gespeichert** werden (z. B. `SharedPreferences`, `UserDefaults`, `localStorage`)
- Änderungen müssen **sofort wirksam** sein — kein App-Neustart erforderlich
- Der API-Endpunkt sollte eine **Validierung** durchlaufen (erreichbar? gültige URL?)
- Bei deaktiviertem Floating Button sollte der Keyboard-Shortcut weiterhin funktionieren

---

→ Weiter mit [wiki/11-Backend-und-Admin.md](11-Backend-und-Admin.md)

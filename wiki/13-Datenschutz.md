## Datenschutz und DSGVO

Datenschutz ist kein optionales Feature — er ist ein gesetzliches Erfordernis. Der Feedback-Hub verarbeitet personenbezogene Daten und muss daher DSGVO-konform implementiert werden.

---

### Die 6 Grundregeln

**1. Einwilligung (Art. 6 DSGVO)**
Nutzer müssen aktiv zustimmen, bevor persönliche Daten (Name, E-Mail, Screenshots) gesendet werden. Eine vorausgefüllte Checkbox oder implizite Zustimmung ist nicht ausreichend.

**2. Transparenz (Art. 13 DSGVO)**
Beim Öffnen des Feedback-Formulars muss klar angezeigt werden, welche Daten gesendet werden. Ein Link zur Datenschutzerklärung ist Pflicht.

**3. Opt-out**
Technische Daten (Gerät, OS, App-Version) und automatische Screenshots müssen einzeln abwählbar sein. Kein Senden von Daten ohne ausdrückliche Aktivierung durch den Nutzer.

**4. Datensparsamkeit (Art. 5 DSGVO)**
Es dürfen nur Daten erfasst werden, die für den jeweiligen Report tatsächlich notwendig sind. Keine versteckten Tracking-Daten, keine unnötigen Metadaten.

**5. Verschlüsselung**
- Alle Übertragungen ausschließlich über **HTTPS**
- Screenshots werden **verschlüsselt** gespeichert (AES-256 empfohlen)
- Zugriff auf gespeicherte Reports nur für autorisierte Personen

**6. Löschrecht (Art. 17 DSGVO)**
Nutzer haben das Recht, ihre Daten löschen zu lassen. Der Admin-Bereich muss eine Löschfunktion bereitstellen. Bei Anfrage per E-Mail: Löschung innerhalb von 30 Tagen.

---

### DSGVO-Checkliste

| Was prüfen | Wie umsetzen |
|-----------|-------------|
| Einwilligung vor Datensendung | Checkbox im Formular, nicht vorausgefüllt |
| Datenschutzlink im Formular | Link zu `feedback.example.com/datenschutz` |
| Opt-out für technische Daten | Toggle in Formular und in Einstellungen |
| Opt-out für Screenshots | Toggle in Formular und in Einstellungen |
| Keine Daten ohne Zustimmung | Validierung im Backend vor Speicherung |
| HTTPS erzwingen | HSTS-Header, HTTP-Redirect auf HTTPS |
| Verschlüsselte Screenshot-Speicherung | Verschlüsselt im Dateisystem oder Object Storage |
| Löschfunktion im Admin | Einzeln und als Bulk-Löschung |
| Datenschutzerklärung aktuell | Explizit Feedback-Hub erwähnen |
| Auftragsverarbeitungsvertrag | Mit Hosting-Anbieter abschließen |

---

### Screenshot-Datenschutz

Bevor ein Screenshot gesendet wird, müssen sensible Felder **unkenntlich gemacht** werden:

- Passwort-Felder → komplett schwarz/geschwärzt
- Kreditkartennummern → geschwärzt
- IBAN / Kontodaten → geschwärzt
- Persönliche Nachrichten (Chat) → optional, je nach App

```dart
// Pseudocode: Sensitive Felder vor Screenshot ausblenden
void prepareScreenshotSafety() {
  for (var field in sensitiveFields) {
    field.isObscured = true; // Passwort-Modus
    // oder: field.visibility = Visibility.hidden;
  }
}
```

---

### Aufbewahrungsfristen

| Datentyp | Empfohlene Aufbewahrungsdauer |
|----------|------------------------------|
| Bug-Reports | Max. 2 Jahre, dann automatisch löschen |
| Ideen | Unbegrenzt (kein Personenbezug) |
| Allgemeines Feedback | Max. 1 Jahr |
| Hilfe-Anfragen | Max. 2 Jahre |
| Screenshots | Max. 1 Jahr oder nach Bearbeitung löschen |
| E-Mail-Adressen | Nur solange notwendig, bei Löschanfrage sofort |

---

### iOS Privacy Manifest (seit iOS 17 Pflicht)

Apple verlangt seit iOS 17 eine `PrivacyInfo.xcprivacy`-Datei für alle Apps, die bestimmte APIs nutzen. Falls der Feedback-Hub Screenshots erstellt oder Gerätedaten sammelt:

```xml
<!-- PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
  <key>NSPrivacyCollectedDataTypes</key>
  <array>
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypeDeviceID</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <false/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
      </array>
    </dict>
  </array>
</dict>
</plist>
```

---

### Datenschutzerklärung

Die App-Datenschutzerklärung muss folgenden Abschnitt enthalten:

- Was der Feedback-Hub erhebt (technische Daten, Screenshots, Kontaktdaten)
- Wer Zugriff auf die Daten hat (Entwicklungsteam)
- Wie lange Daten gespeichert werden
- Wie Nutzer Löschanfragen stellen können
- Ein Link auf die Erklärung muss direkt im Feedback-Formular sichtbar sein

---

→ Weiter mit [wiki/14-Plattformen.md](14-Plattformen.md)

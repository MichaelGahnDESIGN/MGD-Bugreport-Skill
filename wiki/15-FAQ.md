## Häufig gestellte Fragen (FAQ)

---

### 1. Kann ich den Feedback-Hub für interne Apps (ohne App Store) nutzen?

**Ja, ohne Einschränkungen.** Der Feedback-Hub funktioniert für Enterprise-Apps, interne Tools und TestFlight-/Beta-Builds genauso wie für öffentliche Apps. Bei internen Apps entfallen sogar viele App-Store-Richtlinien (z. B. Apples §5.1 für den App Store).

---

### 2. Muss ich alle vier Hub-Einträge aktivieren?

**Nein.** Jeder Eintrag — Bug melden, Idee einreichen, Feedback geben, Hilfe anfordern — ist optional und kann einzeln über die Einstellungen deaktiviert werden. Es ist auch möglich, nur einen einzigen Eintrag zu aktivieren (z. B. nur „Bug melden").

---

### 3. Welche Screenshot-Methode ist für Web-Apps empfohlen?

**html2canvas** ist der bewährte Standard für Web-Apps. Alternativen:
- `dom-to-image` (ähnlich, andere Implementierung)
- Native `getDisplayMedia` API (erfordert Nutzer-Erlaubnis, erfasst den gesamten Bildschirm)

html2canvas erfasst den DOM-Zustand und hat keine Berechtigung-Anfrage, ist aber auf CORS-Ressourcen beschränkt.

---

### 4. Was wenn der Nutzer keinen Screenshot machen will?

Screenshots müssen immer **optional** sein. Das Formular muss ohne Screenshot absendbar sein. Empfehlung:
- Standard: Screenshot automatisch erstellen, aber Opt-out anbieten
- Alternativ: Screenshot nur auf explizite Anfrage des Nutzers

Niemals einen Screenshot ohne Wissen des Nutzers senden.

---

### 5. Brauche ich ein eigenes Backend?

- **Phase 1 (Einfachstart):** Nein. Ein E-Mail-basierter Ansatz reicht aus. Reports werden per E-Mail an eine Adresse gesendet.
- **Phase 2 (Strukturiert):** Ein eigenes Backend wird empfohlen. Es ermöglicht Admin-Oberfläche, GitHub-Sync, Status-Tracking und Datenschutz-Kontrolle.
- **Phase 3 (Skaliert):** Backend mit Datenbank, Webhook-Support und erweiterten Analyse-Funktionen.

---

### 6. Kann ich GitHub direkt vom Client aus nutzen (ohne eigenes Backend)?

**Nein — das ist ein Sicherheitsrisiko.** Der GitHub-Token würde im Client-Code sichtbar sein und könnte missbraucht werden. Die Lösung:
- Client → eigenes Backend (`api.example.com`) → GitHub API
- Das Backend agiert als sicherer Proxy
- Der Token bleibt ausschließlich auf dem Server

---

### 7. Was passiert wenn kein Internet vorhanden ist?

Reports sollten **lokal zwischengespeichert** werden und beim nächsten erfolgreichen Verbindungsaufbau automatisch gesendet werden.

```json
// Lokaler Cache (Beispiel)
{
  "pendingReports": [
    {
      "type": "bug",
      "title": "Absturz beim Laden",
      "createdAt": "2026-06-17T10:30:00Z",
      "retryCount": 0
    }
  ]
}
```

Dem Nutzer sollte angezeigt werden: „Dein Report wird gesendet, sobald eine Verbindung verfügbar ist."

---

### 8. Wie groß darf ein Screenshot sein?

**Empfehlung: max. 5 MB** pro Screenshot. Vor dem Senden komprimieren:
- Format: JPEG statt PNG (deutlich kleinere Dateigröße)
- Qualität: 70–80 % ist in der Regel ausreichend
- Skalierung: Auf max. 1920 × 1080 px reduzieren, wenn höher

```dart
// Komprimierung in Flutter
final compressedBytes = await FlutterImageCompress.compressWithList(
  imageBytes,
  quality: 75,
  format: CompressFormat.jpeg,
);
```

---

### 9. Soll der Nutzer eine Bestätigung erhalten?

**Empfehlung: Ja**, wenn der Nutzer eine Kontaktadresse angegeben hat. Eine kurze Bestätigungs-E-Mail mit:
- Zusammenfassung des Reports
- Einer Referenznummer
- Dem Hinweis, dass das Team sich meldet (nur falls realistisch)

Ohne Kontaktdaten: Bestätigung im UI direkt nach dem Absenden anzeigen.

---

### 10. Wie vermeide ich Spam-Einreichungen?

Mehrere Maßnahmen kombinieren:

| Maßnahme | Beschreibung |
|---------|-------------|
| **Rate-Limiting** | Max. 5 Reports pro IP-Adresse pro Stunde |
| **CAPTCHA** | Optional, z. B. hCaptcha (datenschutzfreundlicher als reCAPTCHA) |
| **IP-basierte Limits** | Temporäre Sperrung nach zu vielen Reports |
| **Honeypot-Felder** | Unsichtbares Formularfeld — wenn ausgefüllt, ist es ein Bot |
| **Mindestlänge** | Beschreibung muss mindestens 20 Zeichen haben |

---

### 11. Wie unterscheidet sich allgemeines Feedback von einem Bug-Report?

| Kriterium | Bug-Report | Feedback |
|-----------|-----------|---------|
| **Zweck** | Etwas funktioniert nicht | Meinung, Wunsch, Bewertung |
| **Technische Daten** | Immer sinnvoll | Optional |
| **Screenshot** | Sehr hilfreich | Selten notwendig |
| **Schweregrad** | Wird angegeben | Nicht relevant |
| **Erwartete Antwort** | Status-Update erwartet | Oft keine Antwort nötig |
| **Beispiel** | „Der Login-Button reagiert nicht" | „Die App macht Spaß, aber ich wünsche mir Dunkel-Modus" |

---

### 12. Kann ich den Button nur für bestimmte Nutzer anzeigen?

**Ja.** Es gibt mehrere Ansätze:

- **Feature-Flag:** `if (featureFlags.feedbackHubEnabled) { ... }`
- **User-Role-Check:** Nur für Nutzer mit Rolle `tester` oder `admin`
- **Build-Konfiguration:** Nur in Debug- oder Staging-Builds aktiviert
- **Remote-Config:** Über Firebase Remote Config oder ähnliche Dienste steuern

```dart
// Beispiel: Nur für Tester anzeigen
if (currentUser.hasRole('tester') || kDebugMode) {
  showFeedbackHubButton();
}
```

---

→ Weiter mit [wiki/16-Glossar.md](16-Glossar.md)

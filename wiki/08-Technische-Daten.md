## Technische Daten

Technische Daten sind der unsichtbare Teil eines Bug-Reports — automatisch erfasst, ohne dass der Nutzer etwas eintippen muss. Sie helfen Entwicklern, Fehler schnell einzugrenzen und zu reproduzieren. Gleichzeitig können sie persönliche Daten enthalten und unterliegen der DSGVO.

### Desktop-Daten

Für native Desktop-Anwendungen (macOS, Windows, Linux):

| Datenpunkt | Beispielwert | DSGVO-Relevanz | Quelle |
|---|---|---|---|
| Betriebssystem | macOS | Niedrig | `platform.system()` / `os.platform()` |
| OS-Version | 14.5 | Niedrig | `platform.mac_ver()` / `os.release()` |
| Architektur | arm64 / x64 | Niedrig | `platform.machine()` |
| Bildschirmauflösung | 2560×1600 | Mittel | Screen-API der jeweiligen Plattform |
| Sprache | de-DE | Niedrig | Systemsprache / App-Locale |
| Zeitzone | Europe/Berlin | Mittel | System-Zeitzone |
| App-Version | 2.1.3 | Keine | App-Manifest / Versionsdatei |
| Prozessor | Apple M2 | Mittel | `platform.processor()` |
| RAM gesamt (optional) | 16 GB | Mittel | Systeminformationen |

**Hinweis RAM:** RAM-Informationen sind hilfreich bei Performance-Bugs, aber optional. Bei vielen Nutzern in Kombination mit anderen Daten können sie zur Identifizierung beitragen.

### Mobile-Daten

Für iOS- und Android-Anwendungen:

| Datenpunkt | Beispielwert | DSGVO-Relevanz | Quelle |
|---|---|---|---|
| Gerät (Hersteller + Modell) | Apple iPhone 15 Pro | Mittel | `UIDevice.current.model` / `Build.MODEL` |
| OS-Version | iOS 17.4 | Niedrig | `UIDevice.current.systemVersion` / `Build.VERSION.RELEASE` |
| Sprache | de-DE | Niedrig | `Locale.current` / `Locale.getDefault()` |
| Bildschirmgröße | 393×852 pt | Niedrig | `UIScreen.main.bounds` / `DisplayMetrics` |
| App-Version | 2.1.3 | Keine | `CFBundleShortVersionString` / `versionName` |
| Build-Nummer | 42 | Keine | `CFBundleVersion` / `versionCode` |

**Kein Geräte-Identifier:** IMEI, IDFA, Android Advertising ID oder ähnliche eindeutige Gerätekennungen dürfen **nicht** erfasst werden, es sei denn, der Nutzer hat explizit eingewilligt und es gibt einen klar definierten Zweck.

### Web-Daten

Für Browser-basierte Anwendungen:

| Datenpunkt | Beispielwert | DSGVO-Relevanz | Quelle |
|---|---|---|---|
| Browser | Chrome | Niedrig | `navigator.userAgent` (geparst) |
| Browser-Version | 123.0.6312 | Niedrig | `navigator.userAgent` (geparst) |
| URL | /dashboard/settings | Niedrig–Mittel | `window.location.pathname` |
| Bildschirmgröße | 2560×1440 | Niedrig | `window.screen.width/height` |
| Viewport-Größe | 1440×900 | Niedrig | `window.innerWidth/innerHeight` |
| Sprache | de-DE | Niedrig | `navigator.language` |
| Zeitzone | Europe/Berlin | Mittel | `Intl.DateTimeFormat().resolvedOptions().timeZone` |
| Referrer | /dashboard | Niedrig | `document.referrer` |

**Vorsicht URL:** Die vollständige URL kann IDs, Token oder Suchbegriffe enthalten (z. B. `/search?q=Mustermann`). Empfehlung: Nur den Pfad erfassen, keine Query-Parameter — oder Query-Parameter gezielt filtern.

### Kontextdaten (alle Plattformen)

Zusätzlich zu plattformspezifischen Daten können folgende Kontextdaten erfasst werden:

| Datenpunkt | Beispielwert | Zweck |
|---|---|---|
| Aktuelle Route / Seite / Ansicht | `/settings/profile` | Lokalisierung des Fehlers |
| Zeitpunkt (UTC) | `2024-03-15T14:32:00Z` | Zeitliche Eingrenzung |
| Session-Dauer | 342 Sekunden | Kontextinfo (war der Nutzer gerade erst gestartet?) |
| Vorherige Seite | `/dashboard` | Navigation vor dem Fehler |

Session-Dauer und Navigationspfad können bei der Fehlerrekonstruktion helfen, sind aber datenschutzsensibel — opt-out ermöglichen.

### Datenschutz: Opt-out Pflicht

Der Nutzer muss technische Daten vollständig ablehnen können. Empfohlene UI-Umsetzung:

```
☑ Technische Daten mitsenden
  Hilft uns, den Fehler schneller zu finden.
  [Details anzeigen ▾]
    • Betriebssystem: macOS 14.5
    • App-Version: 2.1.3
    • Aktuelle Seite: /settings/profile
    • Sprache: de-DE
```

Wenn der Nutzer den Haken entfernt, wird das `system`-Objekt aus dem Payload entfernt. Mindestens die App-Version sollte immer gesendet werden dürfen — sie ist kein personenbezogenes Datum.

### Was niemals erfasst werden darf

Unabhängig von Opt-out-Einstellungen dürfen folgende Daten **niemals** automatisch in einem Feedback-Report landen:

| Verbotene Daten | Risiko |
|---|---|
| Passwörter (auch verschlüsselte Hash-Werte) | Sicherheitsrisiko |
| Session-Token / API-Keys / Auth-Token | Account-Übernahme möglich |
| Persönliche Formularwerte (Name, Adresse, IBAN) | Datenschutzverletzung |
| Zahlungsdaten (Kreditkarten-Nummern) | PCI-DSS-Verstoß |
| Cookie-Inhalte mit Authentifizierungsdaten | Session-Hijacking |
| Standortdaten (GPS-Koordinaten) | Tracking ohne Einwilligung |

**Achtung bei State-Dumps:** Wenn die App einen kompletten Anwendungsstatus als Debug-Info exportiert, muss dieser vor dem Versand auf sensible Daten geprüft und bereinigt werden. Niemals rohen App-State ohne Filterung senden.

### Hinweis für KI-Agenten

Wenn ein KI-Agent die technische Datenerfassung implementiert, muss er sicherstellen:

1. Die Opt-out-Checkbox ist standardmäßig **angehakt** (Opt-in durch Belassen, nicht durch aktive Auswahl)
2. Die "Details anzeigen"-Funktion zeigt genau, welche Daten gesendet werden
3. Der Payload wird client-seitig bereinigt, bevor er das Netzwerk verlässt
4. Im Datenschutzhinweis der App wird die Datenerfassung im Feedback-System dokumentiert

→ Weiter mit [wiki/09-Floating-Button.md](09-Floating-Button.md)

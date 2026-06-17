# Beispiel: Wetter-App

Eine Wetter-App ist ein gutes Beispiel für eine API-abhängige App. Sie hat besondere Anforderungen an das Update-System weil eine alte App-Version brechen kann wenn der Wetter-Anbieter seine API ändert.

---

## Besonderheiten einer Wetter-App

- Hängt von einem externen Wetter-Anbieter ab
- API-Keys oder Backend-Proxy nötig
- Rate-Limits können ältere Clients betreffen
- API-Versionen werden vom Anbieter geändert
- Standort-Berechtigungen variieren je nach Plattform

---

## Empfohlener Phase-1-Ablauf

```text
Wetter-App startet
↓
Eigene Version prüfen
↓
Update-Manifest laden
  ├── App-Update verfügbar → Hinweis anzeigen
  └── Kein Update → weitermachen
↓
Remote-Konfiguration laden (optional)
  ├── API-Endpunkt aktuell?
  ├── Maintenance-Modus aktiv? → Hinweis anzeigen
  └── Minimale Client-Version unterschritten? → Update erzwingen
↓
Wetterdaten laden
```

---

## App-Update-Manifest

```json
{
  "app": "example-weather-app",
  "platform": "macos",
  "latestVersion": "3.2.0",
  "minimumVersion": "2.5.0",
  "apiCompatibilityVersion": "weather-api-v4",
  "downloadUrl": "https://updates.example.com/example-weather-app/releases/example-weather-app-3.2.0-macos.dmg",
  "changelog": [
    "Unterstützung für neue Wetter-API v4",
    "7-Tage-Vorhersage verbessert",
    "Widgets für macOS Sonoma"
  ],
  "forceUpdate": false
}
```

---

## Remote-Konfiguration

```json
{
  "app": "example-weather-app",
  "minimumClientVersion": "2.5.0",
  "apiCompatibilityVersion": "weather-api-v4",
  "maintenanceMode": false,
  "maintenanceMessage": null,
  "features": {
    "hourlyForecast": true,
    "radarMap": false
  }
}
```

---

## API-Key Sicherheit

Niemals den API-Key des Wetter-Anbieters direkt in die App einbetten.

**Option 1: Backend-Proxy**
```text
Wetter-App
↓ GET /weather?city=Berlin
Eigener Backend-Server
↓ GET mit echtem API-Key
Wetter-Anbieter
```

**Option 2: Nutzer-eigene Keys**
Nutzer trägt eigenen API-Key in den App-Einstellungen ein.

---

## Fragen für die Planung

- Wird der Anbieter direkt aufgerufen oder über einen eigenen Proxy?
- Ist die App für Mobile, Desktop oder beide?
- Wird über App-Stores oder direkt verteilt?
- Kann eine API-Änderung die App ohne Update brechen?
- Was passiert wenn der Anbieter offline ist?

→ Mehr dazu: [`../wiki/09-API-Clients.md`](../wiki/09-API-Clients.md)

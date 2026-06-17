# Beispiel: API-Client

Dieser Beispiel-Typ gilt für Apps die von externen APIs oder Backend-Services abhängen: Finanz-Apps, Karten-Apps, KI-Clients, Buchungs-Apps, SaaS-Companion-Apps.

---

## Das besondere Risiko bei API-Clients

Anders als reine Desktop-Apps können API-Clients brechen **ohne dass die App selbst veraltet ist** — nämlich dann wenn der externe Anbieter seine API ändert.

---

## Erweitertes Manifest für API-Clients

```json
{
  "app": "example-api-client",
  "platform": "windows",
  "latestVersion": "1.4.0",
  "minimumVersion": "1.2.0",
  "apiCompatibilityVersion": "api-v3",
  "minimumApiVersion": "api-v2",
  "remoteConfigUrl": "https://updates.example.com/example-api-client/config.json",
  "downloadUrl": "https://updates.example.com/example-api-client/releases/example-api-client-1.4.0-windows.exe",
  "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
  "changelog": [
    "Unterstützung für API v3",
    "Verbesserte Fehlerbehandlung bei Verbindungsabbrüchen",
    "Offline-Modus für gecachte Daten"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Remote-Konfiguration für API-Clients

```json
{
  "app": "example-api-client",
  "environment": "production",
  "apiEndpoint": "https://api.example.com/v3",
  "apiCompatibilityVersion": "api-v3",
  "minimumClientVersion": "1.2.0",
  "maintenanceMode": false,
  "maintenanceMessage": null,
  "features": {
    "offlineMode": true,
    "advancedSearch": false
  },
  "limits": {
    "maxItemsPerRequest": 100,
    "requestTimeoutSeconds": 30
  }
}
```

---

## Was die App prüfen muss

```text
App startet
↓
Eigene Version prüfen
↓
Remote-Konfiguration laden
  ├── maintenanceMode? → Wartungshinweis zeigen
  ├── minimumClientVersion unterschritten? → Pflicht-Update erzwingen
  └── apiCompatibilityVersion prüfen → API-Inkompatibilität warnen
↓
Normale Funktion starten
```

---

## API-Key Sicherheit

**Niemals** API-Keys in der App speichern.

| Lösung | Empfehlung |
|--------|-----------|
| Eigener Backend-Proxy | Beste Lösung — App ruft eigenen Server auf |
| Nutzerspezifische Keys | Gut bei Premium-Produkten |
| OAuth / PKCE | Für Dienste die OAuth unterstützen |

---

## Wichtige Fragen

1. Ruft die App den Anbieter direkt auf oder über ein eigenes Backend?
2. Was passiert wenn der Anbieter eine neue API-Version veröffentlicht?
3. Gibt es Rate-Limits die ältere Clients beeinflussen?
4. Braucht die App einen Offline-Modus?
5. Wer ist für die API-Stabilität verantwortlich?

→ Mehr dazu: [`../wiki/09-API-Clients.md`](../wiki/09-API-Clients.md)

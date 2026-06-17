# 09. API-Clients

Apps die von externen APIs abhängen brauchen spezielle Update-Planung: Die App kann brechen wenn sich die API ändert — auch ohne eine neue App-Version.

Beispiele: Wetter-Apps, Finanz-Apps, Karten-Apps, KI-Clients, SaaS-Clients, Buchungs-Apps.

---

## Das Kernproblem

```text
App-Version 1.0.0 nutzt API v2
↓
Anbieter stellt API v2 ab
↓
App-Version 1.0.0 bricht — auch ohne neue App-Version
```

Ohne Mindestversionsschutz sitzen Nutzer mit einer nicht mehr funktionierenden App.

---

## Zusätzliche Manifest-Felder für API-Clients

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
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Remote-Konfiguration

Für API-Clients ist eine Remote-Konfiguration oft wichtiger als ein App-Update.

Remote-Konfiguration erlaubt:
- API-Endpunkt ohne App-Update wechseln
- Maintenance-Modus aktivieren
- Feature-Flags setzen
- Rate-Limits anpassen

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
    "darkMode": true,
    "experimentalSearch": false
  }
}
```

---

## Fragen vor der Implementierung

1. Ruft die App den Anbieter direkt auf oder über ein eigenes Backend?
2. Sind API-Keys in der App gespeichert?
3. Kann die App brechen wenn der Anbieter die API-Version ändert?
4. Gibt es Rate-Limits die eine ältere App überlasten könnte?
5. Braucht die App plattformspezifische SDK-Versionen?
6. Muss die App bei Wartungsarbeiten des Anbieters einen Hinweis zeigen?

---

## API-Key-Sicherheit

**Niemals** API-Keys direkt in die App einbetten.

Optionen:

| Lösung | Beschreibung |
|--------|-------------|
| Eigener Backend-Proxy | App ruft eigenen Server auf, dieser ruft Anbieter auf |
| Nutzerspezifische Keys | Jeder Nutzer hat eigenen Key (z.B. bei Premium-Konten) |
| Verschlüsselte Konfiguration | Nur bei sehr niedrigem Schutzbedarf |

**Faustregel:** Alle Requests die API-Keys brauchen, gehören hinter einen eigenen Server.

---

## Nächster Schritt

→ Weiter mit [`12-Spiele-und-Content-Updates.md`](12-Spiele-und-Content-Updates.md)

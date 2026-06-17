# Beispiel: Tauri

Tauri hat einen eingebauten Updater. Die Konfiguration erfolgt über `tauri.conf.json`. Tauri erfordert signierte Releases.

---

## Voraussetzungen

- Tauri CLI installiert
- Code-Signierung auf macOS und Windows vorhanden
- Update-Endpunkt mit gültigen signierten Releases

---

## tauri.conf.json — Updater-Konfiguration

```json
{
  "tauri": {
    "updater": {
      "active": true,
      "endpoints": [
        "https://updates.example.com/example-app/{{target}}/{{arch}}/{{current_version}}"
      ],
      "dialog": true,
      "pubkey": "DEIN_OEFFENTLICHER_SCHLUESSEL"
    }
  }
}
```

`{{target}}`, `{{arch}}` und `{{current_version}}` werden von Tauri automatisch ersetzt.

---

## Update-Endpunkt Antwort

Tauri erwartet ein signiertes Update-Response-Format:

```json
{
  "version": "1.0.1",
  "notes": "Update-Hinweis hinzugefügt\nEinstellungen verbessert",
  "pub_date": "2026-06-17T00:00:00Z",
  "platforms": {
    "darwin-x86_64": {
      "signature": "SIGNATUR_HIER",
      "url": "https://updates.example.com/example-app/releases/example-app-1.0.1-x86_64.dmg"
    },
    "darwin-aarch64": {
      "signature": "SIGNATUR_HIER",
      "url": "https://updates.example.com/example-app/releases/example-app-1.0.1-aarch64.dmg"
    },
    "windows-x86_64": {
      "signature": "SIGNATUR_HIER",
      "url": "https://updates.example.com/example-app/releases/example-app-1.0.1-x86_64.msi"
    }
  }
}
```

---

## Signing Keys generieren

```bash
# Tauri-CLI Signing Key generieren
tauri signer generate -w ~/.tauri/myapp.key

# Public Key ausgeben (in tauri.conf.json eintragen)
tauri signer sign --private-key-path ~/.tauri/myapp.key installer.dmg
```

---

## Wichtige Fragen vor der Implementierung

1. Ist Signierung für alle Zielplattformen eingerichtet?
2. Wo wird der Private Key sicher aufbewahrt (nicht im Repository)?
3. Ist der Update-Endpunkt bereits öffentlich erreichbar?
4. Soll der Update-Dialog von Tauri oder eine eigene UI genutzt werden?

→ Checkliste: [`../checklists/phase-1.md`](../checklists/phase-1.md)

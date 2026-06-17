# Beispiel: Spiel-Client

Spiele brauchen oft zwei separate Update-Schichten: Client-Updates (das Programm) und Content-Updates (Level, Assets, Balancing).

---

## Die zwei Schichten

```text
Spiel-Client
├── Client-Update (Programm-Update)
│   → Neue Version des Spiels selbst
│   → Installer-Download wie normale App
│
└── Content-Update (Patch)
    → Neue Level, Quests, Assets, Balancing
    → Kleiner, häufiger, ohne Neuinstallation
```

---

## Client-Manifest

```json
{
  "app": "example-game",
  "platform": "windows",
  "latestVersion": "2.1.0",
  "minimumVersion": "2.0.0",
  "downloadUrl": "https://updates.example.com/example-game/releases/example-game-2.1.0-windows.exe",
  "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
  "changelog": [
    "Neues Gebiet: Kristallhöhle",
    "Balancing-Update für Gegner Stufe 10-15",
    "Performance-Verbesserungen"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Content-Manifest

```json
{
  "contentVersion": "2026-06-17-v2",
  "minimumClientVersion": "2.0.0",
  "baseUrl": "https://cdn.example.com/example-game/content/",
  "patches": [
    {
      "id": "chapter-3-assets",
      "file": "chapter-3.pak",
      "sha256": "b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4c6",
      "size": 89234567,
      "required": true
    },
    {
      "id": "summer-event-2026",
      "file": "summer-2026.pak",
      "sha256": "c5d7e9f1a3b5c7d9e1f3a5b7c9d1e3f5a7b9c1d3e5f7a9b1c3d5e7f9a1b3c5d7",
      "size": 12345678,
      "required": false,
      "expiresAt": "2026-09-01"
    }
  ]
}
```

---

## Update-Ablauf beim Spielstart

```text
Spiel startet
↓
Client-Version prüfen
  └── Update verfügbar → Hinweis anzeigen (nicht blockieren wenn optional)
↓
Content-Version prüfen
  └── Neue Patches → Im Hintergrund herunterladen
  └── Pflicht-Patch fehlt → Download vor Spielstart erzwingen
↓
Patches herunterladen und verifizieren (SHA-256)
↓
Spiel fortsetzen
```

---

## Offline-Betrieb

Offline-Spiele müssen immer startbar sein:

- Update-Prüfung im Hintergrund, niemals blockierend
- Fehlende Verbindung: letzten bekannten Stand nutzen
- Event-Content mit `expiresAt` bei Ablauf lokal deaktivieren

→ Mehr dazu: [`../wiki/10-Spiele-und-Content-Updates.md`](../wiki/10-Spiele-und-Content-Updates.md)

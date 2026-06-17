# 10. Spiele und Content-Updates

Spiele brauchen oft zwei getrennte Update-Schichten.

---

## Die zwei Schichten

### Client-Update

Ersetzt das Spielprogramm selbst.

- Neue Funktionen, Bugfixes, Engine-Änderungen
- Erfordert Download und Neuinstallation
- Versioniert wie normale App-Updates

### Content-Update

Ändert Spielinhalte ohne das Programm zu ersetzen.

Beispiele:
- Neue Level, Karten, Dungeons
- Quests und Story-Erweiterungen
- Balancing-Änderungen
- Gegenstände und Items
- Übersetzungen
- Events und saisonale Inhalte
- Asset-Updates (Grafik, Sound)

---

## Warum trennen?

| Vorteil | Erklärung |
|---------|-----------|
| Kleinere Downloads | Content-Patches oft viel kleiner als Client-Updates |
| Höhere Frequenz | Content kann täglich aktualisiert werden |
| Hotfixes | Balancing ohne App-Store-Prozess |
| A/B-Tests | Verschiedene Content-Varianten testen |

---

## Content-Manifest

```json
{
  "contentVersion": "2026-06-17-v3",
  "minimumClientVersion": "1.2.0",
  "baseUrl": "https://cdn.example.com/example-game/content/",
  "patches": [
    {
      "id": "chapter-3",
      "version": "1.0.0",
      "file": "chapter-3.pak",
      "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
      "size": 12345678,
      "required": true
    },
    {
      "id": "summer-event",
      "version": "2026-06-01",
      "file": "summer-event.pak",
      "sha256": "b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4c6",
      "size": 2345678,
      "required": false,
      "expiresAt": "2026-09-01"
    }
  ]
}
```

---

## Kompatibilität sichern

Content und Client müssen kompatibel bleiben:

```text
Content-Version 2.0 erfordert Client 1.3+
→ minimumClientVersion im Content-Manifest
→ Nutzer mit älterem Client: kein Content-Update, aber Client-Update anbieten
```

---

## Delta-Updates und Patches

Für große Spiele können Delta-Updates (nur Änderungen statt vollständige Dateien) sinnvoll sein.

**Wann Delta-Updates?**
- Assets sind groß (mehrere GB)
- Änderungen sind klein im Verhältnis
- Nutzer haben langsame Verbindungen

**Aufwand:** Delta-Update-Systeme sind komplex. Erst ab Phase 3+ und wirklichem Bedarf implementieren.

---

## Offline-Spiele

Bei Offline-Spielen muss die App auch ohne Internetverbindung starten.

Content-Updates daher:
- Im Hintergrund prüfen (nicht blockierend)
- Fehlende Verbindung stabil behandeln
- Zuletzt heruntergeladenen Content nutzen wenn kein Update möglich

---

## Nächster Schritt

→ Weiter mit [`11-GitHub-Releases.md`](11-GitHub-Releases.md)

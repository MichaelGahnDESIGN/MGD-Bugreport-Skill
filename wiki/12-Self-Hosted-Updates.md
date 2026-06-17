# 12. Self-Hosted Updates

Self-Hosted bedeutet: Der Update-Server liegt unter eigener Kontrolle — kein Drittanbieter-Dienst, kein GitHub, kein CDN-Anbieter entscheidet ob Updates ausgeliefert werden.

Das ist die empfohlene Lösung für professionelle Closed-Source-Apps.

---

## Warum Self-Hosted?

| Vorteil | Erklärung |
|---------|-----------|
| Volle Kontrolle | Kein Anbieter kann den Dienst abschalten |
| Datenschutz | Keine Update-Statistiken bei Drittanbietern |
| Flexibilität | Kanäle, Lizenzprüfung, Statistiken selbst implementierbar |
| Sicherheit | Kein Token in der App nötig |

---

## Minimaler Self-Hosted-Aufbau (Phase 1)

Alles was für Phase 1 nötig ist:

```text
Webhosting oder VPS
↓
Öffentlich erreichbares Verzeichnis
↓
latest.json darin ablegen
↓
Installer-Datei darin ablegen
```

```text
https://updates.example.com/example-app/macos/latest.json
https://updates.example.com/example-app/macos/releases/example-app-1.0.1-macos.dmg
```

Keine Datenbank, kein Framework, kein Server-Code nötig.

---

## Erweiterter Self-Hosted-Aufbau (Phase 2+)

Für mehr Kontrolle:

```text
Update-API (PHP, Node.js, Python, Go, ...)
↓ liest aus
Datenbank oder Konfigurations-Dateien
↓ gibt zurück
Versions-Informationen und signierte Download-URLs
```

Vorteile:
- Rollout-Kontrolle (10% der Nutzer zuerst)
- Lizenzbasierte Updates
- Nutzungsstatistiken
- A/B-Kanäle

---

## Einfache PHP Update-API

```php
<?php
// update.php — vereinfachtes Beispiel
header('Content-Type: application/json');

$app = $_GET['app'] ?? '';
$platform = $_GET['platform'] ?? '';

$releases = [
    'example-app' => [
        'macos' => [
            'latestVersion' => '1.0.1',
            'minimumVersion' => '1.0.0',
            'downloadUrl' => 'https://updates.example.com/releases/example-app-1.0.1-macos.dmg',
            'forceUpdate' => false,
        ],
        'windows' => [
            'latestVersion' => '1.0.1',
            'minimumVersion' => '1.0.0',
            'downloadUrl' => 'https://updates.example.com/releases/example-app-1.0.1-windows.exe',
            'forceUpdate' => false,
        ],
    ],
];

if (isset($releases[$app][$platform])) {
    echo json_encode($releases[$app][$platform]);
} else {
    http_response_code(404);
    echo json_encode(['error' => 'Not found']);
}
```

→ Vollständigeres Beispiel: [`examples/php-update-api.md`](../examples/php-update-api.md)

---

## Datei-Struktur auf dem Server

Empfohlene Struktur:

```text
/updates/
  example-app/
    macos/
      latest.json
      releases/
        example-app-1.0.0-macos.dmg
        example-app-1.0.1-macos.dmg
    windows/
      latest.json
      releases/
        example-app-1.0.0-windows.exe
        example-app-1.0.1-windows.exe
    linux/
      latest.json
      releases/
        example-app-1.0.0-linux.AppImage
        example-app-1.0.1-linux.AppImage
```

---

## Sicherheit beim Self-Hosting

- HTTPS aktivieren (Let's Encrypt, kostenlos)
- Alte Installer-Dateien nicht löschen (Nutzer können downgraden wollen)
- Zugriffslog aktivieren (Anomalien erkennen)
- Backups der Manifeste und Installer

---

## Nächster Schritt

→ Weiter mit [`13-Lizenzserver.md`](13-Lizenzserver.md)

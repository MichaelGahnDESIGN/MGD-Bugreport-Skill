# Beispiel: PHP Update-API

Eine kleine PHP-API kann Update-Informationen zurückgeben und dabei mehr Kontrolle bieten als eine statische JSON-Datei.

---

## Wann PHP-API statt statischem JSON?

Eine eigene API ist nützlich wenn:
- Lizenzstatus geprüft werden soll
- Rollout-Kontrolle (z.B. 10% der Nutzer zuerst) nötig ist
- Statistiken gesammelt werden sollen
- Plattformspezifische Antworten dynamisch generiert werden

---

## Einfache Update-API (update.php)

```php
<?php
header('Content-Type: application/json');
header('Access-Control-Allow-Origin: *');

// Eingabe validieren
$app = filter_input(INPUT_GET, 'app', FILTER_SANITIZE_SPECIAL_CHARS) ?? '';
$platform = filter_input(INPUT_GET, 'platform', FILTER_SANITIZE_SPECIAL_CHARS) ?? '';
$currentVersion = filter_input(INPUT_GET, 'version', FILTER_SANITIZE_SPECIAL_CHARS) ?? '';

// Erlaubte App-IDs und Plattformen
$allowedApps = ['example-app'];
$allowedPlatforms = ['macos', 'windows', 'linux'];

if (!in_array($app, $allowedApps, true) || !in_array($platform, $allowedPlatforms, true)) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid parameters']);
    exit;
}

// Release-Konfiguration (in Produktion aus Datenbank oder Config-Datei)
$releases = [
    'example-app' => [
        'macos' => [
            'latestVersion' => '1.0.1',
            'minimumVersion' => '1.0.0',
            'downloadUrl' => 'https://updates.example.com/releases/example-app-1.0.1-macos.dmg',
            'sha256' => 'a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8',
            'changelog' => ['Update-Prüfung hinzugefügt', 'Einstellungen verbessert'],
            'forceUpdate' => false,
        ],
        'windows' => [
            'latestVersion' => '1.0.1',
            'minimumVersion' => '1.0.0',
            'downloadUrl' => 'https://updates.example.com/releases/example-app-1.0.1-windows.exe',
            'sha256' => 'b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4c6',
            'changelog' => ['Update-Prüfung hinzugefügt', 'Einstellungen verbessert'],
            'forceUpdate' => false,
        ],
        'linux' => [
            'latestVersion' => '1.0.1',
            'minimumVersion' => '1.0.0',
            'downloadUrl' => 'https://updates.example.com/releases/example-app-1.0.1-linux.AppImage',
            'sha256' => 'c5d7e9f1a3b5c7d9e1f3a5b7c9d1e3f5a7b9c1d3e5f7a9b1c3d5e7f9a1b3c5d7',
            'changelog' => ['Update-Prüfung hinzugefügt', 'Einstellungen verbessert'],
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

---

## API-Aufruf von der App

```text
GET https://updates.example.com/update.php?app=example-app&platform=macos&version=1.0.0
```

---

## Sicherheitsregeln für die PHP-API

- Alle Eingaben validieren (nicht blind verwenden)
- Nur erlaubte App-IDs und Plattformen zulassen
- Keine internen Pfade oder Server-Details in Fehlermeldungen
- HTTPS erzwingen
- Logging: Keine Secrets oder private Daten loggen
- Rate-Limiting für Schutz vor automatisierten Anfragen

---

## Erweiterte Funktionen

Mögliche Erweiterungen (nach Bedarf):

| Funktion | Beschreibung |
|----------|-------------|
| Lizenzprüfung | Lizenzschlüssel aus Anfrage prüfen |
| Rollout-Steuerung | Prozentsatz der Nutzer die Update sehen |
| Kanal-Support | stable / beta je nach Parameter |
| Versions-Statistiken | Verteilung der installierten Versionen loggen |

→ Mehr dazu: [`../wiki/12-Self-Hosted-Updates.md`](../wiki/12-Self-Hosted-Updates.md)

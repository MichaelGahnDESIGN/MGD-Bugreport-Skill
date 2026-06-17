# 10. PHP-Backends für Update-APIs

PHP ist eine pragmatische Wahl für einfache Update-APIs. Es läuft auf fast jedem Webhosting, braucht keine komplexe Infrastruktur und ist gut genug für Phase 1 und Phase 2.

---

## Wann PHP statt statischem JSON?

Statisches JSON ist für die meisten Phase-1-Projekte ausreichend. Eine PHP-API wird sinnvoll wenn:

- Lizenzstatus geprüft werden soll
- Rollout-Kontrolle (10% der Nutzer zuerst) nötig ist
- Download-URLs signiert oder zeitlich begrenzt sein sollen
- Statistiken über installierte Versionen gesammelt werden
- Verschiedene Kanäle (stable, beta) bedient werden

---

## Einfaches PHP-Backend (Plain PHP)

```php
<?php
// update.php — Phase-1-Minimal-API
header('Content-Type: application/json');
header('Cache-Control: no-cache, no-store');

$app      = filter_input(INPUT_GET, 'app',      FILTER_SANITIZE_SPECIAL_CHARS) ?? '';
$platform = filter_input(INPUT_GET, 'platform', FILTER_SANITIZE_SPECIAL_CHARS) ?? '';

// Whitelist — nur bekannte Apps und Plattformen zulassen
$allowed = [
    'example-app' => ['macos', 'windows', 'linux'],
];

if (!isset($allowed[$app]) || !in_array($platform, $allowed[$app], true)) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid parameters']);
    exit;
}

// Releases aus Konfigurationsdatei laden (nicht direkt in Code)
$config = json_decode(file_get_contents(__DIR__ . '/releases/' . $app . '.json'), true);

if (isset($config[$platform])) {
    echo json_encode($config[$platform]);
} else {
    http_response_code(404);
    echo json_encode(['error' => 'Not found']);
}
```

### Konfigurations-JSON (releases/example-app.json)

```json
{
  "macos": {
    "latestVersion": "1.0.1",
    "minimumVersion": "1.0.0",
    "downloadUrl": "https://updates.example.com/releases/example-app-1.0.1-macos.dmg",
    "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
    "changelog": ["Update-Prüfung hinzugefügt", "Startproblem behoben"],
    "forceUpdate": false
  },
  "windows": {
    "latestVersion": "1.0.1",
    "minimumVersion": "1.0.0",
    "downloadUrl": "https://updates.example.com/releases/example-app-1.0.1-windows.exe",
    "sha256": "b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4c6",
    "changelog": ["Update-Prüfung hinzugefügt", "Startproblem behoben"],
    "forceUpdate": false
  }
}
```

---

## Flight PHP (Micro-Framework)

```php
<?php
require 'vendor/autoload.php';

use flight\Flight;

Flight::route('GET /updates/@app/@platform', function(string $app, string $platform) {
    $allowed = ['example-app' => ['macos', 'windows', 'linux']];

    if (!isset($allowed[$app]) || !in_array($platform, $allowed[$app], true)) {
        Flight::halt(400, json_encode(['error' => 'Invalid parameters']));
        return;
    }

    $file = __DIR__ . "/releases/{$app}.json";
    if (!file_exists($file)) {
        Flight::halt(404, json_encode(['error' => 'App not found']));
        return;
    }

    $config = json_decode(file_get_contents($file), true);

    if (isset($config[$platform])) {
        Flight::json($config[$platform]);
    } else {
        Flight::halt(404, json_encode(['error' => 'Platform not found']));
    }
});

Flight::start();
```

---

## Sicherheitsregeln für PHP-APIs

| Regel | Warum |
|-------|-------|
| Alle Eingaben validieren | SQL-Injection, Path-Traversal verhindern |
| Whitelist für App-IDs und Plattformen | Nur bekannte Werte akzeptieren |
| Keine internen Pfade in Fehlermeldungen | Angreifer soll nichts über Serverstruktur erfahren |
| HTTPS erzwingen | Manifeste dürfen nicht unverschlüsselt übertragen werden |
| Rate-Limiting aktivieren | Automatisierte Angriffe begrenzen |
| Keine Secrets in Code | Lizenzschlüssel, DB-Passwörter in `.env` oder Config-Dateien |
| `.htaccess` für releases/ schützen | Installer-Verzeichnis nicht direkt browsbar machen |

---

## Laravel-Variante

→ Vollständiges Beispiel: [`../examples/laravel-update-api.md`](../examples/laravel-update-api.md)

## Symfony-Variante

→ Vollständiges Beispiel: [`../examples/symfony-update-api.md`](../examples/symfony-update-api.md)

---

## Nächster Schritt

→ Weiter mit [`11-API-Clients.md`](11-API-Clients.md)

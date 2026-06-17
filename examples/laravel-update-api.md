# Beispiel: Laravel Update-API

Laravel eignet sich für komplexere Update-APIs die Lizenzprüfung, Rollout-Steuerung oder Statistiken benötigen.

---

## Route und Controller

```php
// routes/api.php
Route::get('/updates/{app}/{platform}', [UpdateController::class, 'check']);
Route::post('/updates/check', [UpdateController::class, 'checkWithLicense']);
```

```php
// app/Http/Controllers/UpdateController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class UpdateController extends Controller {
    private array $allowed = [
        'example-app' => ['macos', 'windows', 'linux'],
    ];

    public function check(string $app, string $platform): JsonResponse {
        if (!isset($this->allowed[$app]) ||
            !in_array($platform, $this->allowed[$app], true)) {
            return response()->json(['error' => 'Not found'], 404);
        }

        $release = $this->getRelease($app, $platform);

        if (!$release) {
            return response()->json(['error' => 'No release found'], 404);
        }

        return response()->json($release);
    }

    private function getRelease(string $app, string $platform): ?array {
        // Aus Datenbank, Config oder JSON-Datei laden
        $path = storage_path("releases/{$app}/{$platform}.json");

        if (!file_exists($path)) return null;

        return json_decode(file_get_contents($path), true);
    }
}
```

---

## Mit Lizenzprüfung

```php
public function checkWithLicense(Request $request): JsonResponse {
    $validated = $request->validate([
        'app'     => 'required|string|max:64',
        'platform'=> 'required|string|max:32',
        'version' => 'required|string|max:32',
        'license' => 'nullable|string|max:256',
    ]);

    $license = $validated['license'] ?? null;

    // Lizenz prüfen (eigene Logik)
    if ($license && !$this->isValidLicense($license, $validated['app'])) {
        return response()->json(['error' => 'Invalid license'], 403);
    }

    $release = $this->getRelease($validated['app'], $validated['platform']);

    return $release
        ? response()->json($release)
        : response()->json(['error' => 'Not found'], 404);
}

private function isValidLicense(string $license, string $app): bool {
    // Lizenz-Hash prüfen — niemals Plaintext-Vergleich
    return hash_equals(
        hash('sha256', $app . ':' . config('app.license_secret') . ':' . $license),
        $license
    );
    // In Produktion: Datenbank-Abfrage
}
```

---

## Releases/Storage-Struktur

```text
storage/
  releases/
    example-app/
      macos.json
      windows.json
      linux.json
```

---

## Sicherheitsregeln

- `.env` für alle Secrets (License-Secret, DB-Credentials)
- Rate-Limiting aktivieren (`throttle:60,1`)
- Authentifizierung für Admin-Routen
- HTTPS erzwingen (Laravel Middleware `ForceHttps` oder Server-Konfiguration)
- Keine Pfade oder interne Details in Fehlermeldungen

→ Wiki: [`../wiki/10-PHP-Backends.md`](../wiki/10-PHP-Backends.md)

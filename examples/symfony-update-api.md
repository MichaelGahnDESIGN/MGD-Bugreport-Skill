# Beispiel: Symfony Update-API

Symfony eignet sich für professionelle Update-APIs mit vollständiger Typsicherheit und Dependency Injection.

---

## Controller

```php
// src/Controller/UpdateController.php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

#[Route('/api')]
class UpdateController extends AbstractController {
    private const ALLOWED = [
        'example-app' => ['macos', 'windows', 'linux'],
    ];

    #[Route('/updates/{app}/{platform}', methods: ['GET'])]
    public function check(string $app, string $platform): JsonResponse {
        if (!isset(self::ALLOWED[$app]) ||
            !in_array($platform, self::ALLOWED[$app], true)) {
            return $this->json(['error' => 'Not found'], 404);
        }

        $release = $this->loadRelease($app, $platform);

        return $release
            ? $this->json($release)
            : $this->json(['error' => 'No release configured'], 404);
    }

    private function loadRelease(string $app, string $platform): ?array {
        $path = $this->getParameter('kernel.project_dir')
            . "/var/releases/{$app}/{$platform}.json";

        if (!file_exists($path)) return null;

        return json_decode(file_get_contents($path), true);
    }
}
```

---

## Service für Lizenzprüfung

```php
// src/Service/LicenseService.php
<?php

namespace App\Service;

class LicenseService {
    public function __construct(
        private readonly string $licenseSecret
    ) {}

    public function isValid(string $licenseKey, string $appId): bool {
        // Niemals Plaintext-Vergleich
        $expected = hash('sha256', $appId . ':' . $this->licenseSecret . ':' . $licenseKey);
        // In Produktion: Datenbank-Abfrage
        return hash_equals($expected, $licenseKey);
    }
}
```

```yaml
# config/services.yaml
services:
    App\Service\LicenseService:
        arguments:
            $licenseSecret: '%env(LICENSE_SECRET)%'
```

---

## Dateistruktur

```text
var/
  releases/
    example-app/
      macos.json
      windows.json
      linux.json

.env
  LICENSE_SECRET=sehr-geheimes-passwort
```

---

## Sicherheitsregeln

- Alle Eingaben durch Routing (Symfony validiert Routen-Parameter)
- Secrets in `.env` nie im Repository
- `framework.csrf_protection` für POST-Endpunkte
- Rate-Limiting via `symfony/rate-limiter`
- HTTPS erzwingen (Webserver-Konfiguration oder Symfony TrustedProxies)

→ Wiki: [`../wiki/10-PHP-Backends.md`](../wiki/10-PHP-Backends.md)

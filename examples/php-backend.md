# PHP Backend – Feedback Hub API

Einfacher, sicherer PHP-Endpunkt für Bug-Reports, Ideen und Feedback. Ohne Framework, mit SQLite oder MySQL, Rate-Limiting und Datei-Upload für Screenshots.

## Verzeichnisstruktur

```
api/
├── report.php        ← POST /api/report (Haupt-Endpunkt)
├── config.php        ← Datenbankverbindung und Konstanten
├── RateLimiter.php   ← Einfaches IP-basiertes Rate-Limiting
└── uploads/          ← Screenshot-Uploads (außerhalb webroot ideal)
```

## config.php

```php
<?php
// api/config.php
define('DB_PATH',       __DIR__ . '/feedback.sqlite');
define('UPLOAD_DIR',    __DIR__ . '/uploads/');
define('UPLOAD_MAX_MB', 5);
define('RATE_LIMIT_REQUESTS', 10);   // max. Anfragen pro Fenster
define('RATE_LIMIT_WINDOW',   60);   // Fenster in Sekunden
define('ALLOWED_TYPES', ['bug', 'idea', 'feedback']);
define('ALLOWED_SEVERITIES', ['low', 'medium', 'high', 'critical']);

// SQLite-Verbindung
function get_db(): PDO {
    static $pdo;
    if (!$pdo) {
        $pdo = new PDO('sqlite:' . DB_PATH, null, null, [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        ]);
        $pdo->exec("
            CREATE TABLE IF NOT EXISTS reports (
                id            INTEGER PRIMARY KEY AUTOINCREMENT,
                type          TEXT NOT NULL,
                title         TEXT NOT NULL,
                description   TEXT NOT NULL DEFAULT '',
                severity      TEXT NOT NULL DEFAULT 'medium',
                screenshot    TEXT,         -- relativer Pfad
                system_info   TEXT,         -- JSON
                ip_address    TEXT,
                created_at    DATETIME DEFAULT CURRENT_TIMESTAMP
            )
        ");
        $pdo->exec("
            CREATE TABLE IF NOT EXISTS rate_limit (
                ip_key   TEXT PRIMARY KEY,
                count    INTEGER DEFAULT 0,
                window   INTEGER DEFAULT 0
            )
        ");
    }
    return $pdo;
}
```

## RateLimiter.php

```php
<?php
// api/RateLimiter.php
require_once __DIR__ . '/config.php';

class RateLimiter {
    public static function check(string $ip): bool {
        $db      = get_db();
        $window  = (int) floor(time() / RATE_LIMIT_WINDOW);
        $key     = hash('sha256', $ip . ':' . $window); // keine rohe IP speichern

        $stmt = $db->prepare(
            'INSERT INTO rate_limit (ip_key, count, window)
             VALUES (:key, 1, :win)
             ON CONFLICT(ip_key) DO UPDATE
             SET count = CASE WHEN window = :win THEN count + 1 ELSE 1 END,
                 window = :win'
        );
        $stmt->execute([':key' => $key, ':win' => $window]);

        $row = $db->query("SELECT count FROM rate_limit WHERE ip_key = " .
                          $db->quote($key))->fetch(PDO::FETCH_ASSOC);
        return (int) ($row['count'] ?? 0) <= RATE_LIMIT_REQUESTS;
    }
}
```

## report.php – Haupt-Endpunkt

```php
<?php
// api/report.php
require_once __DIR__ . '/config.php';
require_once __DIR__ . '/RateLimiter.php';

// CORS-Header (anpassen an eigene Domain)
header('Access-Control-Allow-Origin: https://app.example.com');
header('Access-Control-Allow-Methods: POST, OPTIONS');
header('Access-Control-Allow-Headers: Content-Type, Authorization');
header('Content-Type: application/json; charset=utf-8');

if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    http_response_code(204);
    exit;
}

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    json_error(405, 'Methode nicht erlaubt');
}

// IP-Adresse ermitteln (Proxy-Header berücksichtigen)
$ip = filter_var(
    $_SERVER['HTTP_X_FORWARDED_FOR'] ?? $_SERVER['REMOTE_ADDR'] ?? '',
    FILTER_VALIDATE_IP
) ?: '0.0.0.0';

// Rate-Limiting prüfen
if (!RateLimiter::check($ip)) {
    json_error(429, 'Zu viele Anfragen – bitte warte eine Minute.');
}

// Eingabe lesen (JSON oder Multipart)
$contentType = $_SERVER['CONTENT_TYPE'] ?? '';
if (str_contains($contentType, 'application/json')) {
    $raw  = file_get_contents('php://input');
    $data = json_decode($raw, true);
    if (!is_array($data)) json_error(400, 'Ungültiges JSON');
} else {
    $data = $_POST;
}

// Validierung
$type        = trim($data['type'] ?? '');
$title       = trim($data['title'] ?? '');
$description = trim($data['description'] ?? '');
$severity    = trim($data['severity'] ?? 'medium');

if (!in_array($type, ALLOWED_TYPES, true))
    json_error(422, 'Ungültiger Typ. Erlaubt: ' . implode(', ', ALLOWED_TYPES));
if (strlen($title) < 3 || strlen($title) > 200)
    json_error(422, 'Titel muss 3–200 Zeichen lang sein');
if (strlen($description) > 10000)
    json_error(422, 'Beschreibung zu lang (max. 10.000 Zeichen)');
if (!in_array($severity, ALLOWED_SEVERITIES, true))
    $severity = 'medium';

// System-Info (JSON-String oder Array)
$systemInfo = null;
if (!empty($data['system_info'])) {
    $si = is_string($data['system_info'])
        ? json_decode($data['system_info'], true)
        : $data['system_info'];
    $systemInfo = is_array($si) ? json_encode($si) : null;
}

// Screenshot-Upload (Multipart) oder Base64 im JSON
$screenshotPath = null;
if (!empty($_FILES['screenshot'])) {
    $screenshotPath = handle_upload($_FILES['screenshot']);
} elseif (!empty($data['screenshot'])) {
    // Base64 → Datei speichern
    $base64  = preg_replace('/^data:image\/\w+;base64,/', '', $data['screenshot']);
    $imgData = base64_decode($base64, true);
    if ($imgData !== false) {
        $filename = uniqid('scr_', true) . '.png';
        // Pfad-Traversal verhindern: nur basename verwenden
        $dest = UPLOAD_DIR . basename($filename);
        if (file_put_contents($dest, $imgData) !== false) {
            $screenshotPath = 'uploads/' . basename($filename);
        }
    }
}

// In Datenbank speichern
$db   = get_db();
$stmt = $db->prepare(
    'INSERT INTO reports (type, title, description, severity,
                          screenshot, system_info, ip_address)
     VALUES (:type, :title, :desc, :sev, :scr, :sys, :ip)'
);
$stmt->execute([
    ':type'  => $type,
    ':title' => htmlspecialchars($title, ENT_QUOTES, 'UTF-8'),
    ':desc'  => htmlspecialchars($description, ENT_QUOTES, 'UTF-8'),
    ':sev'   => $severity,
    ':scr'   => $screenshotPath,
    ':sys'   => $systemInfo,
    ':ip'    => hash('sha256', $ip),  // IP gehasht speichern
]);
$id = $db->lastInsertId();

http_response_code(201);
echo json_encode(['id' => (int) $id, 'status' => 'created']);

// --- Hilfsfunktionen ---

function json_error(int $code, string $message): never {
    http_response_code($code);
    echo json_encode(['error' => $message]);
    exit;
}

function handle_upload(array $file): ?string {
    if ($file['error'] !== UPLOAD_ERR_OK) return null;

    $maxBytes = UPLOAD_MAX_MB * 1024 * 1024;
    if ($file['size'] > $maxBytes) json_error(422, 'Datei zu groß (max. 5 MB)');

    // MIME-Typ prüfen (kein Verlass auf Dateiendung allein)
    $finfo    = new finfo(FILEINFO_MIME_TYPE);
    $mimeType = $finfo->file($file['tmp_name']);
    if (!in_array($mimeType, ['image/jpeg', 'image/png'], true))
        json_error(422, 'Nur JPG/PNG erlaubt');

    $ext      = $mimeType === 'image/png' ? '.png' : '.jpg';
    $filename = uniqid('scr_', true) . $ext;
    $dest     = UPLOAD_DIR . basename($filename); // Pfad-Traversal verhindern

    if (!is_dir(UPLOAD_DIR)) mkdir(UPLOAD_DIR, 0750, true);
    if (!move_uploaded_file($file['tmp_name'], $dest)) return null;

    return 'uploads/' . basename($filename);
}
```

## Sicherheits-Checkliste

- **Pfad-Traversal**: `basename()` auf allen Dateinamen.
- **MIME-Validierung**: `finfo` prüft echten Dateityp, nicht nur Endung.
- **Input-Sanitierung**: `htmlspecialchars()` beim Speichern.
- **Rate-Limiting**: IP-Hash, kein Klartext in der Datenbank.
- **Datei-Größe**: Serverseitig erzwungen, nicht nur Client-seitig.
- **CORS**: Nur eigene Domain erlauben.

# Laravel Backend – Feedback Hub API

Vollständige Laravel-Integration mit Model, Migration, FormRequest, Controller und optionaler GitHub-Issue-Synchronisation.

## Migration

```php
<?php
// database/migrations/2024_01_01_000000_create_reports_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('reports', function (Blueprint $table) {
            $table->id();
            $table->enum('type', ['bug', 'idea', 'feedback'])->index();
            $table->string('title', 200);
            $table->text('description')->nullable();
            $table->enum('severity', ['low', 'medium', 'high', 'critical'])
                  ->default('medium');
            $table->string('screenshot_path')->nullable();
            $table->json('system_info')->nullable();
            $table->enum('status', ['open', 'in_progress', 'closed', 'duplicate'])
                  ->default('open')->index();
            $table->string('github_issue_url')->nullable();
            $table->string('ip_hash', 64)->nullable();    // gehasht
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('reports');
    }
};
```

## Report Model

```php
<?php
// app/Models/Report.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Report extends Model {
    use HasFactory;

    protected $fillable = [
        'type', 'title', 'description', 'severity',
        'screenshot_path', 'system_info', 'status',
        'github_issue_url', 'ip_hash',
    ];

    protected $casts = [
        'system_info' => 'array',
    ];

    // Scopes für Filterung
    public function scopeOpen($query) {
        return $query->where('status', 'open');
    }

    public function scopeByType($query, string $type) {
        return $query->where('type', $type);
    }
}
```

## ReportRequest (Validierung)

```php
<?php
// app/Http/Requests/ReportRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class ReportRequest extends FormRequest {
    public function authorize(): bool { return true; }

    public function rules(): array {
        return [
            'type'        => 'required|in:bug,idea,feedback',
            'title'       => 'required|string|min:3|max:200',
            'description' => 'nullable|string|max:10000',
            'severity'    => 'sometimes|in:low,medium,high,critical',
            'screenshot'  => 'nullable|file|image|mimes:jpg,jpeg,png|max:5120', // 5 MB
            'system_info' => 'nullable|array',
            'system_info.os'          => 'sometimes|string|max:100',
            'system_info.os_version'  => 'sometimes|string|max:100',
            'system_info.app_version' => 'sometimes|string|max:50',
        ];
    }
}
```

## ReportController

```php
<?php
// app/Http/Controllers/Api/ReportController.php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\ReportRequest;
use App\Models\Report;
use App\Services\GitHubService;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;

class ReportController extends Controller {

    public function __construct(private GitHubService $github) {}

    public function store(ReportRequest $request): JsonResponse {
        $data = $request->validated();

        // Screenshot im private disk speichern (nicht öffentlich zugänglich)
        $screenshotPath = null;
        if ($request->hasFile('screenshot')) {
            $file     = $request->file('screenshot');
            $filename = Str::uuid() . '.' . $file->getClientOriginalExtension();
            $screenshotPath = Storage::disk('private')
                ->putFileAs('screenshots', $file, $filename);
        } elseif (!empty($data['screenshot'])) {
            // Base64-Screenshot (aus JSON-Payload)
            $base64   = preg_replace('/^data:image\/\w+;base64,/', '', $data['screenshot']);
            $imgData  = base64_decode($base64, strict: true);
            if ($imgData !== false) {
                $filename = Str::uuid() . '.png';
                Storage::disk('private')->put("screenshots/{$filename}", $imgData);
                $screenshotPath = "screenshots/{$filename}";
            }
        }

        // IP-Hash (DSGVO-konform)
        $ipHash = hash('sha256', $request->ip() . config('app.key'));

        $report = Report::create([
            'type'            => $data['type'],
            'title'           => $data['title'],
            'description'     => $data['description'] ?? '',
            'severity'        => $data['severity'] ?? 'medium',
            'screenshot_path' => $screenshotPath,
            'system_info'     => $data['system_info'] ?? null,
            'ip_hash'         => $ipHash,
        ]);

        // Optional: GitHub-Issue anlegen (nur für Bugs)
        if ($report->type === 'bug' && config('services.github.token')) {
            $issueUrl = $this->github->createIssue($report);
            if ($issueUrl) {
                $report->update(['github_issue_url' => $issueUrl]);
            }
        }

        return response()->json([
            'id'     => $report->id,
            'status' => 'created',
        ], 201);
    }

    public function index(): JsonResponse {
        $reports = Report::open()
            ->orderByDesc('created_at')
            ->paginate(20);
        return response()->json($reports);
    }
}
```

## GitHubService

```php
<?php
// app/Services/GitHubService.php
namespace App\Services;

use App\Models\Report;
use Illuminate\Http\Client\PendingRequest;
use Illuminate\Support\Facades\Http;

class GitHubService {

    private PendingRequest $client;

    public function __construct() {
        $this->client = Http::withToken(config('services.github.token'))
            ->baseUrl('https://api.github.com')
            ->withHeaders(['Accept' => 'application/vnd.github+json'])
            ->timeout(15);
    }

    public function createIssue(Report $report): ?string {
        $owner = config('services.github.owner');  // z.B. "example-org"
        $repo  = config('services.github.repo');   // z.B. "example-app"

        if (!$owner || !$repo) return null;

        $labels = match($report->severity) {
            'critical' => ['bug', 'critical'],
            'high'     => ['bug', 'priority: high'],
            default    => ['bug'],
        };

        $body = $report->description . "\n\n---\n" .
                "**System-Info:**\n```json\n" .
                json_encode($report->system_info, JSON_PRETTY_PRINT) .
                "\n```\n\n*Erstellt via Feedback Hub – Report #{$report->id}*";

        $response = $this->client->post("/repos/{$owner}/{$repo}/issues", [
            'title'  => "[Bug #{$report->id}] {$report->title}",
            'body'   => $body,
            'labels' => $labels,
        ]);

        return $response->successful()
            ? $response->json('html_url')
            : null;
    }
}
```

## routes/api.php

```php
<?php
// routes/api.php
use App\Http\Controllers\Api\ReportController;
use Illuminate\Support\Facades\Route;

Route::prefix('v1')->group(function () {
    Route::middleware(['throttle:10,1'])  // 10 Anfragen pro Minute
         ->post('/reports', [ReportController::class, 'store']);

    // Nur mit API-Key zugänglich (z.B. für internes Dashboard)
    Route::middleware(['auth:sanctum'])
         ->get('/reports', [ReportController::class, 'index']);
});
```

## config/services.php (Ergänzung)

```php
// config/services.php
'github' => [
    'token' => env('GITHUB_TOKEN'),
    'owner' => env('GITHUB_OWNER', 'example-org'),
    'repo'  => env('GITHUB_REPO',  'example-app'),
],
```

## .env Beispiel

```dotenv
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
GITHUB_OWNER=example-org
GITHUB_REPO=example-app
FILESYSTEM_DISK=local
```

## Sicherheitshinweise

- Screenshots im `private` Disk – nicht über `/storage` öffentlich erreichbar.
- IP-Adresse gehasht mit App-Key (DSGVO-konform).
- `throttle:10,1` Middleware verhindert Spam (Laravel Rate Limiting).
- GitHub-Token als Umgebungsvariable, nie in der Codebasis.

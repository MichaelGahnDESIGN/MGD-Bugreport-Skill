# Beispiel: Godot

Godot-Spiele können PCK-Dateien für Content-Updates nutzen. Client-Updates (das Spiel selbst) werden wie normale Desktop-Apps ausgeliefert.

---

## PCK-Dateien für Content-Updates

Godot kann `.pck`-Dateien zur Laufzeit nachladen. Das erlaubt Content-Updates ohne Neuinstallation des Spiels.

```gdscript
# content_updater.gd

const CONTENT_MANIFEST_URL = "https://updates.example.com/example-game/content/manifest.json"
const DOWNLOAD_DIR = "user://content/"

func check_content_updates() -> void:
    var request = HTTPRequest.new()
    add_child(request)
    request.request_completed.connect(_on_manifest_received)
    request.request(CONTENT_MANIFEST_URL)

func _on_manifest_received(result: int, _code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    if result != OK:
        return

    var json = JSON.parse_string(body.get_string_from_utf8())
    if json == null:
        return

    for patch in json.get("patches", []):
        _download_patch(patch)

func _download_patch(patch: Dictionary) -> void:
    var url: String = patch.get("url", "")
    var expected_sha256: String = patch.get("sha256", "")

    if url.is_empty():
        return

    var request = HTTPRequest.new()
    add_child(request)
    request.request_completed.connect(
        func(result, _code, _headers, body):
            _on_patch_downloaded(result, body, patch, expected_sha256)
    )
    request.request(url)

func _on_patch_downloaded(result: int, body: PackedByteArray, patch: Dictionary, expected_hash: String) -> void:
    if result != OK:
        return

    # SHA-256 prüfen
    var actual_hash = body.sha256_text()
    if actual_hash != expected_hash:
        push_error("Checksumme ungültig für Patch: " + patch.get("id", ""))
        return

    # PCK speichern
    var path = DOWNLOAD_DIR + patch.get("file", "content.pck")
    DirAccess.make_dir_recursive_absolute(DOWNLOAD_DIR)

    var file = FileAccess.open(path, FileAccess.WRITE)
    if file:
        file.store_buffer(body)
        file.close()
        ProjectSettings.load_resource_pack(path)
        print("Patch geladen: ", patch.get("id", ""))
```

---

## Client-Update-Manifest

```json
{
  "app": "example-game-godot",
  "platform": "linux",
  "latestVersion": "0.8.0",
  "minimumVersion": "0.5.0",
  "downloadUrl": "https://updates.example.com/example-game/releases/example-game-0.8.0-linux.AppImage",
  "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
  "changelog": [
    "Neue Karte: Nebelwald",
    "Gegner-KI verbessert",
    "Performance-Verbesserungen"
  ],
  "forceUpdate": false
}
```

---

## Content-Manifest

```json
{
  "contentVersion": "2026-06-17-v2",
  "minimumClientVersion": "0.5.0",
  "patches": [
    {
      "id": "nebelwald-map",
      "file": "nebelwald.pck",
      "url": "https://cdn.example.com/example-game/content/nebelwald.pck",
      "sha256": "b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4c6",
      "size": 8234567,
      "required": true
    }
  ]
}
```

---

## Sicherheitshinweis

PCK-Dateien werden in der Godot-Engine als vertrauenswürdig behandelt. Niemals PCK-Dateien von unverifizierten Quellen laden.

Immer:
- SHA-256 der heruntergeladenen PCK prüfen vor dem Laden
- Nur HTTPS für Downloads
- Keine PCK-URLs aus Nutzereingaben

→ Wiki: [`../wiki/12-Spiele-und-Content-Updates.md`](../wiki/12-Spiele-und-Content-Updates.md)

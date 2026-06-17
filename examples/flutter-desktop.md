# Beispiel: Flutter Desktop

Flutter Desktop unterstützt macOS, Windows und Linux aus einer gemeinsamen Codebasis. Flutter hat keinen eingebauten Auto-Updater — Update-Systeme müssen manuell implementiert oder über Packages eingebunden werden.

---

## Empfohlene Dateistruktur

```text
lib/
  update/
    update_service.dart       — Update-Prüfung und Download-Logik
    update_manifest.dart      — Manifest-Model
    version_comparator.dart   — Versionsvergleich
    update_dialog.dart        — Update-Dialog Widget
```

---

## update_manifest.dart

```dart
class UpdateManifest {
  final String app;
  final String platform;
  final String latestVersion;
  final String minimumVersion;
  final String downloadUrl;
  final List<String> changelog;
  final bool forceUpdate;

  const UpdateManifest({
    required this.app,
    required this.platform,
    required this.latestVersion,
    required this.minimumVersion,
    required this.downloadUrl,
    required this.changelog,
    required this.forceUpdate,
  });

  factory UpdateManifest.fromJson(Map<String, dynamic> json) {
    return UpdateManifest(
      app: json['app'] as String,
      platform: json['platform'] as String,
      latestVersion: json['latestVersion'] as String,
      minimumVersion: json['minimumVersion'] as String,
      downloadUrl: json['downloadUrl'] as String,
      changelog: List<String>.from(json['changelog'] as List),
      forceUpdate: json['forceUpdate'] as bool,
    );
  }
}
```

---

## version_comparator.dart

```dart
class VersionComparator {
  static int compare(String a, String b) {
    final partsA = a.split('.').map(int.parse).toList();
    final partsB = b.split('.').map(int.parse).toList();

    for (var i = 0; i < 3; i++) {
      final pa = i < partsA.length ? partsA[i] : 0;
      final pb = i < partsB.length ? partsB[i] : 0;
      if (pa != pb) return pa.compareTo(pb);
    }
    return 0;
  }

  static bool isNewer(String remote, String installed) =>
      compare(remote, installed) > 0;

  static bool isBelowMinimum(String installed, String minimum) =>
      compare(installed, minimum) < 0;
}
```

---

## update_service.dart (Phase 1)

```dart
import 'dart:convert';
import 'dart:io';
import 'package:http/http.dart' as http;

class UpdateService {
  static const String _manifestBaseUrl =
      'https://updates.example.com/example-app';
  static const String _currentVersion = '1.0.0';

  static String get _platform {
    if (Platform.isMacOS) return 'macos';
    if (Platform.isWindows) return 'windows';
    if (Platform.isLinux) return 'linux';
    return 'unknown';
  }

  Future<UpdateManifest?> checkForUpdate() async {
    try {
      final uri = Uri.parse('$_manifestBaseUrl/$_platform/latest.json');
      final response = await http
          .get(uri)
          .timeout(const Duration(seconds: 15));

      if (response.statusCode != 200) return null;

      final manifest = UpdateManifest.fromJson(
        jsonDecode(response.body) as Map<String, dynamic>,
      );

      final hasUpdate =
          VersionComparator.isNewer(manifest.latestVersion, _currentVersion);
      final belowMinimum =
          VersionComparator.isBelowMinimum(_currentVersion, manifest.minimumVersion);

      if (hasUpdate || belowMinimum) return manifest;
      return null;
    } catch (_) {
      // Netzwerkfehler still behandeln — App darf nicht abstürzen
      return null;
    }
  }
}
```

---

## Manifest-URL

```text
https://updates.example.com/example-app/macos/latest.json
https://updates.example.com/example-app/windows/latest.json
https://updates.example.com/example-app/linux/latest.json
```

---

## Nächste Schritte

1. Phase 1: `UpdateService.checkForUpdate()` beim App-Start im Hintergrund aufrufen
2. Bei Ergebnis: `UpdateDialog` anzeigen mit Changelog und Download-Button
3. Phase 2: Download-Logik mit SHA-256-Verifikation hinzufügen
4. Phase 3: Erst nach stabilem Release-Prozess und Code-Signierung

→ Checkliste: [`../checklists/phase-1.md`](../checklists/phase-1.md)

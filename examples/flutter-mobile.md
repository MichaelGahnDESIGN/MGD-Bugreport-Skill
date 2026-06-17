# Beispiel: Flutter Mobile

Flutter Mobile-Apps (iOS und Android) werden über App Stores verteilt. Das bedeutet: **Kein eigener Installer-Updater**. Das eigene Backend dient nur für Mindestversionscheck, Content-Updates und Remote-Konfiguration.

---

## Was ist möglich, was nicht?

| Funktion | iOS | Android |
|---------|-----|---------|
| Eigener Installer | Nicht erlaubt | Side-Loading möglich, nicht empfohlen |
| Mindestversionscheck | Ja — zum Store leiten | Ja — zum Play Store leiten |
| Content-Updates (Daten) | Ja | Ja |
| Remote-Konfiguration | Ja | Ja |
| Play Core In-App-Update | Nein | Ja |

---

## Mindestversionscheck (Phase 1 für Mobile)

```dart
import 'dart:convert';
import 'dart:io';
import 'package:http/http.dart' as http;
import 'package:url_launcher/url_launcher.dart';
import 'package:package_info_plus/package_info_plus.dart';

class MobileUpdateService {
  static const String _manifestBaseUrl =
      'https://updates.example.com/example-app';

  static String get _platform => Platform.isIOS ? 'ios' : 'android';

  Future<void> checkMinimumVersion(BuildContext context) async {
    final packageInfo = await PackageInfo.fromPlatform();
    final currentVersion = packageInfo.version;

    try {
      final uri = Uri.parse('$_manifestBaseUrl/$_platform/latest.json');
      final response = await http.get(uri).timeout(const Duration(seconds: 10));
      if (response.statusCode != 200) return;

      final manifest = jsonDecode(response.body) as Map<String, dynamic>;
      final minimum = manifest['minimumVersion'] as String;

      if (_isBelowMinimum(currentVersion, minimum)) {
        _showForceUpdateDialog(context, manifest);
      }
    } catch (_) {
      // Netzwerkfehler still behandeln
    }
  }

  bool _isBelowMinimum(String installed, String minimum) {
    final a = installed.split('.').map(int.parse).toList();
    final b = minimum.split('.').map(int.parse).toList();
    for (var i = 0; i < 3; i++) {
      final diff = (a.elementAtOrNull(i) ?? 0) - (b.elementAtOrNull(i) ?? 0);
      if (diff != 0) return diff < 0;
    }
    return false;
  }

  void _showForceUpdateDialog(BuildContext context, Map<String, dynamic> manifest) {
    final storeUrl = Platform.isIOS
        ? 'https://apps.apple.com/app/idXXXXXXXXX'
        : 'https://play.google.com/store/apps/details?id=com.example.app';

    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (_) => AlertDialog(
        title: const Text('Update erforderlich'),
        content: const Text('Bitte aktualisiere die App um fortzufahren.'),
        actions: [
          TextButton(
            onPressed: () => launchUrl(Uri.parse(storeUrl)),
            child: const Text('Jetzt aktualisieren'),
          ),
        ],
      ),
    );
  }
}
```

---

## Manifest für Mobile

```json
{
  "app": "example-app",
  "platform": "ios",
  "latestVersion": "2.3.0",
  "minimumVersion": "2.0.0",
  "storeUrl": "https://apps.apple.com/app/idXXXXXXXXX",
  "changelog": [
    "Neue Funktion X",
    "Verbesserte Performance",
    "Bugfix beim Login"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Content-Updates für Flutter Mobile

Für Spiele oder Apps mit Remote-Content kann das eigene Backend dynamische Inhalte liefern ohne App-Store-Update.

```dart
// Remote-Konfiguration laden
Future<Map<String, dynamic>> loadRemoteConfig() async {
  final uri = Uri.parse('https://updates.example.com/example-app/config.json');
  final response = await http.get(uri).timeout(const Duration(seconds: 10));
  return jsonDecode(response.body) as Map<String, dynamic>;
}
```

→ Checklisten: [`../checklists/ios.md`](../checklists/ios.md) · [`../checklists/android.md`](../checklists/android.md)

# Flutter Mobile – Feedback Hub Integration

Unterschiede zur Desktop-Version: Plattform-spezifische System-Infos, Screenshot nur mit Nutzerbestätigung auf iOS, Privacy Manifest für iOS erforderlich.

## Voraussetzungen

```yaml
# pubspec.yaml
dependencies:
  http: ^1.2.0
  device_info_plus: ^9.1.0
  # Kein flutter_screenshot nötig – RepaintBoundary reicht für eigene Widgets
```

## Plattform-Check und System-Infos

```dart
// lib/feedback/system_info_mobile.dart
import 'dart:io';
import 'package:device_info_plus/device_info_plus.dart';

class SystemInfoMobile {
  static Future<Map<String, String>> collect() async {
    final info = DeviceInfoPlugin();

    if (Platform.isIOS) {
      final ios = await info.iosInfo;
      return {
        'platform': 'iOS',
        'os_version': ios.systemVersion,
        'device_model': ios.model,
        'device_name': ios.name,
        // Kein Platform.version auf iOS – dart:io liefert keinen sinnvollen Wert
        'locale': Platform.localeName,
      };
    } else if (Platform.isAndroid) {
      final android = await info.androidInfo;
      return {
        'platform': 'Android',
        'os_version': android.version.release,
        'sdk_int': android.version.sdkInt.toString(),
        'device_model': android.model,
        'manufacturer': android.manufacturer,
        'locale': Platform.localeName,
      };
    }
    return {'platform': Platform.operatingSystem};
  }
}
```

## FeedbackService (Mobile)

```dart
// lib/feedback/feedback_service_mobile.dart
import 'dart:convert';
import 'dart:ui' as ui;
import 'package:flutter/rendering.dart';
import 'package:http/http.dart' as http;
import 'system_info_mobile.dart';

enum FeedbackType { bug, idea, feedback }

class FeedbackServiceMobile {
  static const String _baseUrl = 'https://feedback.example.com/api';

  // Screenshot vom eigenen Widget-Baum – keine App-weiten Screenshots auf iOS
  // iOS: Nutzer muss explizit zustimmen (siehe Modal)
  static Future<String?> captureWidget(GlobalKey key) async {
    try {
      final boundary = key.currentContext?.findRenderObject()
          as RenderRepaintBoundary?;
      if (boundary == null) return null;
      final image = await boundary.toImage(pixelRatio: 2.0);
      final bytes = await image.toByteData(format: ui.ImageByteFormat.png);
      return bytes != null ? base64Encode(bytes.buffer.asUint8List()) : null;
    } catch (_) {
      return null;
    }
  }

  Future<bool> submitReport({
    required FeedbackType type,
    required String title,
    required String description,
    String? screenshotBase64,
  }) async {
    final sysInfo = await SystemInfoMobile.collect();
    try {
      final response = await http.post(
        Uri.parse('$_baseUrl/report'),
        headers: {'Content-Type': 'application/json'},
        body: jsonEncode({
          'type': type.name,
          'title': title,
          'description': description,
          'screenshot': screenshotBase64,
          'system_info': sysInfo,
          'created_at': DateTime.now().toIso8601String(),
        }),
      ).timeout(const Duration(seconds: 15));
      return response.statusCode == 201;
    } catch (_) {
      return false;
    }
  }
}
```

## FeedbackModal mit iOS-Bestätigung

```dart
// lib/feedback/feedback_modal_mobile.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'feedback_service_mobile.dart';

final GlobalKey feedbackRepaintKey = GlobalKey();

class FeedbackModalMobile extends StatefulWidget {
  const FeedbackModalMobile({super.key});

  @override
  State<FeedbackModalMobile> createState() => _FeedbackModalMobileState();
}

class _FeedbackModalMobileState extends State<FeedbackModalMobile> {
  FeedbackType _type = FeedbackType.bug;
  final _titleCtrl = TextEditingController();
  final _descCtrl = TextEditingController();
  bool _includeScreenshot = false;
  bool _sending = false;

  @override
  Widget build(BuildContext context) {
    return DraggableScrollableSheet(
      expand: false,
      initialChildSize: 0.75,
      builder: (_, ctrl) => SingleChildScrollView(
        controller: ctrl,
        padding: const EdgeInsets.all(16),
        child: Column(crossAxisAlignment: CrossAxisAlignment.stretch, children: [
          const Text('Feedback', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          const SizedBox(height: 12),
          // Typ-Auswahl
          SegmentedButton<FeedbackType>(
            segments: const [
              ButtonSegment(value: FeedbackType.bug, label: Text('Bug')),
              ButtonSegment(value: FeedbackType.idea, label: Text('Idee')),
              ButtonSegment(value: FeedbackType.feedback, label: Text('Feedback')),
            ],
            selected: {_type},
            onSelectionChanged: (s) => setState(() => _type = s.first),
          ),
          const SizedBox(height: 12),
          TextField(controller: _titleCtrl,
              decoration: const InputDecoration(labelText: 'Titel', border: OutlineInputBorder())),
          const SizedBox(height: 8),
          TextField(controller: _descCtrl, maxLines: 3,
              decoration: const InputDecoration(labelText: 'Beschreibung', border: OutlineInputBorder())),
          const SizedBox(height: 8),
          // iOS: Nutzer muss Screenshot explizit erlauben (App Store Compliance)
          SwitchListTile(
            title: Text(Platform.isIOS
                ? 'Screenshot anhängen (nur aktuelles Widget)'
                : 'Screenshot anhängen'),
            subtitle: Platform.isIOS
                ? const Text('Nur der App-Inhalt, kein System-Screenshot.')
                : null,
            value: _includeScreenshot,
            onChanged: (v) => setState(() => _includeScreenshot = v),
          ),
          const SizedBox(height: 12),
          FilledButton(
            onPressed: _sending ? null : _send,
            child: _sending
                ? const SizedBox(width: 20, height: 20,
                    child: CircularProgressIndicator(color: Colors.white, strokeWidth: 2))
                : const Text('Senden'),
          ),
        ]),
      ),
    );
  }

  Future<void> _send() async {
    if (_titleCtrl.text.trim().isEmpty) return;
    setState(() => _sending = true);

    String? screenshot;
    if (_includeScreenshot) {
      screenshot = await FeedbackServiceMobile.captureWidget(feedbackRepaintKey);
    }

    final ok = await FeedbackServiceMobile().submitReport(
      type: _type,
      title: _titleCtrl.text.trim(),
      description: _descCtrl.text.trim(),
      screenshotBase64: screenshot,
    );

    if (mounted) {
      Navigator.pop(context);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(ok ? 'Gesendet!' : 'Fehler – bitte erneut versuchen.')),
      );
    }
  }
}
```

## iOS Privacy Manifest

Für iOS 17+ muss `PrivacyInfo.xcprivacy` im iOS-Ordner angelegt werden, falls `device_info_plus` Gerätedaten abfragt:

```xml
<!-- ios/Runner/PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
  <key>NSPrivacyAccessedAPITypes</key>
  <array>
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array><string>CA92.1</string></array>
    </dict>
  </array>
</dict>
</plist>
```

## Hinweise zur App Store Compliance

- Kein automatischer Screenshot ohne Nutzerbestätigung (App Store Review Guideline 5.1.1).
- `UIGraphicsImageRenderer` bzw. `RenderRepaintBoundary` erfassen nur den eigenen Widget-Baum – kein System-UI.
- `CGWindowListCreateImage` ist auf iOS **nicht** verfügbar (nur macOS).
- Automatische Hintergrund-Screenshots werden von Apple abgelehnt.

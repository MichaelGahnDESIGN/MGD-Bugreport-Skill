# Flutter Desktop – Feedback Hub Integration

Dieses Beispiel zeigt die vollständige Integration eines Feedback-Hubs (Bug-Reports, Ideen, allgemeines Feedback) in eine Flutter-Desktop-App.

## Voraussetzungen

```yaml
# pubspec.yaml
dependencies:
  http: ^1.2.0
  path_provider: ^2.1.0
```

## FeedbackReport Datenklasse

```dart
// lib/feedback/feedback_report.dart
import 'dart:convert';

enum FeedbackType { bug, idea, feedback }

class FeedbackReport {
  final FeedbackType type;
  final String title;
  final String description;
  final String severity; // low | medium | high | critical
  final String? screenshotBase64;
  final Map<String, String> systemInfo;
  final DateTime createdAt;

  FeedbackReport({
    required this.type,
    required this.title,
    required this.description,
    this.severity = 'medium',
    this.screenshotBase64,
    required this.systemInfo,
    DateTime? createdAt,
  }) : createdAt = createdAt ?? DateTime.now();

  Map<String, dynamic> toJson() => {
    'type': type.name,
    'title': title,
    'description': description,
    'severity': severity,
    'screenshot': screenshotBase64,
    'system_info': systemInfo,
    'created_at': createdAt.toIso8601String(),
  };
}
```

## FeedbackService

```dart
// lib/feedback/feedback_service.dart
import 'dart:io';
import 'dart:convert';
import 'dart:ui' as ui;
import 'package:flutter/rendering.dart';
import 'package:http/http.dart' as http;
import 'feedback_report.dart';

class FeedbackService {
  static const String _baseUrl = 'https://feedback.example.com/api';

  // System-Informationen sammeln (Desktop: dart:io verfügbar)
  static Map<String, String> collectSystemInfo() => {
    'os': Platform.operatingSystem,
    'os_version': Platform.operatingSystemVersion,
    'dart_version': Platform.version,
    'locale': Platform.localeName,
    'number_of_processors': Platform.numberOfProcessors.toString(),
  };

  // Screenshot über GlobalKey des Root-Widgets
  static Future<String?> takeScreenshot(GlobalKey key) async {
    try {
      final boundary = key.currentContext?.findRenderObject()
          as RenderRepaintBoundary?;
      if (boundary == null) return null;
      final image = await boundary.toImage(pixelRatio: 1.5);
      final byteData = await image.toByteData(format: ui.ImageByteFormat.png);
      if (byteData == null) return null;
      return base64Encode(byteData.buffer.asUint8List());
    } catch (_) {
      return null;
    }
  }

  Future<bool> submitBug(String title, String description,
      {String severity = 'medium', String? screenshotBase64}) =>
      _submit(FeedbackReport(
        type: FeedbackType.bug,
        title: title,
        description: description,
        severity: severity,
        screenshotBase64: screenshotBase64,
        systemInfo: collectSystemInfo(),
      ));

  Future<bool> submitIdea(String title, String description) =>
      _submit(FeedbackReport(
        type: FeedbackType.idea,
        title: title,
        description: description,
        systemInfo: collectSystemInfo(),
      ));

  Future<bool> submitFeedback(String title, String description) =>
      _submit(FeedbackReport(
        type: FeedbackType.feedback,
        title: title,
        description: description,
        systemInfo: collectSystemInfo(),
      ));

  Future<bool> _submit(FeedbackReport report) async {
    try {
      final response = await http.post(
        Uri.parse('$_baseUrl/report'),
        headers: {'Content-Type': 'application/json'},
        body: jsonEncode(report.toJson()),
      ).timeout(const Duration(seconds: 15));
      return response.statusCode == 201;
    } catch (_) {
      return false;
    }
  }
}
```

## FeedbackHubButton Widget

```dart
// lib/feedback/feedback_hub_button.dart
import 'package:flutter/material.dart';
import 'feedback_service.dart';
import 'feedback_modal.dart';

// Globaler Key für Screenshot des App-Inhalts
final GlobalKey appRepaintKey = GlobalKey();

class FeedbackHubButton extends StatefulWidget {
  const FeedbackHubButton({super.key});

  @override
  State<FeedbackHubButton> createState() => _FeedbackHubButtonState();
}

class _FeedbackHubButtonState extends State<FeedbackHubButton> {
  @override
  Widget build(BuildContext context) {
    return FloatingActionButton.extended(
      onPressed: () => _openModal(context),
      icon: const Icon(Icons.feedback_outlined),
      label: const Text('Feedback'),
      backgroundColor: Colors.indigo,
    );
  }

  Future<void> _openModal(BuildContext context) async {
    // Screenshot vor dem Öffnen des Modals aufnehmen
    final screenshot = await FeedbackService.takeScreenshot(appRepaintKey);
    if (!context.mounted) return;
    showDialog(
      context: context,
      builder: (_) => FeedbackModal(screenshotBase64: screenshot),
    );
  }
}
```

## FeedbackModal Widget

```dart
// lib/feedback/feedback_modal.dart
import 'package:flutter/material.dart';
import 'feedback_report.dart';
import 'feedback_service.dart';

class FeedbackModal extends StatefulWidget {
  final String? screenshotBase64;
  const FeedbackModal({super.key, this.screenshotBase64});

  @override
  State<FeedbackModal> createState() => _FeedbackModalState();
}

class _FeedbackModalState extends State<FeedbackModal> {
  FeedbackType _type = FeedbackType.bug;
  final _titleCtrl = TextEditingController();
  final _descCtrl = TextEditingController();
  bool _sending = false;

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Feedback senden'),
      content: SizedBox(
        width: 480,
        child: Column(mainAxisSize: MainAxisSize.min, children: [
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
          TextField(controller: _titleCtrl, decoration: const InputDecoration(labelText: 'Titel')),
          const SizedBox(height: 8),
          TextField(controller: _descCtrl, maxLines: 4,
              decoration: const InputDecoration(labelText: 'Beschreibung')),
        ]),
      ),
      actions: [
        TextButton(onPressed: () => Navigator.pop(context), child: const Text('Abbrechen')),
        FilledButton(
          onPressed: _sending ? null : _send,
          child: _sending ? const SizedBox(width: 16, height: 16,
              child: CircularProgressIndicator(strokeWidth: 2)) : const Text('Senden'),
        ),
      ],
    );
  }

  Future<void> _send() async {
    if (_titleCtrl.text.trim().isEmpty) return;
    setState(() => _sending = true);
    final svc = FeedbackService();
    final ok = await svc._submit(FeedbackReport(
      type: _type, title: _titleCtrl.text.trim(),
      description: _descCtrl.text.trim(),
      screenshotBase64: widget.screenshotBase64,
      systemInfo: FeedbackService.collectSystemInfo(),
    ));
    if (mounted) {
      Navigator.pop(context);
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(
        content: Text(ok ? 'Feedback gesendet!' : 'Fehler – bitte erneut versuchen.'),
      ));
    }
  }
}
```

## App-Root mit RepaintBoundary

```dart
// lib/main.dart (Ausschnitt)
RepaintBoundary(
  key: appRepaintKey,
  child: Scaffold(
    body: const MyAppContent(),
    floatingActionButton: const FeedbackHubButton(),
  ),
)
```

## API-Endpunkt

`POST https://feedback.example.com/api/report`  
Content-Type: `application/json`  
Erwartete Antwort: `201 Created` mit `{"id": "..."}`.

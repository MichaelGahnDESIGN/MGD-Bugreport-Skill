# Swift / SwiftUI macOS – Feedback Hub Integration

Vollständige Integration eines Feedback-Hubs für macOS mit SwiftUI, Screenshot via CoreGraphics und async/await Netzwerkkommunikation.

## FeedbackReport Datenstruktur

```swift
// FeedbackReport.swift
import Foundation

enum FeedbackType: String, Codable, CaseIterable {
    case bug, idea, feedback
}

struct FeedbackReport: Codable {
    let type: FeedbackType
    let title: String
    let description: String
    let severity: String        // low | medium | high | critical
    let screenshotBase64: String?
    let systemInfo: SystemInfo
    let createdAt: Date

    struct SystemInfo: Codable {
        let osVersion: String
        let appVersion: String
        let appBuild: String
        let hostname: String
        let processorCount: Int
    }
}
```

## SystemInfoCollector

```swift
// SystemInfoCollector.swift
import Foundation
import AppKit

struct SystemInfoCollector {
    static func collect() -> FeedbackReport.SystemInfo {
        let info = Bundle.main.infoDictionary
        return FeedbackReport.SystemInfo(
            osVersion: ProcessInfo.processInfo.operatingSystemVersionString,
            appVersion: info?["CFBundleShortVersionString"] as? String ?? "unbekannt",
            appBuild: info?["CFBundleVersion"] as? String ?? "0",
            hostname: ProcessInfo.processInfo.hostName,
            processorCount: ProcessInfo.processInfo.processorCount
        )
    }
}
```

## Screenshot (macOS: CGWindowListCreateImage)

```swift
// ScreenshotHelper.swift
import AppKit
import CoreGraphics

struct ScreenshotHelper {
    // Nimmt einen Screenshot aller Fenster auf – nur auf macOS verfügbar
    static func capture() -> String? {
        guard let screen = NSScreen.main else { return nil }
        let bounds = screen.frame
        guard let image = CGWindowListCreateImage(
            bounds,
            .optionOnScreenOnly,
            kCGNullWindowID,
            [.bestResolution]
        ) else { return nil }

        let nsImage = NSImage(cgImage: image, size: bounds.size)
        guard let tiffData = nsImage.tiffRepresentation,
              let bitmap = NSBitmapImageRep(data: tiffData),
              let pngData = bitmap.representation(using: .png, properties: [:])
        else { return nil }

        return pngData.base64EncodedString()
    }
}
```

## FeedbackService

```swift
// FeedbackService.swift
import Foundation

actor FeedbackService {
    private let baseURL = URL(string: "https://feedback.example.com/api")!

    func submitBug(title: String, description: String,
                   severity: String = "medium") async -> Bool {
        let report = buildReport(type: .bug, title: title,
                                 description: description, severity: severity,
                                 screenshot: ScreenshotHelper.capture())
        return await send(report)
    }

    func submitIdea(title: String, description: String) async -> Bool {
        let report = buildReport(type: .idea, title: title,
                                 description: description)
        return await send(report)
    }

    func submitFeedback(title: String, description: String) async -> Bool {
        let report = buildReport(type: .feedback, title: title,
                                 description: description)
        return await send(report)
    }

    private func buildReport(type: FeedbackType, title: String,
                              description: String, severity: String = "medium",
                              screenshot: String? = nil) -> FeedbackReport {
        FeedbackReport(type: type, title: title, description: description,
                       severity: severity, screenshotBase64: screenshot,
                       systemInfo: SystemInfoCollector.collect(),
                       createdAt: Date())
    }

    private func send(_ report: FeedbackReport) async -> Bool {
        var request = URLRequest(url: baseURL.appendingPathComponent("report"))
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        let encoder = JSONEncoder()
        encoder.dateEncodingStrategy = .iso8601
        guard let body = try? encoder.encode(report) else { return false }
        request.httpBody = body
        do {
            let (_, response) = try await URLSession.shared.data(for: request)
            return (response as? HTTPURLResponse)?.statusCode == 201
        } catch {
            return false
        }
    }
}
```

## SwiftUI FeedbackButton (immer sichtbar)

```swift
// FeedbackButton.swift
import SwiftUI

struct FeedbackButton: View {
    @State private var showModal = false
    private let service = FeedbackService()

    var body: some View {
        Button {
            showModal = true
        } label: {
            Label("Feedback", systemImage: "bubble.left.and.bubble.right")
        }
        .keyboardShortcut("f", modifiers: [.command, .shift])
        .sheet(isPresented: $showModal) {
            FeedbackHubView(service: service)
        }
    }
}
```

## FeedbackHubView

```swift
// FeedbackHubView.swift
import SwiftUI

struct FeedbackHubView: View {
    let service: FeedbackService
    @State private var type = FeedbackType.bug
    @State private var title = ""
    @State private var description = ""
    @State private var severity = "medium"
    @State private var sending = false
    @State private var resultMessage: String?
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        Form {
            Section("Art des Feedbacks") {
                Picker("Typ", selection: $type) {
                    ForEach(FeedbackType.allCases, id: \.self) { t in
                        Text(t.rawValue.capitalized).tag(t)
                    }
                }.pickerStyle(.segmented)
            }
            Section("Details") {
                TextField("Titel", text: $title)
                TextEditor(text: $description)
                    .frame(minHeight: 80)
                if type == .bug {
                    Picker("Schweregrad", selection: $severity) {
                        ForEach(["low","medium","high","critical"], id: \.self) {
                            Text($0.capitalized).tag($0)
                        }
                    }
                }
            }
            if let msg = resultMessage {
                Text(msg).foregroundColor(msg.contains("Fehler") ? .red : .green)
            }
        }
        .padding()
        .frame(width: 480, height: 360)
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {
                Button("Abbrechen") { dismiss() }
            }
            ToolbarItem(placement: .confirmationAction) {
                Button("Senden") { Task { await send() } }
                    .disabled(title.isEmpty || sending)
            }
        }
    }

    private func send() async {
        sending = true
        var ok = false
        switch type {
        case .bug: ok = await service.submitBug(title: title, description: description, severity: severity)
        case .idea: ok = await service.submitIdea(title: title, description: description)
        case .feedback: ok = await service.submitFeedback(title: title, description: description)
        }
        resultMessage = ok ? "Erfolgreich gesendet!" : "Fehler – bitte erneut versuchen."
        sending = false
        if ok { try? await Task.sleep(nanoseconds: 1_500_000_000); dismiss() }
    }
}
```

## Info.plist / Entitlements

Für den Netzwerkzugriff muss in `<AppName>.entitlements` gesetzt sein:

```xml
<key>com.apple.security.network.client</key>
<true/>
```

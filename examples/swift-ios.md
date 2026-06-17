# Swift iOS – Feedback Hub Integration

iOS unterscheidet sich grundlegend von macOS: Kein `CGWindowListCreateImage`, Screenshots nur vom eigenen View-Baum, floating Button als UIWindow-Overlay.

## Wichtige iOS-Einschränkungen

- `CGWindowListCreateImage` ist auf iOS **nicht verfügbar** (nur macOS/Catalyst).
- Screenshots im Hintergrund oder fremder Apps sind von Apple verboten.
- Nur `UIGraphicsImageRenderer` auf eigenen Views erlaubt.
- Privacy Manifest (`PrivacyInfo.xcprivacy`) ab iOS 17 / Xcode 15 erforderlich.
- App Store Guideline 5.1.1: Keine Nutzer-Daten ohne explizite Zustimmung.

## FeedbackReport

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
    let screenshotBase64: String?
    let systemInfo: SystemInfo
    let createdAt: Date

    struct SystemInfo: Codable {
        let platform: String
        let osVersion: String
        let deviceModel: String
        let screenWidth: Int
        let screenHeight: Int
        let appVersion: String
    }
}
```

## SystemInfo für iOS

```swift
// SystemInfoiOS.swift
import UIKit

struct SystemInfoiOS {
    static func collect() -> FeedbackReport.SystemInfo {
        let screen = UIScreen.main.bounds
        let info = Bundle.main.infoDictionary
        return FeedbackReport.SystemInfo(
            platform: "iOS",
            osVersion: UIDevice.current.systemVersion,
            deviceModel: UIDevice.current.model,
            screenWidth: Int(screen.width),
            screenHeight: Int(screen.height),
            appVersion: info?["CFBundleShortVersionString"] as? String ?? "?"
        )
    }
}
```

## Screenshot vom eigenen View (UIKit)

```swift
// ScreenshotHelper+iOS.swift
import UIKit

struct ScreenshotHelperiOS {
    // Erfasst nur den übergebenen UIView – kein System-UI, kein fremder App-Inhalt
    static func capture(view: UIView) -> String? {
        let renderer = UIGraphicsImageRenderer(bounds: view.bounds)
        let image = renderer.image { _ in
            view.drawHierarchy(in: view.bounds, afterScreenUpdates: true)
        }
        return image.pngData()?.base64EncodedString()
    }
}
```

## FeedbackService

```swift
// FeedbackService+iOS.swift
import Foundation

actor FeedbackServiceiOS {
    private let baseURL = URL(string: "https://feedback.example.com/api/report")!

    func submit(type: FeedbackType, title: String,
                description: String, screenshotBase64: String? = nil) async -> Bool {
        let report = FeedbackReport(
            type: type, title: title, description: description,
            screenshotBase64: screenshotBase64,
            systemInfo: SystemInfoiOS.collect(),
            createdAt: Date()
        )
        var request = URLRequest(url: baseURL)
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

## Floating Button als UIWindow-Overlay

```swift
// FeedbackOverlayWindow.swift
import UIKit

/// Separates UIWindow, das immer über dem Haupt-Fenster liegt.
/// So bleibt der Button in jeder View-Hierarchie sichtbar.
final class FeedbackOverlayWindow: UIWindow {
    init(scene: UIWindowScene) {
        super.init(windowScene: scene)
        windowLevel = .alert + 1          // immer ganz oben
        isUserInteractionEnabled = true
        backgroundColor = .clear
        let vc = FeedbackButtonViewController()
        rootViewController = vc
        isHidden = false
    }

    required init?(coder: NSCoder) { fatalError() }

    // Klick-Events nur auf dem Button abfangen, Rest durchlassen
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        let hit = super.hitTest(point, with: event)
        return hit == self ? nil : hit
    }
}
```

## FeedbackButtonViewController

```swift
// FeedbackButtonViewController.swift
import UIKit

final class FeedbackButtonViewController: UIViewController {
    private let service = FeedbackServiceiOS()
    private var hostView: UIView? // Referenz auf Root-View für Screenshot

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .clear
        setupButton()
    }

    private func setupButton() {
        let btn = UIButton(type: .system)
        btn.setImage(UIImage(systemName: "bubble.left.fill"), for: .normal)
        btn.backgroundColor = UIColor.systemIndigo
        btn.tintColor = .white
        btn.layer.cornerRadius = 28
        btn.layer.shadowColor = UIColor.black.cgColor
        btn.layer.shadowOpacity = 0.3
        btn.layer.shadowRadius = 6
        btn.translatesAutoresizingMaskIntoConstraints = false
        btn.addTarget(self, action: #selector(openFeedback), for: .touchUpInside)
        view.addSubview(btn)
        NSLayoutConstraint.activate([
            btn.widthAnchor.constraint(equalToConstant: 56),
            btn.heightAnchor.constraint(equalToConstant: 56),
            btn.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            btn.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -100),
        ])
    }

    @objc private func openFeedback() {
        // Screenshot vom eigenen Fenster – Nutzer wurde informiert (z.B. via Onboarding)
        let screenshot = hostView.map { ScreenshotHelperiOS.capture(view: $0) } ?? nil
        let vc = FeedbackViewController(service: service, screenshotBase64: screenshot)
        let nav = UINavigationController(rootViewController: vc)
        nav.modalPresentationStyle = .formSheet
        present(nav, animated: true)
    }
}
```

## FeedbackViewController (UIKit)

```swift
// FeedbackViewController.swift
import UIKit

final class FeedbackViewController: UIViewController {
    private let service: FeedbackServiceiOS
    private let screenshotBase64: String?
    private var selectedType: FeedbackType = .bug

    private let titleField = UITextField()
    private let descView = UITextView()
    private let segmentControl = UISegmentedControl(
        items: FeedbackType.allCases.map { $0.rawValue.capitalized }
    )

    init(service: FeedbackServiceiOS, screenshotBase64: String?) {
        self.service = service
        self.screenshotBase64 = screenshotBase64
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError() }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Feedback"
        view.backgroundColor = .systemBackground
        navigationItem.leftBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .cancel, target: self, action: #selector(cancel))
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "Senden", style: .done, target: self, action: #selector(send))
        setupUI()
    }

    private func setupUI() { /* Auto-Layout für segmentControl, titleField, descView */ }

    @objc private func cancel() { dismiss(animated: true) }

    @objc private func send() {
        guard let title = titleField.text, !title.isEmpty else { return }
        Task {
            let ok = await service.submit(
                type: selectedType, title: title,
                description: descView.text ?? "",
                screenshotBase64: screenshotBase64
            )
            await MainActor.run {
                dismiss(animated: true) {
                    // Zeige Ergebnis in der aufrufenden UI
                    _ = ok
                }
            }
        }
    }
}
```

## Privacy Manifest (PrivacyInfo.xcprivacy)

```xml
<!-- ios/Runner/PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<plist version="1.0">
<dict>
  <key>NSPrivacyCollectedDataTypes</key>
  <array>
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypeDeviceID</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <false/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
      </array>
    </dict>
  </array>
</dict>
</plist>
```

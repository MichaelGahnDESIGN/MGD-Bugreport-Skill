# Beispiel: Swift iOS (App Store)

iOS-Apps werden über den App Store verteilt. Ein eigener Installer-Updater ist nicht erlaubt. Das eigene Backend dient für Mindestversionscheck und Remote-Konfiguration.

---

## Was möglich ist

- Mindestversionscheck → Nutzer zum App Store leiten
- Remote-Konfiguration (Feature-Flags, API-Endpunkte, Maintenance-Modus)
- Content-Updates (Daten, Assets — kein nativer Code)

---

## UpdateChecker.swift

```swift
import UIKit

struct AppVersionManifest: Codable {
    let minimumVersion: String
    let latestVersion: String
    let storeUrl: String
    let forceUpdate: Bool
    let maintenanceMode: Bool
    let maintenanceMessage: String?
}

class UpdateChecker {
    static let shared = UpdateChecker()

    private let manifestURL = URL(string: "https://updates.example.com/example-app/ios/latest.json")!
    private var currentVersion: String {
        Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "0.0.0"
    }

    func checkOnLaunch(from viewController: UIViewController) {
        Task {
            await check(from: viewController)
        }
    }

    private func check(from vc: UIViewController) async {
        guard let manifest = await fetchManifest() else { return }

        if manifest.maintenanceMode {
            await MainActor.run { showMaintenanceAlert(vc, message: manifest.maintenanceMessage) }
            return
        }

        if isBelowMinimum(manifest.minimumVersion) {
            await MainActor.run { showForceUpdateAlert(vc, storeUrl: manifest.storeUrl) }
        }
    }

    private func fetchManifest() async -> AppVersionManifest? {
        do {
            let (data, response) = try await URLSession.shared.data(from: manifestURL)
            guard let http = response as? HTTPURLResponse, http.statusCode == 200 else { return nil }
            return try JSONDecoder().decode(AppVersionManifest.self, from: data)
        } catch { return nil }
    }

    private func isBelowMinimum(_ minimum: String) -> Bool {
        currentVersion.compare(minimum, options: .numeric) == .orderedAscending
    }

    private func showForceUpdateAlert(_ vc: UIViewController, storeUrl: String) {
        let alert = UIAlertController(
            title: "Update erforderlich",
            message: "Bitte aktualisiere die App um fortzufahren.",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "App Store öffnen", style: .default) { _ in
            if let url = URL(string: storeUrl) { UIApplication.shared.open(url) }
        })
        vc.present(alert, animated: true)
    }

    private func showMaintenanceAlert(_ vc: UIViewController, message: String?) {
        let alert = UIAlertController(
            title: "Wartungsarbeiten",
            message: message ?? "Bitte versuche es später erneut.",
            preferredStyle: .alert
        )
        vc.present(alert, animated: true)
    }
}
```

---

## Manifest für iOS

```json
{
  "app": "example-app",
  "platform": "ios",
  "latestVersion": "2.3.0",
  "minimumVersion": "2.0.0",
  "storeUrl": "https://apps.apple.com/app/idXXXXXXXXX",
  "forceUpdate": false,
  "maintenanceMode": false,
  "maintenanceMessage": null,
  "publishedAt": "2026-06-17"
}
```

---

## App Store Regeln

- Kein Herunterladen und Ausführen von nativem Code
- Kein Remote-Updating von Swift/Objective-C Code
- JavaScript-Bundles über React Native/Expo OTA: erlaubt mit Einschränkungen
- In-App-Purchases über eigene Systeme: Nicht erlaubt (App Store muss genutzt werden)

→ Checkliste: [`../checklists/ios.md`](../checklists/ios.md)

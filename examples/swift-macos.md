# Beispiel: Swift macOS (Direktdownload)

Für macOS-Apps die über Direktdownload (nicht App Store) verteilt werden und in Swift geschrieben sind.

---

## Voraussetzungen

- Apple Developer Account
- Developer ID Application Zertifikat
- Code-Signierung und Notarisierung eingerichtet

---

## Projektstruktur

```text
YourApp/
  Services/
    UpdateService.swift       — Update-Prüfung
    UpdateManifest.swift      — Manifest-Modell
  Views/
    UpdateAlertView.swift     — SwiftUI Update-Dialog
```

---

## UpdateManifest.swift

```swift
import Foundation

struct UpdateManifest: Codable {
    let app: String
    let platform: String
    let latestVersion: String
    let minimumVersion: String
    let downloadUrl: String
    let changelog: [String]
    let forceUpdate: Bool
    let sha256: String?
    let publishedAt: String?
}
```

---

## UpdateService.swift

```swift
import Foundation

@MainActor
final class UpdateService: ObservableObject {
    static let shared = UpdateService()

    @Published var availableUpdate: UpdateManifest?

    private let manifestURL = URL(string: "https://updates.example.com/example-app/macos/latest.json")!
    private var currentVersion: String {
        Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "0.0.0"
    }

    func checkForUpdate() async {
        guard let manifest = await fetchManifest() else { return }
        if shouldUpdate(manifest: manifest) {
            availableUpdate = manifest
        }
    }

    func openDownloadPage() {
        guard let urlString = availableUpdate?.downloadUrl,
              let url = URL(string: urlString) else { return }
        NSWorkspace.shared.open(url)
    }

    private func fetchManifest() async -> UpdateManifest? {
        do {
            let (data, response) = try await URLSession.shared.data(from: manifestURL)
            guard let http = response as? HTTPURLResponse, http.statusCode == 200 else { return nil }
            return try JSONDecoder().decode(UpdateManifest.self, from: data)
        } catch {
            return nil
        }
    }

    private func shouldUpdate(manifest: UpdateManifest) -> Bool {
        let latest = manifest.latestVersion
        let minimum = manifest.minimumVersion
        return compareVersions(latest, currentVersion) > 0 ||
               compareVersions(currentVersion, minimum) < 0
    }

    private func compareVersions(_ a: String, _ b: String) -> Int {
        a.compare(b, options: .numeric).rawValue
    }
}
```

---

## UpdateAlertView.swift (SwiftUI)

```swift
import SwiftUI

struct UpdateAlertView: View {
    let manifest: UpdateManifest
    let onDownload: () -> Void
    let onDismiss: () -> Void

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("Update verfügbar — Version \(manifest.latestVersion)")
                .font(.headline)

            Text("Installiert: \(installedVersion)")
                .font(.subheadline)
                .foregroundStyle(.secondary)

            Divider()

            Text("Änderungen:")
                .font(.subheadline).bold()

            ForEach(manifest.changelog, id: \.self) { entry in
                Label(entry, systemImage: "checkmark.circle")
                    .font(.caption)
            }

            HStack {
                if !manifest.forceUpdate {
                    Button("Später", action: onDismiss)
                }
                Spacer()
                Button("Jetzt herunterladen", action: onDownload)
                    .buttonStyle(.borderedProminent)
            }
        }
        .padding()
        .frame(width: 380)
    }

    private var installedVersion: String {
        Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "—"
    }
}
```

---

## App-Start Integration

```swift
@main
struct ExampleApp: App {
    @StateObject private var updater = UpdateService.shared

    var body: some Scene {
        WindowGroup {
            ContentView()
                .task { await updater.checkForUpdate() }
                .sheet(item: $updater.availableUpdate) { manifest in
                    UpdateAlertView(
                        manifest: manifest,
                        onDownload: { updater.openDownloadPage() },
                        onDismiss: { updater.availableUpdate = nil }
                    )
                }
        }
    }
}
```

→ Checkliste: [`../checklists/macos.md`](../checklists/macos.md)

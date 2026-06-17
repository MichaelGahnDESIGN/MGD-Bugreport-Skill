# Beispiel: Unity

Unity-Spiele brauchen in der Regel zwei Update-Schichten: Client-Update (Unity-Player/Executable) und Content-Update (AssetBundles, Addressables).

---

## Die zwei Schichten

```
Unity-Spiel
├── Client-Update (Unity-Executable)
│   → Neue Engine-Version, C#-Code-Änderungen
│   → Muss als neues Build ausgeliefert werden
│   → Wie normale Desktop/Mobile-App
│
└── Content-Update (AssetBundles / Addressables)
    → Neue Level, Assets, Prefabs, Sounds
    → Ohne Neuinstallation möglich
    → Über Addressables oder eigenen Bundle-Server
```

---

## Client-Update-Manifest

```json
{
  "app": "example-game",
  "platform": "windows",
  "latestVersion": "1.3.0",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-game/releases/example-game-1.3.0-windows.exe",
  "sha256": "a3f5b2c8d1e4f7a0b9c2d5e8f1a4b7c0d3e6f9a2b5c8d1e4f7a0b3c6d9e2f5a8",
  "changelog": [
    "Neues Gebiet: Eiswüste",
    "Balancing-Update für Gegner",
    "Performance-Verbesserungen"
  ],
  "forceUpdate": false
}
```

---

## Update-Check in Unity (C#)

```csharp
using System;
using System.Collections;
using UnityEngine;
using UnityEngine.Networking;

[Serializable]
public class UpdateManifest {
    public string app;
    public string platform;
    public string latestVersion;
    public string minimumVersion;
    public string downloadUrl;
    public string[] changelog;
    public bool forceUpdate;
}

public class UpdateService : MonoBehaviour {
    private const string ManifestUrl =
        "https://updates.example.com/example-game/windows/latest.json";

    private const string CurrentVersion = "1.2.0";

    public void CheckForUpdate(Action<UpdateManifest> onUpdateFound) {
        StartCoroutine(FetchManifest(onUpdateFound));
    }

    private IEnumerator FetchManifest(Action<UpdateManifest> callback) {
        using var request = UnityWebRequest.Get(ManifestUrl);
        request.timeout = 10;
        yield return request.SendWebRequest();

        if (request.result != UnityWebRequest.Result.Success) yield break;

        var manifest = JsonUtility.FromJson<UpdateManifest>(request.downloadHandler.text);

        if (IsNewer(manifest.latestVersion) || IsBelowMinimum(manifest.minimumVersion)) {
            callback?.Invoke(manifest);
        }
    }

    private bool IsNewer(string remote) => CompareVersions(remote, CurrentVersion) > 0;
    private bool IsBelowMinimum(string minimum) => CompareVersions(CurrentVersion, minimum) < 0;

    private static int CompareVersions(string a, string b) =>
        string.Compare(a, b, StringComparison.OrdinalIgnoreCase);
    // Für echtes Semver: Version-Parsing implementieren
}
```

---

## Content-Updates mit Addressables

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

public class ContentUpdater : MonoBehaviour {
    public IEnumerator CheckAndUpdateContent() {
        // Catalog aktualisieren
        var checkHandle = Addressables.CheckForCatalogUpdates();
        yield return checkHandle;

        if (checkHandle.Status != AsyncOperationStatus.Succeeded) yield break;

        var catalogs = checkHandle.Result;
        if (catalogs.Count > 0) {
            var updateHandle = Addressables.UpdateCatalogs(catalogs);
            yield return updateHandle;
            Debug.Log("Content-Update abgeschlossen");
        }
    }
}
```

---

## Wichtige Entscheidungen

1. Werden Updates über Steam, Epic oder direkt ausgeliefert?
2. Wird Addressables für Content-Updates genutzt?
3. Auf welchen Plattformen läuft das Spiel (PC, macOS, Konsole)?
4. Gibt es ein Launcher-System?

→ Wiki: [`../wiki/12-Spiele-und-Content-Updates.md`](../wiki/12-Spiele-und-Content-Updates.md)

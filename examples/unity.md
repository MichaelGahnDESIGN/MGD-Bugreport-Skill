# Unity C# – Feedback Hub Integration

Feedback-Hub-Integration für Unity-Spiele und -Apps: Screenshot via `ScreenCapture`, floating Canvas-Button mit Sorting Order 999, UnityWebRequest für den API-Call.

## BugReport Datenklasse

```csharp
// Assets/FeedbackHub/BugReport.cs
using System;
using UnityEngine;

namespace FeedbackHub
{
    [Serializable]
    public class BugReport
    {
        public string type;           // "bug" | "idea" | "feedback"
        public string title;
        public string description;
        public string severity;       // "low" | "medium" | "high" | "critical"
        public string screenshotBase64;
        public SystemInfoData systemInfo;
        public string createdAt;      // ISO 8601

        [Serializable]
        public class SystemInfoData
        {
            public string os;
            public string deviceModel;
            public string processorType;
            public int systemMemoryMB;
            public string graphicsDevice;
            public string appVersion;
            public int screenWidth;
            public int screenHeight;
        }
    }
}
```

## SystemInfo-Hilfsmethode

```csharp
// Assets/FeedbackHub/SystemInfoCollector.cs
using UnityEngine;
using FeedbackHub;

namespace FeedbackHub
{
    public static class SystemInfoCollector
    {
        public static BugReport.SystemInfoData Collect() =>
            new BugReport.SystemInfoData
            {
                os              = SystemInfo.operatingSystem,
                deviceModel     = SystemInfo.deviceModel,
                processorType   = SystemInfo.processorType,
                systemMemoryMB  = SystemInfo.systemMemorySize,
                graphicsDevice  = SystemInfo.graphicsDeviceName,
                appVersion      = Application.version,
                screenWidth     = Screen.currentResolution.width,
                screenHeight    = Screen.currentResolution.height,
            };
    }
}
```

## FeedbackService MonoBehaviour

```csharp
// Assets/FeedbackHub/FeedbackService.cs
using System;
using System.Collections;
using System.Text;
using UnityEngine;
using UnityEngine.Networking;

namespace FeedbackHub
{
    public class FeedbackService : MonoBehaviour
    {
        private const string ApiUrl = "https://feedback.example.com/api/report";

        public IEnumerator SubmitBug(string title, string description,
                                     string severity = "medium",
                                     Action<bool> callback = null)
        {
            yield return StartCoroutine(CaptureAndSubmit(
                new BugReport { type = "bug", title = title,
                    description = description, severity = severity },
                callback));
        }

        public IEnumerator SubmitIdea(string title, string description,
                                      Action<bool> callback = null)
        {
            yield return StartCoroutine(Submit(
                new BugReport { type = "idea", title = title,
                    description = description },
                screenshotBase64: null, callback));
        }

        public IEnumerator SubmitFeedback(string title, string description,
                                          Action<bool> callback = null)
        {
            yield return StartCoroutine(Submit(
                new BugReport { type = "feedback", title = title,
                    description = description },
                screenshotBase64: null, callback));
        }

        private IEnumerator CaptureAndSubmit(BugReport report,
                                              Action<bool> callback)
        {
            // Screenshot muss am Ende eines Frames aufgenommen werden
            yield return new WaitForEndOfFrame();

            string screenshotBase64 = null;
            try
            {
                var tex = ScreenCapture.CaptureScreenshotAsTexture();
                screenshotBase64 = Convert.ToBase64String(
                    tex.EncodeToPNG());
                Destroy(tex);
            }
            catch (Exception e)
            {
                Debug.LogWarning($"Screenshot fehlgeschlagen: {e.Message}");
            }

            yield return StartCoroutine(Submit(report, screenshotBase64, callback));
        }

        private IEnumerator Submit(BugReport report, string screenshotBase64,
                                    Action<bool> callback)
        {
            report.systemInfo      = SystemInfoCollector.Collect();
            report.screenshotBase64 = screenshotBase64;
            report.createdAt       = DateTime.UtcNow.ToString("o");

            var json = JsonUtility.ToJson(report);
            var bytes = Encoding.UTF8.GetBytes(json);

            using var request = new UnityWebRequest(ApiUrl, "POST");
            request.uploadHandler   = new UploadHandlerRaw(bytes);
            request.downloadHandler = new DownloadHandlerBuffer();
            request.SetRequestHeader("Content-Type", "application/json");

            yield return request.SendWebRequest();

            bool success = request.result == UnityWebRequest.Result.Success
                           && request.responseCode == 201;

            if (!success)
                Debug.LogError($"Feedback-Fehler: {request.error}");

            callback?.Invoke(success);
        }
    }
}
```

## Floating Button – Canvas Setup

```csharp
// Assets/FeedbackHub/FeedbackHubUI.cs
using UnityEngine;
using UnityEngine.UI;
using TMPro;

namespace FeedbackHub
{
    /// <summary>
    /// Erstellt einen Canvas mit Sorting Order 999 (immer über allem).
    /// Per Code instanziiert – kein Prefab nötig.
    /// </summary>
    public class FeedbackHubUI : MonoBehaviour
    {
        [SerializeField] private FeedbackService feedbackService;

        private Canvas _canvas;
        private GameObject _modal;
        private TMP_InputField _titleInput;
        private TMP_InputField _descInput;
        private TMP_Dropdown _typeDropdown;
        private string _currentType = "bug";

        private void Awake()
        {
            CreateCanvas();
            CreateFloatingButton();
            CreateModal();
        }

        private void CreateCanvas()
        {
            var go = new GameObject("FeedbackCanvas");
            DontDestroyOnLoad(go);

            _canvas = go.AddComponent<Canvas>();
            _canvas.renderMode = RenderMode.ScreenSpaceOverlay;
            _canvas.sortingOrder = 999;   // immer über allem

            go.AddComponent<CanvasScaler>().uiScaleMode =
                CanvasScaler.ScaleMode.ScaleWithScreenSize;
            go.AddComponent<GraphicRaycaster>();
        }

        private void CreateFloatingButton()
        {
            // Button-GameObject im Canvas erstellen
            var btnGo = new GameObject("FeedbackBtn");
            btnGo.transform.SetParent(_canvas.transform, false);

            var rect = btnGo.AddComponent<RectTransform>();
            rect.anchorMin = rect.anchorMax = new Vector2(1, 0);
            rect.pivot = new Vector2(1, 0);
            rect.anchoredPosition = new Vector2(-20, 20);
            rect.sizeDelta = new Vector2(140, 44);

            var img = btnGo.AddComponent<Image>();
            img.color = new Color(0.31f, 0.27f, 0.9f);

            var btn = btnGo.AddComponent<Button>();
            btn.onClick.AddListener(OpenModal);

            // Label
            var labelGo = new GameObject("Label");
            labelGo.transform.SetParent(btnGo.transform, false);
            var label = labelGo.AddComponent<TextMeshProUGUI>();
            label.text = "💬 Feedback";
            label.alignment = TextAlignmentOptions.Center;
            label.fontSize = 14;
            label.GetComponent<RectTransform>().sizeDelta = Vector2.zero;
            label.GetComponent<RectTransform>().anchorMin = Vector2.zero;
            label.GetComponent<RectTransform>().anchorMax = Vector2.one;
        }

        private void CreateModal()
        {
            // Vereinfacht: Modal-Panel mit Titel-Input, Desc-Input, Dropdown, Buttons
            _modal = new GameObject("FeedbackModal");
            _modal.transform.SetParent(_canvas.transform, false);
            _modal.SetActive(false);
            // (Vollständiges UI-Setup würde den Rahmen sprengen –
            //  in der Praxis: Unity UI Prefab verwenden)
        }

        public void OpenModal() => _modal.SetActive(true);
        public void CloseModal() => _modal.SetActive(false);

        public void OnSendClicked()
        {
            string title = _titleInput?.text ?? "";
            string desc = _descInput?.text ?? "";
            if (string.IsNullOrWhiteSpace(title)) return;

            StartCoroutine(feedbackService.SubmitBug(title, desc,
                callback: ok => {
                    CloseModal();
                    Debug.Log(ok ? "Feedback gesendet!" : "Fehler beim Senden.");
                }));
        }
    }
}
```

## Verwendung im Projekt

```csharp
// Irgendwo in deinem GameManager / App-Startup:
var hub = new GameObject("FeedbackHub");
var svc = hub.AddComponent<FeedbackService>();
var ui  = hub.AddComponent<FeedbackHubUI>();
// feedbackService-Referenz im Inspector oder per Code setzen
DontDestroyOnLoad(hub);
```

## Hinweise

- `ScreenCapture.CaptureScreenshotAsTexture()` erfordert `WaitForEndOfFrame()`.
- Auf WebGL ist `ScreenCapture` eingeschränkt – dort lieber deaktivieren.
- `DontDestroyOnLoad` sorgt dafür, dass der Button bei Szenen-Wechseln erhalten bleibt.
- `UnityWebRequest` läuft asynchron via Coroutine – kein Einfrieren der UI.

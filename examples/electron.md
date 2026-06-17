# Electron – Feedback Hub Integration

Screenshot via `desktopCapturer` im Main-Prozess, IPC-Kommunikation zum Renderer, floating Button mit `position:fixed`.

## Projektstruktur

```
example-app/
├── main.js          ← Electron Main-Prozess
├── preload.js       ← Context Bridge
└── renderer/
    ├── feedback.js  ← FeedbackHub Klasse
    └── feedback.html
```

## Main-Prozess: Screenshot + IPC Handler

```javascript
// main.js
const { app, BrowserWindow, ipcMain, desktopCapturer } = require('electron');

let mainWindow;

app.whenReady().then(() => {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: `${__dirname}/preload.js`,
      contextIsolation: true,
      nodeIntegration: false,
    },
  });
  mainWindow.loadFile('renderer/index.html');
});

// Screenshot des gesamten Bildschirms aufnehmen
ipcMain.handle('feedback:screenshot', async () => {
  try {
    const sources = await desktopCapturer.getSources({
      types: ['screen'],
      thumbnailSize: { width: 1920, height: 1080 },
    });
    const primary = sources[0];
    if (!primary) return null;
    // thumbnail ist ein NativeImage – in Base64 konvertieren
    return primary.thumbnail.toDataURL(); // "data:image/png;base64,..."
  } catch (err) {
    console.error('Screenshot fehlgeschlagen:', err);
    return null;
  }
});

// Report an Backend weiterleiten (aus Renderer via IPC)
ipcMain.handle('feedback:submit', async (_event, report) => {
  try {
    const response = await fetch('https://feedback.example.com/api/report', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(report),
    });
    return response.status === 201;
  } catch {
    return false;
  }
});
```

## Preload: Context Bridge

```javascript
// preload.js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('feedbackBridge', {
  takeScreenshot: () => ipcRenderer.invoke('feedback:screenshot'),
  submitReport: (report) => ipcRenderer.invoke('feedback:submit', report),
  getAppVersion: () => process.env.npm_package_version || 'unbekannt',
  getPlatform: () => process.platform,
  getArch: () => process.arch,
});
```

## Renderer: FeedbackHub Klasse

```javascript
// renderer/feedback.js
class FeedbackHub {
  constructor() {
    this.isOpen = false;
    this._injectStyles();
    this._createButton();
    this._createModal();
  }

  // System-Infos aus dem Renderer sammeln
  _getSystemInfo() {
    return {
      platform: window.feedbackBridge.getPlatform(),
      arch: window.feedbackBridge.getArch(),
      app_version: window.feedbackBridge.getAppVersion(),
      user_agent: navigator.userAgent,
      language: navigator.language,
      screen_width: screen.width,
      screen_height: screen.height,
    };
  }

  async submitBug(title, description, severity = 'medium') {
    return this._submit({ type: 'bug', title, description, severity });
  }

  async submitIdea(title, description) {
    return this._submit({ type: 'idea', title, description });
  }

  async submitFeedback(title, description) {
    return this._submit({ type: 'feedback', title, description });
  }

  async _submit(data) {
    // Screenshot vor dem Absenden aufnehmen
    const screenshotDataUrl = await window.feedbackBridge.takeScreenshot();
    const report = {
      ...data,
      screenshot: screenshotDataUrl,
      system_info: this._getSystemInfo(),
      created_at: new Date().toISOString(),
    };
    return window.feedbackBridge.submitReport(report);
  }

  _injectStyles() {
    const style = document.createElement('style');
    style.textContent = `
      #feedback-btn {
        position: fixed;
        bottom: 20px;
        right: 20px;
        z-index: 99999;
        background: #4f46e5;
        color: white;
        border: none;
        border-radius: 28px;
        padding: 12px 20px;
        font-size: 14px;
        cursor: pointer;
        box-shadow: 0 4px 12px rgba(0,0,0,0.3);
        display: flex;
        align-items: center;
        gap: 8px;
      }
      #feedback-btn:hover { background: #4338ca; }

      #feedback-overlay {
        display: none;
        position: fixed;
        inset: 0;
        background: rgba(0,0,0,0.5);
        z-index: 99998;
        align-items: center;
        justify-content: center;
      }
      #feedback-overlay.open { display: flex; }

      #feedback-modal {
        background: white;
        border-radius: 12px;
        padding: 24px;
        width: 480px;
        box-shadow: 0 20px 60px rgba(0,0,0,0.3);
      }
      #feedback-modal h2 { margin: 0 0 16px; font-size: 18px; }
      #feedback-modal select,
      #feedback-modal input,
      #feedback-modal textarea {
        width: 100%; box-sizing: border-box;
        padding: 8px; margin-bottom: 12px;
        border: 1px solid #d1d5db; border-radius: 6px;
        font-size: 14px;
      }
      #feedback-modal textarea { height: 100px; resize: vertical; }
      .feedback-actions { display: flex; justify-content: flex-end; gap: 8px; }
      .feedback-actions button {
        padding: 8px 16px; border-radius: 6px;
        border: none; cursor: pointer; font-size: 14px;
      }
      .btn-cancel { background: #f3f4f6; color: #374151; }
      .btn-submit { background: #4f46e5; color: white; }
    `;
    document.head.appendChild(style);
  }

  _createButton() {
    const btn = document.createElement('button');
    btn.id = 'feedback-btn';
    btn.innerHTML = '💬 Feedback';
    btn.addEventListener('click', () => this.open());
    document.body.appendChild(btn);
  }

  _createModal() {
    const overlay = document.createElement('div');
    overlay.id = 'feedback-overlay';
    overlay.innerHTML = `
      <div id="feedback-modal">
        <h2>Feedback senden</h2>
        <select id="fb-type">
          <option value="bug">Bug melden</option>
          <option value="idea">Idee vorschlagen</option>
          <option value="feedback">Allgemeines Feedback</option>
        </select>
        <input id="fb-title" type="text" placeholder="Titel" />
        <textarea id="fb-desc" placeholder="Beschreibung"></textarea>
        <div class="feedback-actions">
          <button class="btn-cancel" id="fb-cancel">Abbrechen</button>
          <button class="btn-submit" id="fb-send">Senden</button>
        </div>
      </div>
    `;
    overlay.querySelector('#fb-cancel').addEventListener('click', () => this.close());
    overlay.querySelector('#fb-send').addEventListener('click', () => this._handleSend());
    document.body.appendChild(overlay);
  }

  open() { document.getElementById('feedback-overlay').classList.add('open'); }
  close() { document.getElementById('feedback-overlay').classList.remove('open'); }

  async _handleSend() {
    const type = document.getElementById('fb-type').value;
    const title = document.getElementById('fb-title').value.trim();
    const desc = document.getElementById('fb-desc').value.trim();
    if (!title) return alert('Bitte einen Titel eingeben.');

    const btn = document.getElementById('fb-send');
    btn.textContent = 'Sende...';
    btn.disabled = true;
    const ok = await this._submit({ type, title, description: desc });
    this.close();
    alert(ok ? 'Feedback gesendet! Danke.' : 'Fehler – bitte erneut versuchen.');
    btn.textContent = 'Senden';
    btn.disabled = false;
  }
}

// Initialisierung
const feedbackHub = new FeedbackHub();
```

## HTML-Einbindung

```html
<!-- renderer/index.html (am Ende vor </body>) -->
<script src="feedback.js"></script>
```

## Hinweis: desktopCapturer Berechtigungen

Unter macOS muss in `Info.plist` die Bildschirmaufnahme-Berechtigung gesetzt sein. Bei Electron-Apps greift das macOS Privacy-System – der Nutzer muss die App unter *Systemeinstellungen → Datenschutz → Bildschirmaufnahme* erlauben.

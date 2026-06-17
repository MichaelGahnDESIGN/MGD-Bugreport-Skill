# Beispiel: Electron

Electron hat einen eingebauten `autoUpdater` (basiert auf Squirrel). Das häufiger verwendete Package ist `electron-updater` aus `electron-builder`, da es mehr Plattformen und Konfigurationsoptionen bietet.

---

## Wichtiger Hinweis zuerst

Electron-Updater und Squirrel erfordern auf macOS und Windows Code-Signierung. **Ohne Signierung funktioniert der Auto-Updater nicht.**

Mit der Planung und Signierung beginnen, bevor Update-Code geschrieben wird.

---

## Empfohlene Struktur

```text
src/
  main/
    updater.js         — Update-Logik
    main.js            — App-Einstiegspunkt

package.json           — electron-builder Konfiguration
```

---

## updater.js (Phase 1 — manuell)

```javascript
const { shell } = require('electron');
const https = require('https');

const MANIFEST_URL = 'https://updates.example.com/example-app/';
const CURRENT_VERSION = '1.0.0';

function getPlatform() {
  if (process.platform === 'darwin') return 'macos';
  if (process.platform === 'win32') return 'windows';
  return 'linux';
}

function compareVersions(a, b) {
  const pa = a.split('.').map(Number);
  const pb = b.split('.').map(Number);
  for (let i = 0; i < 3; i++) {
    const diff = (pa[i] || 0) - (pb[i] || 0);
    if (diff !== 0) return diff;
  }
  return 0;
}

async function checkForUpdate() {
  return new Promise((resolve) => {
    const url = `${MANIFEST_URL}${getPlatform()}/latest.json`;
    https.get(url, (res) => {
      let data = '';
      res.on('data', (chunk) => { data += chunk; });
      res.on('end', () => {
        try {
          const manifest = JSON.parse(data);
          const hasUpdate = compareVersions(manifest.latestVersion, CURRENT_VERSION) > 0;
          resolve(hasUpdate ? manifest : null);
        } catch {
          resolve(null);
        }
      });
    }).on('error', () => resolve(null));
  });
}

function openDownloadPage(url) {
  shell.openExternal(url);
}

module.exports = { checkForUpdate, openDownloadPage };
```

---

## electron-builder Konfiguration (Phase 3)

```json
{
  "build": {
    "publish": [
      {
        "provider": "generic",
        "url": "https://updates.example.com/example-app/"
      }
    ]
  }
}
```

---

## Wichtige Fragen vor Phase 3

1. Ist Code-Signierung für macOS und Windows vorhanden?
2. Welche Kanäle sollen unterstützt werden (stable, beta)?
3. Sollen Updates sofort installiert werden oder beim nächsten Start?
4. Wie soll mit Nutzern umgegangen werden die gerade arbeiten?

→ Checkliste: [`../checklists/phase-1.md`](../checklists/phase-1.md)

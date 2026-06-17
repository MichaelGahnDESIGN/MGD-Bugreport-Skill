# Beispiel: Node.js Update-API

Node.js eignet sich für leichtgewichtige Update-APIs die schnell deploybar sind und gut mit modernen Hosting-Plattformen (Vercel, Railway, Fly.io) funktionieren.

---

## Express.js Minimal-API

```javascript
// server.js
import express from 'express';
import { readFileSync, existsSync } from 'fs';
import { join, resolve } from 'path';

const app = express();
const PORT = process.env.PORT ?? 3000;

const ALLOWED_APPS = new Set(['example-app']);
const ALLOWED_PLATFORMS = new Set(['macos', 'windows', 'linux']);

// Sicherheit: Input-Validierung
function validateParams(app, platform) {
    if (!ALLOWED_APPS.has(app)) return false;
    if (!ALLOWED_PLATFORMS.has(platform)) return false;
    return true;
}

// Update-Prüfung
app.get('/updates/:app/:platform', (req, res) => {
    const { app, platform } = req.params;

    if (!validateParams(app, platform)) {
        return res.status(400).json({ error: 'Invalid parameters' });
    }

    // Pfad-Traversal verhindern — nie direkt req.params in Pfade einbauen
    const releasePath = join(
        resolve('./releases'),
        app,
        `${platform}.json`
    );

    if (!releasePath.startsWith(resolve('./releases'))) {
        return res.status(400).json({ error: 'Invalid path' });
    }

    if (!existsSync(releasePath)) {
        return res.status(404).json({ error: 'No release found' });
    }

    try {
        const release = JSON.parse(readFileSync(releasePath, 'utf8'));
        res.json(release);
    } catch {
        res.status(500).json({ error: 'Internal error' });
    }
});

app.listen(PORT, () => console.log(`Update API running on port ${PORT}`));
```

---

## Dateistruktur

```text
releases/
  example-app/
    macos.json
    windows.json
    linux.json
server.js
package.json
.env
```

---

## package.json

```json
{
  "name": "update-api",
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

---

## Mit Rate-Limiting (express-rate-limit)

```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
    windowMs: 60 * 1000,   // 1 Minute
    max: 100,              // Max 100 Anfragen pro Minute pro IP
    standardHeaders: true,
    legacyHeaders: false,
});

app.use('/updates', limiter);
```

---

## Deployment-Optionen

| Plattform | Geeignet für |
|-----------|-------------|
| Railway | Schnelles Deployment, Docker-Support |
| Fly.io | Edge-Deployment, global |
| Vercel | Serverless Functions (Edge) |
| VPS/Root-Server | Volle Kontrolle, nginx davor |

---

## Sicherheitsregeln

- Pfad-Traversal explizit verhindern (wie im Beispiel gezeigt)
- Whitelist für App-IDs und Plattformen
- Rate-Limiting aktivieren
- HTTPS erzwingen (HTTPS-Terminierung am Loadbalancer oder Reverse-Proxy)
- Secrets in Umgebungsvariablen (`.env`), nie im Code
- Fehler-Responses keine internen Pfade enthalten

→ Wiki: [`../wiki/14-Self-Hosted-Updates.md`](../wiki/14-Self-Hosted-Updates.md)

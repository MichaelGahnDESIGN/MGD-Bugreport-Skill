# Node.js / Express Backend – Feedback Hub API

Express-API mit Multer (Screenshot-Upload), Input-Validierung, SQLite-Datenbank, GitHub-Integration und Rate-Limiting.

## Projektstruktur

```
api-server/
├── src/
│   ├── index.js          ← Express-App Einstiegspunkt
│   ├── routes/
│   │   └── reports.js    ← POST /api/reports
│   ├── middleware/
│   │   └── upload.js     ← Multer-Konfiguration
│   ├── db.js             ← SQLite Datenbankverbindung
│   └── services/
│       └── github.js     ← GitHub Issue-Erstellung
├── uploads/              ← Screenshot-Dateien (außerhalb public/)
└── package.json
```

## package.json – Abhängigkeiten

```json
{
  "name": "feedback-hub-api",
  "version": "1.0.0",
  "type": "module",
  "dependencies": {
    "better-sqlite3": "^9.4.3",
    "cors": "^2.8.5",
    "express": "^4.19.2",
    "express-rate-limit": "^7.3.1",
    "express-validator": "^7.1.0",
    "multer": "^1.4.5-lts.1",
    "uuid": "^10.0.0"
  }
}
```

## db.js – SQLite-Datenbankverbindung

```javascript
// src/db.js
import Database from 'better-sqlite3';
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

const __dir = dirname(fileURLToPath(import.meta.url));
const db = new Database(join(__dir, '..', 'feedback.sqlite'));

db.pragma('journal_mode = WAL');
db.pragma('foreign_keys = ON');

// Tabellen anlegen
db.exec(`
  CREATE TABLE IF NOT EXISTS reports (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    type            TEXT NOT NULL CHECK(type IN ('bug','idea','feedback')),
    title           TEXT NOT NULL,
    description     TEXT DEFAULT '',
    severity        TEXT DEFAULT 'medium',
    screenshot_path TEXT,
    system_info     TEXT,        -- JSON-String
    github_issue    TEXT,
    ip_hash         TEXT,
    created_at      TEXT DEFAULT (datetime('now'))
  );
`);

export default db;
```

## middleware/upload.js – Multer-Konfiguration

```javascript
// src/middleware/upload.js
import multer from 'multer';
import { randomUUID } from 'crypto';
import { join, extname, basename } from 'path';
import { fileURLToPath } from 'url';

const __dir = dirname(fileURLToPath(import.meta.url));

// Pfad-Traversal verhindern: Dateiname nur aus UUID + sicherer Endung
const storage = multer.diskStorage({
  destination: join(__dir, '..', '..', 'uploads'),
  filename: (_req, file, cb) => {
    const ext = ['.jpg', '.jpeg', '.png'].includes(
      extname(file.originalname).toLowerCase()
    ) ? extname(file.originalname).toLowerCase() : '.bin';
    // Keine Nutzereingabe im Dateinamen
    cb(null, randomUUID() + ext);
  },
});

function fileFilter(_req, file, cb) {
  const allowed = ['image/jpeg', 'image/png'];
  cb(null, allowed.includes(file.mimetype));
}

export const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5 MB
});
```

## routes/reports.js – Haupt-Router

```javascript
// src/routes/reports.js
import { Router } from 'express';
import { body, validationResult } from 'express-validator';
import { createHash } from 'crypto';
import { randomUUID } from 'crypto';
import { writeFileSync } from 'fs';
import { join } from 'path';
import { fileURLToPath } from 'url';
import rateLimit from 'express-rate-limit';
import db from '../db.js';
import { upload } from '../middleware/upload.js';
import { createGitHubIssue } from '../services/github.js';

const router = Router();
const __dir = dirname(fileURLToPath(import.meta.url));

// Rate-Limiting: 10 Anfragen pro Minute pro IP
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Zu viele Anfragen – bitte warte eine Minute.' },
});

// Validierungsregeln
const reportValidation = [
  body('type')
    .isIn(['bug', 'idea', 'feedback'])
    .withMessage('Ungültiger Typ'),
  body('title')
    .trim().isLength({ min: 3, max: 200 })
    .withMessage('Titel: 3–200 Zeichen'),
  body('description')
    .optional().trim().isLength({ max: 10000 })
    .withMessage('Beschreibung zu lang'),
  body('severity')
    .optional().isIn(['low', 'medium', 'high', 'critical'])
    .withMessage('Ungültiger Schweregrad'),
  body('system_info')
    .optional().isObject()
    .withMessage('system_info muss ein Objekt sein'),
];

// POST /api/reports
router.post('/',
  limiter,
  upload.single('screenshot'),   // optionaler Datei-Upload
  reportValidation,
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({ errors: errors.array() });
    }

    const { type, title, description = '', severity = 'medium',
            system_info, screenshot: screenshotBase64 } = req.body;

    // IP-Hash (DSGVO-konform)
    const ip = req.ip || req.socket.remoteAddress || '';
    const ipHash = createHash('sha256').update(ip).digest('hex');

    // Screenshot-Pfad ermitteln (Datei-Upload hat Vorrang vor Base64)
    let screenshotPath = null;
    if (req.file) {
      // Multer hat bereits validiert und gespeichert
      screenshotPath = req.file.filename; // nur Dateiname, kein absoluter Pfad
    } else if (screenshotBase64) {
      // Base64-Dekodierung mit Pfad-Traversal-Schutz
      const base64Data = screenshotBase64.replace(/^data:image\/\w+;base64,/, '');
      const buf = Buffer.from(base64Data, 'base64');
      if (buf.length > 5 * 1024 * 1024) {
        return res.status(422).json({ error: 'Screenshot zu groß (max. 5 MB)' });
      }
      const filename = randomUUID() + '.png';
      const dest = join(__dir, '..', '..', 'uploads', filename);
      writeFileSync(dest, buf);
      screenshotPath = filename;
    }

    // Datenbank-Insert
    const stmt = db.prepare(`
      INSERT INTO reports
        (type, title, description, severity, screenshot_path, system_info, ip_hash)
      VALUES
        (@type, @title, @description, @severity, @screenshot, @sysinfo, @ip)
    `);
    const result = stmt.run({
      type,
      title,
      description,
      severity,
      screenshot: screenshotPath,
      sysinfo: system_info ? JSON.stringify(system_info) : null,
      ip: ipHash,
    });

    // GitHub-Issue anlegen (nur für Bugs, wenn konfiguriert)
    let githubUrl = null;
    if (type === 'bug' && process.env.GITHUB_TOKEN) {
      githubUrl = await createGitHubIssue({
        id: result.lastInsertRowid,
        title, description, severity, system_info,
      });
      if (githubUrl) {
        db.prepare('UPDATE reports SET github_issue = ? WHERE id = ?')
          .run(githubUrl, result.lastInsertRowid);
      }
    }

    return res.status(201).json({
      id: result.lastInsertRowid,
      status: 'created',
      github_issue: githubUrl,
    });
  }
);

export default router;
```

## services/github.js – GitHub Issue-Erstellung

```javascript
// src/services/github.js

const GITHUB_API = 'https://api.github.com';

export async function createGitHubIssue({ id, title, description,
                                          severity, system_info }) {
  const owner = process.env.GITHUB_OWNER || 'example-org';
  const repo  = process.env.GITHUB_REPO  || 'example-app';
  const token = process.env.GITHUB_TOKEN;
  if (!token) return null;

  const labels = severity === 'critical' ? ['bug', 'critical']
               : severity === 'high'     ? ['bug', 'priority: high']
               : ['bug'];

  const body = `${description}\n\n---\n**System-Info:**\n\`\`\`json\n` +
               `${JSON.stringify(system_info, null, 2)}\n\`\`\`\n` +
               `\n*Report #${id} via Feedback Hub*`;

  try {
    const response = await fetch(`${GITHUB_API}/repos/${owner}/${repo}/issues`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Accept':        'application/vnd.github+json',
        'Content-Type':  'application/json',
      },
      body: JSON.stringify({ title: `[Bug #${id}] ${title}`, body, labels }),
    });
    if (!response.ok) return null;
    const json = await response.json();
    return json.html_url ?? null;
  } catch {
    return null;
  }
}
```

## index.js – App-Einstiegspunkt

```javascript
// src/index.js
import express from 'express';
import cors from 'cors';
import reportsRouter from './routes/reports.js';

const app  = express();
const PORT = process.env.PORT || 3000;

app.set('trust proxy', 1); // für korrekte IP hinter Reverse-Proxy

app.use(cors({ origin: process.env.ALLOWED_ORIGIN || 'https://app.example.com' }));
app.use(express.json({ limit: '10mb' }));

app.use('/api/reports', reportsRouter);

app.listen(PORT, () => console.log(`Feedback API läuft auf Port ${PORT}`));
```

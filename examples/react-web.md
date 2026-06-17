# React Web App – Feedback Hub Integration

TypeScript-Integration mit Custom Hook, festem Floating-Button und html2canvas-Screenshot.

## Voraussetzungen

```bash
npm install html2canvas
# TypeScript-Typen sind in neueren Versionen inbegriffen
```

## TypeScript Interfaces

```typescript
// src/feedback/types.ts
export type FeedbackType = 'bug' | 'idea' | 'feedback';
export type Severity = 'low' | 'medium' | 'high' | 'critical';

export interface FeedbackReport {
  type: FeedbackType;
  title: string;
  description: string;
  severity?: Severity;
  screenshot?: string;        // Base64-PNG (Data URL)
  system_info: SystemInfo;
  created_at: string;         // ISO 8601
}

export interface SystemInfo {
  user_agent: string;
  language: string;
  url: string;
  screen_width: number;
  screen_height: number;
  viewport_width: number;
  viewport_height: number;
  platform: string;
}
```

## useFeedback Hook

```typescript
// src/feedback/useFeedback.ts
import { useCallback, useState } from 'react';
import html2canvas from 'html2canvas';
import type { FeedbackType, FeedbackReport, Severity } from './types';

const API_URL = 'https://api.example.com/reports';

function collectSystemInfo() {
  return {
    user_agent:      navigator.userAgent,
    language:        navigator.language,
    url:             window.location.href,
    screen_width:    screen.width,
    screen_height:   screen.height,
    viewport_width:  window.innerWidth,
    viewport_height: window.innerHeight,
    platform:        navigator.platform ?? 'unknown',
  };
}

async function takeScreenshot(): Promise<string | undefined> {
  try {
    const canvas = await html2canvas(document.body, {
      useCORS: true,          // Cross-Origin-Bilder einbeziehen
      scale: 0.75,            // Auflösung reduzieren für kleinere Payload
      logging: false,
    });
    return canvas.toDataURL('image/png');
  } catch {
    return undefined;
  }
}

async function sendReport(report: FeedbackReport): Promise<boolean> {
  try {
    const res = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(report),
    });
    return res.status === 201;
  } catch {
    return false;
  }
}

export function useFeedback() {
  const [loading, setLoading] = useState(false);

  const submitBug = useCallback(async (
    title: string,
    description: string,
    severity: Severity = 'medium',
    withScreenshot = true,
  ) => {
    setLoading(true);
    const screenshot = withScreenshot ? await takeScreenshot() : undefined;
    const ok = await sendReport({
      type: 'bug', title, description, severity,
      screenshot,
      system_info: collectSystemInfo(),
      created_at: new Date().toISOString(),
    });
    setLoading(false);
    return ok;
  }, []);

  const submitIdea = useCallback(async (title: string, description: string) => {
    setLoading(true);
    const ok = await sendReport({
      type: 'idea', title, description,
      system_info: collectSystemInfo(),
      created_at: new Date().toISOString(),
    });
    setLoading(false);
    return ok;
  }, []);

  const submitFeedback = useCallback(async (
    title: string, description: string
  ) => {
    setLoading(true);
    const ok = await sendReport({
      type: 'feedback', title, description,
      system_info: collectSystemInfo(),
      created_at: new Date().toISOString(),
    });
    setLoading(false);
    return ok;
  }, []);

  return { submitBug, submitIdea, submitFeedback, loading };
}
```

## FeedbackButton Komponente

```tsx
// src/feedback/FeedbackButton.tsx
import { useState } from 'react';
import { FeedbackModal } from './FeedbackModal';

export function FeedbackButton() {
  const [open, setOpen] = useState(false);

  return (
    <>
      <button
        onClick={() => setOpen(true)}
        style={{
          position:     'fixed',
          bottom:       20,
          right:        20,
          zIndex:       9999,
          background:   '#4f46e5',
          color:        'white',
          border:       'none',
          borderRadius: 28,
          padding:      '12px 20px',
          fontSize:     14,
          cursor:       'pointer',
          boxShadow:    '0 4px 12px rgba(0,0,0,0.3)',
          display:      'flex',
          alignItems:   'center',
          gap:          8,
        }}
        aria-label="Feedback senden"
      >
        💬 Feedback
      </button>
      {open && <FeedbackModal onClose={() => setOpen(false)} />}
    </>
  );
}
```

## FeedbackModal Komponente

```tsx
// src/feedback/FeedbackModal.tsx
import { useState } from 'react';
import { useFeedback } from './useFeedback';
import type { FeedbackType } from './types';

interface Props {
  onClose: () => void;
}

const TABS: { value: FeedbackType; label: string }[] = [
  { value: 'bug',      label: 'Bug melden' },
  { value: 'idea',     label: 'Idee' },
  { value: 'feedback', label: 'Feedback' },
];

export function FeedbackModal({ onClose }: Props) {
  const { submitBug, submitIdea, submitFeedback, loading } = useFeedback();
  const [type, setType]           = useState<FeedbackType>('bug');
  const [title, setTitle]         = useState('');
  const [description, setDesc]    = useState('');
  const [severity, setSeverity]   = useState('medium');
  const [screenshot, setShot]     = useState(true);
  const [result, setResult]       = useState<string | null>(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!title.trim()) return;

    let ok = false;
    if (type === 'bug')      ok = await submitBug(title, description, severity as any, screenshot);
    else if (type === 'idea') ok = await submitIdea(title, description);
    else                      ok = await submitFeedback(title, description);

    setResult(ok ? 'Gesendet! Danke für dein Feedback.' : 'Fehler – bitte erneut versuchen.');
    if (ok) setTimeout(onClose, 1500);
  }

  return (
    <div style={{
      position: 'fixed', inset: 0, zIndex: 9998,
      background: 'rgba(0,0,0,0.5)', display: 'flex',
      alignItems: 'center', justifyContent: 'center',
    }}>
      <form onSubmit={handleSubmit} style={{
        background: 'white', borderRadius: 12, padding: 24,
        width: 480, display: 'flex', flexDirection: 'column', gap: 12,
      }}>
        <h2 style={{ margin: 0 }}>Feedback senden</h2>

        {/* Typ-Auswahl */}
        <div style={{ display: 'flex', gap: 8 }}>
          {TABS.map(tab => (
            <button key={tab.value} type="button"
              onClick={() => setType(tab.value)}
              style={{
                flex: 1, padding: '8px 0',
                background: type === tab.value ? '#4f46e5' : '#f3f4f6',
                color: type === tab.value ? 'white' : '#374151',
                border: 'none', borderRadius: 6, cursor: 'pointer',
              }}
            >{tab.label}</button>
          ))}
        </div>

        <input value={title} onChange={e => setTitle(e.target.value)}
          placeholder="Titel *" required
          style={{ padding: 8, border: '1px solid #d1d5db', borderRadius: 6 }} />

        <textarea value={description} onChange={e => setDesc(e.target.value)}
          placeholder="Beschreibung" rows={4}
          style={{ padding: 8, border: '1px solid #d1d5db', borderRadius: 6, resize: 'vertical' }} />

        {type === 'bug' && (
          <select value={severity} onChange={e => setSeverity(e.target.value)}
            style={{ padding: 8, border: '1px solid #d1d5db', borderRadius: 6 }}>
            <option value="low">Niedrig</option>
            <option value="medium">Mittel</option>
            <option value="high">Hoch</option>
            <option value="critical">Kritisch</option>
          </select>
        )}

        <label style={{ display: 'flex', alignItems: 'center', gap: 8, fontSize: 14 }}>
          <input type="checkbox" checked={screenshot}
            onChange={e => setShot(e.target.checked)} />
          Screenshot anhängen (html2canvas)
        </label>

        {result && (
          <p style={{ color: result.includes('Fehler') ? 'red' : 'green', margin: 0 }}>
            {result}
          </p>
        )}

        <div style={{ display: 'flex', justifyContent: 'flex-end', gap: 8 }}>
          <button type="button" onClick={onClose}
            style={{ padding: '8px 16px', background: '#f3f4f6',
                     border: 'none', borderRadius: 6, cursor: 'pointer' }}>
            Abbrechen
          </button>
          <button type="submit" disabled={loading}
            style={{ padding: '8px 16px', background: '#4f46e5', color: 'white',
                     border: 'none', borderRadius: 6, cursor: 'pointer' }}>
            {loading ? 'Sende...' : 'Senden'}
          </button>
        </div>
      </form>
    </div>
  );
}
```

## App-Integration

```tsx
// src/App.tsx
import { FeedbackButton } from './feedback/FeedbackButton';

export default function App() {
  return (
    <>
      {/* Dein App-Inhalt */}
      <main>...</main>
      {/* Feedback-Button immer sichtbar */}
      <FeedbackButton />
    </>
  );
}
```

## Hinweis zu html2canvas

`html2canvas` funktioniert bei den meisten Web-Apps zuverlässig. Einschränkungen:
- Cross-Origin-Bilder ohne CORS-Header werden nicht gerendert.
- CSS-`clip-path` und einige Filter werden nicht vollständig unterstützt.
- `useCORS: true` aktiviert den Versuch, externe Bilder einzubinden.

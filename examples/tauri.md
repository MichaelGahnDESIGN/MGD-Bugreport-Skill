# Tauri – Feedback Hub Integration

Tauri verbindet einen Rust-Backend-Prozess mit einem Web-Frontend. Screenshots werden per Rust-Command aufgenommen, Berichte per `invoke()` übermittelt.

## Projektstruktur

```
example-app/
├── src-tauri/
│   ├── src/
│   │   ├── main.rs
│   │   └── feedback.rs   ← Tauri-Commands
│   └── tauri.conf.json
└── src/
    └── feedback/
        ├── FeedbackService.ts
        └── FeedbackButton.svelte  (oder .vue / .tsx)
```

## Cargo.toml – Abhängigkeiten

```toml
# src-tauri/Cargo.toml
[dependencies]
tauri = { version = "2", features = ["http"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
screenshots = "0.8"   # Screenshot-Crate: screenshots-rs
base64 = "0.22"
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }
```

## Rust: FeedbackReport und Commands

```rust
// src-tauri/src/feedback.rs
use base64::{engine::general_purpose, Engine};
use serde::{Deserialize, Serialize};
use screenshots::Screen;
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct FeedbackReport {
    pub r#type: String,         // "bug" | "idea" | "feedback"
    pub title: String,
    pub description: String,
    pub severity: Option<String>,
    pub screenshot: Option<String>,
    pub system_info: SystemInfo,
    pub created_at: u64,        // Unix-Timestamp
}

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct SystemInfo {
    pub os: String,
    pub os_version: String,
    pub arch: String,
    pub app_version: String,
}

impl SystemInfo {
    pub fn collect(app_version: &str) -> Self {
        SystemInfo {
            os: std::env::consts::OS.to_string(),
            os_version: os_info::get().version().to_string(),
            arch: std::env::consts::ARCH.to_string(),
            app_version: app_version.to_string(),
        }
    }
}

/// Screenshot des primären Monitors aufnehmen
#[tauri::command]
pub fn take_screenshot() -> Option<String> {
    let screens = Screen::all().ok()?;
    let screen = screens.into_iter().next()?;
    let image = screen.capture().ok()?;
    let png_bytes = image.to_png(None).ok()?;
    Some(general_purpose::STANDARD.encode(&png_bytes))
}

/// Report an das Backend senden
#[tauri::command]
pub async fn submit_report(report: FeedbackReport) -> Result<bool, String> {
    let client = reqwest::Client::new();
    let response = client
        .post("https://feedback.example.com/api/report")
        .json(&report)
        .timeout(std::time::Duration::from_secs(15))
        .send()
        .await
        .map_err(|e| e.to_string())?;
    Ok(response.status().as_u16() == 201)
}
```

## Rust: main.rs

```rust
// src-tauri/src/main.rs
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

mod feedback;
use feedback::{submit_report, take_screenshot};

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![
            take_screenshot,
            submit_report,
        ])
        .run(tauri::generate_context!())
        .expect("Tauri-App konnte nicht gestartet werden");
}
```

## tauri.conf.json – Berechtigungen

```json
{
  "$schema": "../node_modules/@tauri-apps/cli/schema.json",
  "productName": "example-app",
  "version": "1.0.0",
  "tauri": {
    "allowlist": {
      "http": {
        "all": true,
        "request": true,
        "scope": ["https://feedback.example.com/*"]
      },
      "shell": { "all": false }
    },
    "windows": [
      {
        "title": "Example App",
        "width": 1200,
        "height": 800
      }
    ]
  }
}
```

## TypeScript Frontend: FeedbackService

```typescript
// src/feedback/FeedbackService.ts
import { invoke } from '@tauri-apps/api/core';
import { platform, version, arch } from '@tauri-apps/api/os';
import { getVersion } from '@tauri-apps/api/app';

export type FeedbackType = 'bug' | 'idea' | 'feedback';

export interface FeedbackReport {
  type: FeedbackType;
  title: string;
  description: string;
  severity?: string;
  screenshot?: string;
  system_info: {
    os: string;
    os_version: string;
    arch: string;
    app_version: string;
  };
  created_at: number;
}

export class FeedbackService {
  private async getSystemInfo() {
    const [os, osVersion, architecture, appVersion] = await Promise.all([
      platform(),
      version(),
      arch(),
      getVersion(),
    ]);
    return { os, os_version: osVersion, arch: architecture, app_version: appVersion };
  }

  async submitBug(title: string, description: string,
                  severity = 'medium'): Promise<boolean> {
    return this._submit({ type: 'bug', title, description, severity });
  }

  async submitIdea(title: string, description: string): Promise<boolean> {
    return this._submit({ type: 'idea', title, description });
  }

  async submitFeedback(title: string, description: string): Promise<boolean> {
    return this._submit({ type: 'feedback', title, description });
  }

  private async _submit(
    data: Omit<FeedbackReport, 'screenshot' | 'system_info' | 'created_at'>
  ): Promise<boolean> {
    try {
      const [screenshot, system_info] = await Promise.all([
        invoke<string | null>('take_screenshot'),
        this.getSystemInfo(),
      ]);
      const report: FeedbackReport = {
        ...data,
        screenshot: screenshot ?? undefined,
        system_info,
        created_at: Math.floor(Date.now() / 1000),
      };
      return await invoke<boolean>('submit_report', { report });
    } catch (err) {
      console.error('Feedback-Fehler:', err);
      return false;
    }
  }
}
```

## Svelte FeedbackButton-Komponente

```svelte
<!-- src/feedback/FeedbackButton.svelte -->
<script lang="ts">
  import { FeedbackService, type FeedbackType } from './FeedbackService';

  const service = new FeedbackService();
  let open = false;
  let type: FeedbackType = 'bug';
  let title = '';
  let description = '';
  let sending = false;
  let message = '';

  async function send() {
    if (!title.trim()) return;
    sending = true;
    let ok = false;
    if (type === 'bug') ok = await service.submitBug(title, description);
    else if (type === 'idea') ok = await service.submitIdea(title, description);
    else ok = await service.submitFeedback(title, description);
    message = ok ? 'Gesendet! Danke.' : 'Fehler – bitte erneut versuchen.';
    sending = false;
    if (ok) setTimeout(() => { open = false; message = ''; }, 1500);
  }
</script>

<!-- Floating Button -->
<button class="fab" on:click={() => (open = true)}>💬 Feedback</button>

{#if open}
  <div class="overlay" on:click|self={() => (open = false)}>
    <div class="modal">
      <h2>Feedback senden</h2>
      <select bind:value={type}>
        <option value="bug">Bug melden</option>
        <option value="idea">Idee vorschlagen</option>
        <option value="feedback">Allgemeines Feedback</option>
      </select>
      <input bind:value={title} placeholder="Titel" />
      <textarea bind:value={description} placeholder="Beschreibung" rows="4"></textarea>
      {#if message}<p class="msg">{message}</p>{/if}
      <div class="actions">
        <button on:click={() => (open = false)}>Abbrechen</button>
        <button class="primary" on:click={send} disabled={sending}>
          {sending ? 'Sende...' : 'Senden'}
        </button>
      </div>
    </div>
  </div>
{/if}

<style>
  .fab {
    position: fixed; bottom: 20px; right: 20px;
    z-index: 9999; background: #4f46e5; color: white;
    border: none; border-radius: 28px; padding: 12px 20px;
    cursor: pointer; font-size: 14px; box-shadow: 0 4px 12px rgba(0,0,0,.3);
  }
  .overlay {
    position: fixed; inset: 0; background: rgba(0,0,0,.5);
    z-index: 9998; display: flex; align-items: center; justify-content: center;
  }
  .modal {
    background: white; border-radius: 12px; padding: 24px; width: 440px;
    display: flex; flex-direction: column; gap: 10px;
  }
  .actions { display: flex; justify-content: flex-end; gap: 8px; }
  .primary { background: #4f46e5; color: white; }
  select, input, textarea { padding: 8px; border: 1px solid #d1d5db;
    border-radius: 6px; font-size: 14px; width: 100%; box-sizing: border-box; }
</style>
```

# Vue 3 Web App – Feedback Hub Integration

Vue 3 Composition API mit TypeScript, `<script setup>`, Composable, Pinia-Store und html2canvas-Screenshot.

## Voraussetzungen

```bash
npm install html2canvas pinia
npm install --save-dev @types/html2canvas
```

## TypeScript-Interfaces

```typescript
// src/feedback/types.ts
export type FeedbackType = 'bug' | 'idea' | 'feedback';
export type Severity = 'low' | 'medium' | 'high' | 'critical';

export interface FeedbackReport {
  type: FeedbackType;
  title: string;
  description: string;
  severity?: Severity;
  screenshot?: string;
  system_info: SystemInfo;
  created_at: string;
}

export interface SystemInfo {
  user_agent: string;
  language: string;
  url: string;
  screen_width: number;
  screen_height: number;
  viewport_width: number;
  viewport_height: number;
}
```

## Pinia Store (optional)

```typescript
// src/feedback/feedbackStore.ts
import { defineStore } from 'pinia';
import { ref } from 'vue';
import type { FeedbackType, Severity } from './types';

export const useFeedbackStore = defineStore('feedback', () => {
  const isOpen        = ref(false);
  const activeType    = ref<FeedbackType>('bug');
  const lastResult    = ref<'success' | 'error' | null>(null);
  const totalSent     = ref(0);

  function open(type: FeedbackType = 'bug') {
    activeType.value = type;
    isOpen.value     = true;
  }

  function close() {
    isOpen.value  = false;
    lastResult.value = null;
  }

  function setResult(ok: boolean) {
    lastResult.value = ok ? 'success' : 'error';
    if (ok) totalSent.value++;
  }

  return { isOpen, activeType, lastResult, totalSent, open, close, setResult };
});
```

## useFeedback Composable

```typescript
// src/feedback/useFeedback.ts
import { ref } from 'vue';
import html2canvas from 'html2canvas';
import type { FeedbackReport, FeedbackType, Severity } from './types';

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
  };
}

async function captureScreenshot(): Promise<string | undefined> {
  try {
    const canvas = await html2canvas(document.body, {
      useCORS: true,
      scale: 0.75,
      logging: false,
    });
    return canvas.toDataURL('image/png');
  } catch {
    return undefined;
  }
}

async function postReport(report: FeedbackReport): Promise<boolean> {
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
  const loading = ref(false);
  const error   = ref<string | null>(null);

  async function submitBug(
    title: string,
    description: string,
    severity: Severity = 'medium',
    withScreenshot = true,
  ): Promise<boolean> {
    loading.value = true;
    error.value   = null;
    const screenshot = withScreenshot ? await captureScreenshot() : undefined;
    const ok = await postReport({
      type: 'bug', title, description, severity, screenshot,
      system_info: collectSystemInfo(),
      created_at: new Date().toISOString(),
    });
    if (!ok) error.value = 'Senden fehlgeschlagen';
    loading.value = false;
    return ok;
  }

  async function submitIdea(title: string, description: string): Promise<boolean> {
    loading.value = true;
    const ok = await postReport({
      type: 'idea', title, description,
      system_info: collectSystemInfo(),
      created_at: new Date().toISOString(),
    });
    loading.value = false;
    return ok;
  }

  async function submitFeedback(title: string, description: string): Promise<boolean> {
    loading.value = true;
    const ok = await postReport({
      type: 'feedback', title, description,
      system_info: collectSystemInfo(),
      created_at: new Date().toISOString(),
    });
    loading.value = false;
    return ok;
  }

  return { loading, error, submitBug, submitIdea, submitFeedback };
}
```

## FeedbackButton.vue

```vue
<!-- src/feedback/FeedbackButton.vue -->
<script setup lang="ts">
import { useFeedbackStore } from './feedbackStore';
const store = useFeedbackStore();
</script>

<template>
  <button class="feedback-fab" @click="store.open('bug')" aria-label="Feedback senden">
    💬 Feedback
  </button>

  <!-- Modal wird separat gerendert -->
  <FeedbackModal v-if="store.isOpen" />
</template>

<style scoped>
.feedback-fab {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
  background: #4f46e5;
  color: white;
  border: none;
  border-radius: 28px;
  padding: 12px 20px;
  font-size: 14px;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  gap: 8px;
  transition: background 0.15s;
}

.feedback-fab:hover {
  background: #4338ca;
}
</style>
```

## FeedbackModal.vue

```vue
<!-- src/feedback/FeedbackModal.vue -->
<script setup lang="ts">
import { ref } from 'vue';
import { useFeedback } from './useFeedback';
import { useFeedbackStore } from './feedbackStore';
import type { FeedbackType, Severity } from './types';

const store       = useFeedbackStore();
const { submitBug, submitIdea, submitFeedback, loading } = useFeedback();

const title       = ref('');
const description = ref('');
const severity    = ref<Severity>('medium');
const withShot    = ref(true);
const resultMsg   = ref('');

const tabs: { value: FeedbackType; label: string }[] = [
  { value: 'bug',      label: 'Bug melden' },
  { value: 'idea',     label: 'Idee' },
  { value: 'feedback', label: 'Feedback' },
];

async function send() {
  if (!title.value.trim()) return;
  let ok = false;

  if (store.activeType === 'bug') {
    ok = await submitBug(title.value, description.value, severity.value, withShot.value);
  } else if (store.activeType === 'idea') {
    ok = await submitIdea(title.value, description.value);
  } else {
    ok = await submitFeedback(title.value, description.value);
  }

  store.setResult(ok);
  resultMsg.value = ok ? 'Gesendet! Danke.' : 'Fehler – bitte erneut versuchen.';
  if (ok) setTimeout(() => store.close(), 1500);
}
</script>

<template>
  <Teleport to="body">
    <div class="overlay" @click.self="store.close()">
      <form class="modal" @submit.prevent="send">
        <h2>Feedback senden</h2>

        <!-- Typ-Tabs -->
        <div class="tabs">
          <button
            v-for="tab in tabs" :key="tab.value"
            type="button"
            :class="['tab', { active: store.activeType === tab.value }]"
            @click="store.activeType = tab.value"
          >{{ tab.label }}</button>
        </div>

        <input v-model="title" placeholder="Titel *" required class="field" />
        <textarea v-model="description" placeholder="Beschreibung"
                  rows="4" class="field" />

        <!-- Schweregrad nur für Bugs -->
        <select v-if="store.activeType === 'bug'" v-model="severity" class="field">
          <option value="low">Niedrig</option>
          <option value="medium">Mittel</option>
          <option value="high">Hoch</option>
          <option value="critical">Kritisch</option>
        </select>

        <label class="checkbox-row">
          <input type="checkbox" v-model="withShot" />
          Screenshot anhängen (html2canvas)
        </label>

        <p v-if="resultMsg" :class="{ error: resultMsg.includes('Fehler') }">
          {{ resultMsg }}
        </p>

        <div class="actions">
          <button type="button" class="btn-cancel" @click="store.close()">
            Abbrechen
          </button>
          <button type="submit" class="btn-primary" :disabled="loading">
            {{ loading ? 'Sende...' : 'Senden' }}
          </button>
        </div>
      </form>
    </div>
  </Teleport>
</template>

<style scoped>
.overlay {
  position: fixed; inset: 0; z-index: 9998;
  background: rgba(0,0,0,0.5);
  display: flex; align-items: center; justify-content: center;
}
.modal {
  background: white; border-radius: 12px; padding: 24px;
  width: 480px; display: flex; flex-direction: column; gap: 12px;
}
.modal h2 { margin: 0; }
.tabs { display: flex; gap: 8px; }
.tab {
  flex: 1; padding: 8px; border: 1px solid #d1d5db;
  border-radius: 6px; background: #f9fafb; cursor: pointer;
}
.tab.active { background: #4f46e5; color: white; border-color: #4f46e5; }
.field {
  padding: 8px; border: 1px solid #d1d5db; border-radius: 6px;
  font-size: 14px; width: 100%; box-sizing: border-box;
}
.checkbox-row { display: flex; align-items: center; gap: 8px; font-size: 14px; }
.actions { display: flex; justify-content: flex-end; gap: 8px; }
.btn-cancel { padding: 8px 16px; background: #f3f4f6; border: none; border-radius: 6px; cursor: pointer; }
.btn-primary { padding: 8px 16px; background: #4f46e5; color: white; border: none; border-radius: 6px; cursor: pointer; }
.error { color: red; margin: 0; }
</style>
```

## App.vue – Integration

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { createPinia } from 'pinia';
import FeedbackButton from './feedback/FeedbackButton.vue';
</script>

<template>
  <main>
    <!-- App-Inhalt -->
  </main>

  <!-- Feedback-Button immer sichtbar (per Teleport ins body) -->
  <FeedbackButton />
</template>
```

## main.ts

```typescript
// src/main.ts
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import App from './App.vue';

const app = createApp(App);
app.use(createPinia());
app.mount('#app');
```

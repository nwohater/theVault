---
id: ea628b18-5fa2-416e-8859-f007188e735e
title: 'Feature: Settings & Local AI Chat'
author: ai
tags:
- feature
- settings
- ai
- chat
created: 2026-04-07T15:15:54.485883500Z
modified: 2026-04-07T15:15:54.485883500Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: Settings & Local AI Chat

The Settings view configures a connection to a locally-running AI model and provides an in-app chat interface for asking questions about game strategy, player insights, and lineup decisions.

---

## Key Files

| File | Role |
|---|---|
| `src/views/Settings.tsx` | Settings form + `AiChatModal` component (co-located) |

---

## Configuration Fields

| Field | localStorage key | Default |
|---|---|---|
| Local AI Base URL | `wbmgr_ai_url` | `http://localhost:11434` |
| Model Name (optional) | `wbmgr_ai_model` | `""` (empty) |

Settings are saved by clicking **Save**. A brief "Saved ✓" confirmation replaces the button text for ~2 seconds.

---

## Supported Backends

Any OpenAI-compatible `/v1/chat/completions` endpoint:
- **Ollama** (default port 11434) — model name required
- **LM Studio** (default port 1234) — model name optional (uses loaded model)
- **Jan** — OpenAI-compatible, same as LM Studio
- Any other OpenAI-spec server

Recommended: models with at least 4B parameters (examples: `llama3.2:3b`, `mistral:7b`, `phi3:mini`).

---

## AI Chat Modal

Opened via the **Test Connection** button. A floating modal over the settings page.

**Request format** sent to the AI endpoint:
```json
{
  "model": "<configured model or 'local-model'>",
  "messages": [{ "role": "user"|"assistant", "content": "..." }, ...],
  "stream": false
}
```

**Features:**
- Full conversation history is maintained in component state (`msgs: ChatMsg[]`) and sent with every request (multi-turn context).
- Auto-scrolls to the latest message.
- Shows animated "thinking" dots while loading.
- Displays error messages inline if the fetch fails.
- Enter key submits (Shift+Enter does not).
- Chat history is **not persisted** — cleared when the modal closes.

---

## No Tauri IPC

The AI requests are made directly from the React layer using the browser `fetch` API. No Rust commands are involved. This means the endpoint must be reachable from the Tauri webview (localhost URLs work fine).

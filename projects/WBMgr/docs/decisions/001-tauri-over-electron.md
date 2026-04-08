---
id: 0413bfc7-2deb-40df-8c88-4a8fab8fa131
title: 001 — Tauri over Electron
author: ai
tags:
- decision
- architecture
- tauri
created: 2026-04-07T15:16:13.585587900Z
modified: 2026-04-07T15:16:13.585587900Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# 001 — Tauri over Electron

**Status:** Accepted  
**Inferred from:** `src-tauri/`, `package.json`, `tauri.conf.json`

---

## Decision

Use Tauri 2 (Rust backend + system WebView) instead of Electron (Chromium bundled) as the desktop shell.

---

## Rationale

Tauri produces significantly smaller application bundles because it uses the OS-native WebView rather than bundling Chromium. It also has a lower memory footprint at runtime. For a small utility app like WBMgr, the Rust backend complexity is minimal — the app only needed a window shell, and the single `greet` command in `lib.rs` confirms the backend was never meaningfully used.

---

## Consequences

- **Build requirement**: Rust toolchain (rustup) must be installed to build or run in dev mode. This is a higher setup bar than a pure Node.js project.
- **Rust backend is mostly unused**: All logic lives in React/TypeScript. The Rust layer is a scaffold.
- **WebView rendering differences**: The app renders in whatever WebView the OS provides (WebView2 on Windows, WebKit on macOS). CSS/JS behavior may differ slightly across platforms, unlike Electron's consistent Chromium.
- **Local AI calls use browser fetch**: No Rust IPC needed because `fetch` works from the WebView directly to localhost.

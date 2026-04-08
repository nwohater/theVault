---
id: 1a6394f8-c6d8-4591-af1f-59e1d12d6973
title: 003 — localStorage for User State
author: ai
tags:
- decision
- persistence
- localStorage
created: 2026-04-07T15:16:32.634524100Z
modified: 2026-04-07T15:16:32.634524100Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# 003 — localStorage for User State

**Status:** Accepted  
**Inferred from:** `src/views/MyTeam.tsx`, `src/views/Settings.tsx`, `src/utils/stats.ts`

---

## Decision

User-mutable state (roster composition, batting order, AI settings) is persisted using `localStorage` rather than Tauri file system APIs or a database.

---

## localStorage Keys

| Key | Type | Content |
|---|---|---|
| `rc-active` | JSON string[] | Names of active roster players |
| `rc-reserves` | JSON string[] | Names of reserve players |
| `rc-batting-order` | JSON LineupEntry[] | Current batting order slots |
| `wbmgr_ai_url` | string | AI server base URL |
| `wbmgr_ai_model` | string | AI model name |

---

## Rationale

`localStorage` is immediately available in the Tauri WebView with no additional dependencies or Rust IPC. For the small amount of user-editable state in this app (a few arrays and two strings), it is more than sufficient and avoids the complexity of file system permissions, async Tauri commands, or a database setup.

---

## Consequences

- **Data is WebView-scoped**: Stored in the WebView's origin storage, tied to the app's identifier (`com.wbmgr.app`). Data persists across app restarts.
- **Not portable**: Data cannot easily be exported or shared between machines.
- **No error recovery UI**: Parsing errors are silently caught and fall back to defaults (see `loadStrArr()` in `utils/stats.ts`).
- **Simulated schedule results are not persisted**: Current-week sim results live only in React state and are lost on navigation.

---
id: cfa87686-802d-42e7-be56-49698eb578b8
title: Architecture Overview
author: ai
tags:
- context
- architecture
created: 2026-04-06T15:35:26.811359100Z
modified: 2026-04-06T15:35:26.811359100Z
project: vibe2
status: draft
ai_tool: claude-code
---

# Architecture Overview

## System Diagram

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Next.js    │────▶│  Tauri IPC   │────▶│  Rust Core   │
│   Frontend   │◀────│  Commands    │◀────│  (slatevault  │
│              │     │              │     │   -core)      │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                                    ┌─────────────┼─────────────┐
                                    │             │             │
                              ┌─────▼───┐  ┌─────▼───┐  ┌─────▼───┐
                              │ SQLite  │  │  git2   │  │  File   │
                              │  FTS5   │  │  Repo   │  │ System  │
                              └─────────┘  └─────────┘  └─────────┘
```

## Data Flow
1. User interacts with React UI
2. Zustand stores manage frontend state
3. `commands.ts` calls Tauri `invoke()` for backend operations
4. `commands.rs` delegates to `slatevault-core` library
5. Core performs file I/O, git ops, and search indexing

## Key Directories
- `src/` — Next.js frontend (components, stores, types)
- `src-tauri/` — Tauri app shell and commands
- `src-tauri/crates/slatevault-core/` — Core Rust library
- `src-tauri/crates/slatevault-mcp/` — MCP server binary

---
id: edc6ba38-46eb-451d-aa9d-ddf53065f5ed
title: System Prompt for AI Coding Sessions
author: ai
tags:
- prompts
- ai
- system
created: 2026-04-06T15:35:19.490630700Z
modified: 2026-04-06T15:35:19.490630700Z
project: vibe2
status: draft
ai_tool: claude-code
---

# System Prompt for AI Coding Sessions

Use this prompt when starting a new AI coding session:

```
You are helping build a desktop document vault application called slateVault.

Tech stack:
- Frontend: Next.js 15, React 19, TypeScript, Tailwind CSS 4, Zustand
- Backend: Rust, Tauri 2, SQLite (rusqlite with FTS5)
- Editor: CodeMirror 6
- Version Control: git2 crate

Conventions:
- Use TypeScript strict mode
- Prefer functional components with hooks
- Use Zustand for state management
- Tauri commands follow the pattern: Rust function -> commands.rs wrapper -> commands.ts invoke -> store action
- All file paths use forward slashes internally
- Documents have YAML frontmatter with id, title, author, status, tags

When writing code, follow existing patterns in the codebase. Do not add unnecessary abstractions.
```

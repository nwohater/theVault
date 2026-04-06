---
id: abb0c3d8-a954-4abf-88ee-efe8b4befae4
title: Session 001 — Project Setup
author: ai
tags:
- changelog
- session
created: 2026-04-06T15:35:33.195375800Z
modified: 2026-04-06T15:35:33.195375800Z
project: vibe2
status: draft
ai_tool: claude-code
---

# Session 001 — Project Setup

**Date:** 2026-04-06
**Duration:** 3 hours
**AI Tool:** Claude Code

## What was built
- Initialized Tauri 2 + Next.js 15 project
- Set up Tailwind CSS 4 with dark theme
- Created AppShell layout with resizable sidebar, editor, and preview panes
- Implemented basic file tree component
- Added CodeMirror 6 editor with markdown syntax highlighting

## Decisions made
- Chose Zustand over Redux for state management (simpler API, less boilerplate)
- Went with CodeMirror 6 over Monaco (lighter weight, better mobile support)
- Using SQLite FTS5 for search instead of a separate search library

## Issues encountered
- Tauri 2 WebView doesn't support native drag-and-drop by default — had to set `dragDropEnabled: false`
- CodeMirror theme needed custom CSS overrides for proper dark mode integration

## Next session
- Implement document CRUD operations
- Add git integration with git2 crate
- Set up SQLite search index

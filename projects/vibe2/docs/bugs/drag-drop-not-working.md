---
id: 3b3d6181-b6c9-4367-a7ef-f1a80157fa39
title: 'Bug: Drag and Drop Shows Forbidden Cursor'
author: ai
tags:
- bug
- resolved
- drag-drop
created: 2026-04-06T15:35:39.877751Z
modified: 2026-04-06T15:35:39.877751Z
project: vibe2
status: draft
ai_tool: claude-code
---

# Bug: Drag and Drop Shows Forbidden Cursor

**Status:** Resolved
**Severity:** Medium
**Found in:** v0.1.0

## Description
When dragging a document to a folder in the file tree, the cursor shows a red circle with a line through it (forbidden cursor) instead of allowing the drop.

## Steps to Reproduce
1. Open a project with folders and documents
2. Try to drag a document onto a folder
3. Observe the forbidden cursor — drop is not allowed

## Expected Behavior
Document should be movable to the target folder via drag and drop.

## Root Cause
Tauri's WebView has built-in file drop handling (`dragDropEnabled`) that intercepts HTML5 drag events before they reach the web page. Additionally, `e.preventDefault()` must be called directly in the `onDragOver` handler, not in a callback.

## Fix
1. Set `dragDropEnabled: false` in `tauri.conf.json`
2. Call `e.preventDefault()` and `e.dataTransfer.dropEffect = "move"` directly in TreeNode's `onDragOver`
3. Set `e.dataTransfer.setData("text/plain", path)` on drag start

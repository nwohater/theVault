---
id: 6e681128-8ab2-4898-892a-04c55c0f0cab
title: Folder Management - Implemented
author: ai
tags:
- feature
- ui
- file-management
- completed
created: 2026-04-05T15:36:32.999034100Z
modified: 2026-04-05T15:36:32.999034100Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# Folder Management

## Status: Complete

## Features Implemented
- **New Folder** — right-click a project or sub-folder → "New Folder" → enter name
- **Folder tree rendering** — documents grouped by directory with expandable folder nodes
- **Drag and drop** — drag documents onto folders (or project root) to move them
- **Delete empty folders** — right-click folder → Delete (only if no .md files inside, handles hidden Windows system files)
- **Git integration** — file moves auto-stage both old (delete) and new (add) paths, git status refreshes after moves

## Key changes
- `create_folder`, `delete_folder`, `list_folders` Tauri commands
- `buildFolderTree()` groups docs by path into nested folder nodes
- TreeNode supports `draggable`, `onDragStart`, `onDragOver`, `onDrop`
- `dragDropEnabled: false` in tauri.conf.json to prevent Tauri intercepting web drag events
- Context menu has "project", "doc", and "folder" types with appropriate actions for each

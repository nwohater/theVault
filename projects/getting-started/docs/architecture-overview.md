---
id: 7150f8af-dda6-4803-808a-bc66fbf2bb24
title: Architecture Overview
author: ai
tags:
- architecture
- overview
- tauri
- nextjs
created: 2026-03-31T17:16:10.411361400Z
modified: 2026-03-31T17:16:10.411361400Z
project: getting-started
status: draft
ai_tool: claude-code
---

# Architecture Overview

slateVault is built on a Tauri v2 + Next.js stack with a Rust core library.

## Stack

- **Frontend**: Next.js 15 (static export), React, Tailwind CSS v4
- **Backend**: Tauri v2 (Rust), exposes commands via IPC
- **Core**: `slatevault-core` crate handles vault, project, and document logic
- **Storage**: Markdown files on disk + SQLite FTS5 search index
- **Git**: `git2` crate for all version control operations
- **MCP**: Separate `slatevault-mcp` binary, connects Claude Code to the vault

## Data Flow

1. User opens a vault (folder with `vault.toml`)
2. Tauri loads the vault into a `Mutex<Option<Vault>>` state
3. Next.js frontend calls Tauri commands via `invoke()`
4. Documents are stored as markdown files with YAML front matter
5. Search index is rebuilt on vault open and updated on every write

## Folder Structure

```
vault/
  vault.toml          # Vault config (name, sync, MCP settings)
  .gitignore
  index.db            # SQLite FTS5 search index (gitignored)
  projects/
    my-project/
      project.toml    # Project metadata
      docs/
        note.md
```

---
id: fa3fe50e-ff23-44aa-9e2e-9a8ae0c50907
title: MCP Integration with Claude Code
author: ai
tags:
- mcp
- claude
- integration
- ai
created: 2026-03-31T17:16:19.692700400Z
modified: 2026-03-31T17:16:19.692700400Z
project: getting-started
status: draft
ai_tool: claude-code
---

# MCP Integration with Claude Code

slateVault ships a Model Context Protocol (MCP) server that lets Claude Code read and write documents directly in your vault.

## Setup

Run this once in your terminal:

```bash
claude mcp add -s user slatevault -- slatevault-mcp
```

The MCP server automatically connects to whichever vault is currently open in the slateVault app.

## Available Tools

| Tool | Description |
|---|---|
| `list_projects` | List all projects in the vault |
| `list_documents` | List documents in a project |
| `read_document` | Read a document's full content |
| `write_document` | Create or update a document |
| `search_documents` | Full-text search across all docs |
| `get_project_context` | Load all pinned context files for a project |

## How Claude Uses It

When you ask Claude to document something, it will:
1. Call `search_documents` to check if a doc already exists
2. Call `write_document` to save the result to your vault
3. Optionally call `get_project_context` to understand your project before writing

## Auto-staging

Documents written by Claude via MCP are automatically staged for git commit when **Auto-stage AI writes** is enabled in Settings.

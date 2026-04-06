---
id: 916b8f3d-2cf6-43e5-a50f-e64504c9a66d
title: Git Sync Guide
author: ai
tags:
- git
- sync
- guide
- ssh
created: 2026-03-31T17:16:12.999199200Z
modified: 2026-03-31T17:16:12.999199200Z
project: getting-started
status: draft
ai_tool: claude-code
---

# Git Sync Guide

slateVault treats every vault as a git repository, giving you a full history of all your documents.

## Setting Up a Remote

1. Create a repo on GitHub (or any git host)
2. Open **Settings → Git** in slateVault
3. Paste your remote URL
4. Set your SSH key path if using SSH auth
5. Click **Save Settings**

## Auto Sync Options

| Option | Description |
|---|---|
| Pull on open | Automatically pull latest changes when you open the vault |
| Push on close | Automatically push when you close the app |

## Manual Operations

Use the **Git** tab in the sidebar to:
- View file status (modified, staged, untracked)
- Stage individual files
- Write a commit message and commit
- Push / Pull manually

## SSH Keys

If your remote uses SSH, set the path to your private key in Settings. Typical paths:
- `~/.ssh/id_ed25519`
- `~/.ssh/id_rsa`

## Conflict Resolution

If a pull results in merge conflicts, resolve them in the terminal panel (`Ctrl+T`) using standard git commands.

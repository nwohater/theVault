---
id: 10c56c5a-8c5e-4106-b6d1-1d73b441542b
title: PR Review Workflow - Implemented
author: human
tags:
- feature
- enterprise
- git
- pr
- completed
created: 2026-04-06T19:58:50.638856500Z
modified: 2026-04-06T19:58:50.638856500Z
project: slatevault-app
status: draft
canonical: true
protected: false
---



# PR Review Workflow

## Status: Complete

## Phase 1: Branch Management
- `current_branch()`, `list_branches()`, `create_branch()`, `switch_branch()`, `delete_branch()` in vault.rs
- 5 Tauri commands + `git_push_branch`
- BranchSelector dropdown in GitPanel with create/switch/delete
- Dynamic branch display in StatusBar with branch icon
- Switch refuses on dirty worktree

## Phase 2: Diff Viewer
- `diff_file(path, staged)` and `diff_branches(base, head)` using git2 Patch API
- `git_diff_file`, `git_diff_branches` Tauri commands
- DiffViewer component with line numbers, green/red highlighting, hunk headers
- Click any file in ChangesTab to view its diff inline

## Phase 3: PR Creation (GitHub + Azure DevOps)
- `credentials.rs` — PATs stored in `~/.slatevault/credentials.toml` (outside git-tracked vault)
- `pr.rs` — auto-detects platform from remote URL, calls GitHub/Azure DevOps REST APIs
- `git_create_pr`, `git_detect_platform`, `git_save_credentials`, `git_load_credentials` commands
- PrTab as 4th tab in GitPanel — title, description, target branch, diff summary, "Push & Create PR"
- Opens PR URL in browser via tauri-plugin-shell
- Credentials section in SettingsPanel with masked display

## New Files
- `src-tauri/crates/slatevault-core/src/credentials.rs`
- `src-tauri/crates/slatevault-core/src/pr.rs`
- `src/components/git/BranchSelector.tsx`
- `src/components/git/DiffViewer.tsx`
- `src/components/git/PrTab.tsx`

## Dependencies Added
- `reqwest` (blocking, json) for API calls
- `dirs` for credential path resolution

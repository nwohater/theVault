---
id: 85d7685b-4699-4f86-b8d2-f291881fef97
title: Tauri Commands Reference (Updated)
author: ai
tags:
- commands
- reference
- api
- updated
created: 2026-04-05T15:48:40.655845Z
modified: 2026-04-05T15:48:40.655845Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# Tauri Commands Reference

## Vault
- `create_vault(path, name)` — create new vault
- `open_vault(path)` — open existing vault, auto-pull if configured

## Projects
- `create_project(name, description?, tags?)` — create project
- `list_projects()` — list all projects
- `delete_project(name)` — delete project and contents
- `rename_project(oldName, newName)` — rename project directory

## Documents
- `write_document(project, path, title, content, tags?, ai_tool?)` — create/update document
- `read_document(project, path)` — read document content
- `list_documents(project, tags?)` — list documents with optional tag filter
- `delete_document(project, path)` — delete document
- `rename_document(project, oldPath, newPath)` — move/rename document, auto-stages in git

## Folders
- `list_folders(project)` — list all subdirectories in project docs
- `create_folder(project, folderPath)` — create directory
- `delete_folder(project, folderPath)` — delete empty folder (checks for .md files, handles hidden system files)

## Search
- `search_documents(query, project?, limit?)` — full-text search
- `get_project_context(project)` — load pinned AI context files
- `rebuild_index()` — rebuild FTS5 search index
- `vault_stats()` — project/doc counts, MCP status

## Git — Basic
- `git_status()` — file statuses (staged/unstaged)
- `git_stage(path)` — stage file
- `git_unstage(path)` — unstage file
- `git_commit(message)` — commit staged changes
- `git_log(limit?)` — commit history
- `git_push()` — push configured branch
- `git_push_branch(branch)` — push specific branch
- `git_pull()` — pull from remote
- `git_clone(url, path)` — clone repository

## Git — Branches
- `git_current_branch()` — current branch name
- `git_list_branches()` — all local branches
- `git_create_branch(name)` — create from HEAD
- `git_switch_branch(name)` — checkout branch (refuses on dirty worktree)
- `git_delete_branch(name)` — delete branch (refuses if current)

## Git — Diff
- `git_diff_file(path, staged)` — diff for single file
- `git_diff_branches(base, head)` — diff between two branches

## Git — PR
- `git_create_pr(title, description, sourceBranch, targetBranch)` — push + create PR on GitHub/Azure DevOps
- `git_detect_platform()` — auto-detect GitHub or Azure DevOps from remote URL
- `git_save_credentials(githubPat?, adoPat?, adoOrganization?, adoProject?)` — save to ~/.slatevault/credentials.toml
- `git_load_credentials()` — load masked credentials

## Git — Config
- `git_remote_config()` — get remote settings
- `git_set_remote_config(args)` — update remote URL, branch, auto-sync settings

## Config
- `get_vault_config()` — vault settings
- `set_vault_config(args)` — update name, MCP, SSH, auto-stage settings

## Other
- `show_in_folder(project, path?)` — open in OS file explorer
- `spawn_terminal(cwd)` / `write_terminal(data)` / `resize_terminal(rows, cols)` / `close_terminal()`
- `start_mcp_server(vaultPath, port)` / `stop_mcp_server()` / `mcp_server_status()`

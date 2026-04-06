---
id: e1c52b7b-c4c9-405a-beba-c68b60d7f630
title: Git Integration Audit
author: ai
tags:
- git
- architecture
- audit
- implementation
created: 2026-04-05T14:59:21.006445100Z
modified: 2026-04-05T14:59:21.006445100Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# slateVault Git Integration Audit

## Overview

The slateVault application has a comprehensive git integration that spans the Rust backend (via git2 library), Tauri command layer, and a React-based frontend. This document captures all git capabilities, the end-to-end flow, and identifies gaps.

---

## 1. Rust Core Git Operations

### File: `/src-tauri/crates/slatevault-core/src/vault.rs`

All git operations are methods on the `Vault` struct, which wraps a `git2::Repository`.

#### Available Functions

| Function | Signature | Purpose |
|----------|-----------|---------|
| `stage_file()` | `fn(&self, path: &Path) -> Result<()>` | Stages a single file by path. Adds to index if exists, removes if deleted. |
| `stage_path()` | `fn(&self, relative_path: &str) -> Result<()>` | Public wrapper around `stage_file()` for relative paths |
| `unstage_file()` | `fn(&self, relative_path: &str) -> Result<()>` | Unstages a file. If file existed in HEAD, restores HEAD version. If new file, removes from index entirely. |
| `status()` | `fn(&self) -> Result<Vec<FileStatus>>` | Returns all git file status entries (staged and unstaged) |
| `commit()` | `fn(&self, message: &str) -> Result<git2::Oid>` | Creates a commit with staged changes. Uses signature "slateVault User <user@slatevault.local>" if not configured. |
| `log()` | `fn(&self, limit: usize) -> Result<Vec<CommitInfo>>` | Fetches commit history (default 50, time-sorted). Returns OID, message, author, date. |
| `set_git_remote()` | `fn(&self, url: &str) -> Result<()>` | Creates or updates the "origin" remote. |

#### Data Types

**FileStatus:**
```rust
pub struct FileStatus {
    pub path: String,
    pub status: String, // "staged_new" | "staged_modified" | "staged_deleted" | "new" | "modified" | "deleted"
}
```

**CommitInfo:**
```rust
pub struct CommitInfo {
    pub oid: String,          // First 8 chars of commit hash
    pub message: String,      // Commit message
    pub author: String,       // Commit author name
    pub date: String,         // RFC3339 formatted ISO timestamp
}
```

#### Implementation Details

- **Staging Logic** (lines 275-292): Uses git2's index operations directly.
- **Unstaging Logic** (lines 294-335): Smart handling—if file was in HEAD, restores HEAD version; if new, removes from index.
- **Status Enumeration** (lines 337-372): Uses git2::StatusOptions with untracked files included. Maps status flags to status strings.
- **Commit Creation** (lines 407-432): Gets staged tree from index, creates signature, finds parent commit if exists. Handles initial commit case (no parents).
- **Log Walking** (lines 374-405): Uses git2's revwalk with TIME sorting. Limits results to parameter (default 50).
- **Remote Config** (lines 442-453): Uses git2's remote API to create or update "origin".

#### Git Repository Initialization

In `Vault::create()` (lines 17-57):
- Initializes git repo with `git2::Repository::init()`
- Sets `init.defaultBranch = "main"`
- Creates `.gitignore` with `index.db`, `index.db-journal`, `.DS_Store`

---

## 2. Tauri Command Layer

### File: `/src-tauri/src/commands.rs`

This layer exposes git operations as Tauri commands callable from the frontend.

#### Git Commands (with signatures)

| Command | Signature | Notes |
|---------|-----------|-------|
| `git_status()` | `pub fn(state: State) -> CmdResult<Vec<FileStatus>>` | Calls `vault.status()` |
| `git_stage()` | `pub fn(path: String, state) -> CmdResult<String>` | Calls `vault.stage_path(path)` |
| `git_unstage()` | `pub fn(path: String, state) -> CmdResult<String>` | Calls `vault.unstage_file(path)` |
| `git_commit()` | `pub fn(message: String, state) -> CmdResult<String>` | Calls `vault.commit(message)`, returns OID string |
| `git_log()` | `pub fn(limit: Option<usize>, state) -> CmdResult<Vec<CommitInfo>>` | Calls `vault.log(limit.unwrap_or(50))` |
| `git_remote_config()` | `pub fn(state) -> CmdResult<RemoteConfig>` | Returns remote URL, branch, pull_on_open, push_on_close flags |
| `git_set_remote_config()` | `pub fn(args: SetRemoteConfigArgs, state) -> CmdResult<String>` | Updates vault config; mutates vault.config directly |
| `git_clone()` | `pub fn(url: String, path: String) -> CmdResult<String>` | **CLI-based**: runs `git clone` subprocess directly (not git2) |
| `git_push()` | `pub fn(state) -> CmdResult<String>` | **CLI-based**: runs `git push -u origin <branch>` subprocess |
| `git_pull()` | `pub fn(state) -> CmdResult<String>` | **CLI-based**: runs `git pull origin <branch>` subprocess |

#### Special Behaviors

**Auto-pull on vault open** (lines 33-39):
```rust
if vault.config.sync.pull_on_open && vault.config.sync.remote_url.is_some() {
    let branch = &vault.config.sync.remote_branch;
    let _ = std::process::Command::new("git")
        .args(["-C", &vault.root.to_string_lossy(), "pull", "origin", branch])
        .output();
}
```

**Auto-push on app close** (lines 76-82 in lib.rs):
```rust
if vault.config.sync.push_on_close && vault.config.sync.remote_url.is_some() {
    let branch = &vault.config.sync.remote_branch;
    let _ = std::process::Command::new("git")
        .args(["-C", &root, "push", "origin", branch])
        .output();
}
```

#### Remote Config Structure (returned to frontend)

```rust
pub struct RemoteConfig {
    pub remote_url: Option<String>,
    pub remote_branch: String,
    pub pull_on_open: bool,
    pub push_on_close: bool,
}
```

---

## 3. Frontend TypeScript/React Components

### File: `/src/components/git/GitPanel.tsx`

**Root component** that orchestrates three tabs:
- Changes Tab (staging/unstaging/committing)
- History Tab (viewing commit log)
- Remote Tab (push/pull/config)

### File: `/src/components/git/ChangesTab.tsx`

**Purpose**: Stage/unstage files and create commits.

**Features**:
- Loads file status on mount
- Separates files into "Staged Changes" and "Changes" (unstaged)
- Displays status labels: A (added), M (modified), D (deleted), ? (untracked)
- Commit textarea with Ctrl+Enter shortcut
- "Stage All" button appears when unstaged files exist
- Shows output message on commit/error
- Hover actions to stage/unstage individual files

**State Management** via `useGitStore`:
- `files`: FileStatus[]
- `commitMessage`: string
- `output`: string | null
- Methods: `loadStatus()`, `stage()`, `unstage()`, `stageAll()`, `commit()`

### File: `/src/components/git/HistoryTab.tsx`

**Purpose**: View commit history.

**Features**:
- Displays list of commits (up to 50 default)
- Shows commit hash (short OID), author, message, relative date
- Relative date formatting (e.g., "5m ago", "2h ago", "3d ago")
- Loads on tab mount

**State Management** via `useGitStore`:
- `commits`: CommitInfo[]
- Methods: `loadLog()`

### File: `/src/components/git/RemoteTab.tsx`

**Purpose**: Configure and execute push/pull operations.

**Features**:
- Text inputs for remote URL and branch name
- "Save Config" button (saves to vault.toml)
- Pull/Push buttons (disabled when no URL or operation running)
- Checkboxes for `pull_on_open` and `push_on_close`
- Output textarea showing command results
- Running state to disable buttons during operations

**State Management** via `useGitStore`:
- `remoteConfig`: RemoteConfig | null
- Methods: `loadRemoteConfig()`, `setRemoteConfig()`

### File: `/src/stores/gitStore.ts`

**Zustand store** managing all git UI state.

**State**:
```typescript
interface GitState {
  files: FileStatus[];
  commits: CommitInfo[];
  remoteConfig: RemoteConfig | null;
  commitMessage: string;
  loading: boolean;
  output: string | null;
}
```

**Methods**:
- `loadStatus()`: Fetches file status from backend
- `stage(path)`: Stages file, reloads status
- `unstage(path)`: Unstages file, reloads status
- `stageAll()`: Stages all unstaged files sequentially
- `commit()`: Commits staged changes with message, clears message, reloads status and log
- `loadLog()`: Fetches commit history
- `loadRemoteConfig()`: Fetches remote config from vault
- `setRemoteConfig(config)`: Updates remote config, reloads

### File: `/src/lib/commands.ts`

**Tauri invocation wrappers** for all git commands.

```typescript
export async function gitStatus(): Promise<FileStatus[]>
export async function gitStage(path: string): Promise<string>
export async function gitUnstage(path: string): Promise<string>
export async function gitCommit(message: string): Promise<string>
export async function gitLog(limit?: number): Promise<CommitInfo[]>
export async function gitRemoteConfig(): Promise<RemoteConfig>
export async function gitSetRemoteConfig(config: {...}): Promise<string>
export async function gitPush(): Promise<string>
export async function gitPull(): Promise<string>
export async function gitClone(url: string, path: string): Promise<string>
```

---

## 4. MCP Server Integration

### File: `/src-tauri/crates/slatevault-mcp/src/server.rs`

The MCP server exposes document/project operations but **does NOT expose git operations**.

**Available Tools** (via MCP):
- `create_project()`
- `list_projects()`
- `get_project_context()`
- `write_document()`
- `read_document()`
- `list_documents()`
- `search_documents()`

**Git operations are NOT available via MCP** — only through the native Tauri app.

---

## 5. End-to-End Git Flow

### Staging Flow
1. **Frontend**: User clicks "+" button on unstaged file
2. **Frontend**: `ChangesTab` calls `useGitStore.stage(path)`
3. **Store**: Calls `commands.gitStage(path)`
4. **IPC**: Tauri invokes `git_stage` command
5. **Backend**: `vault.stage_path(path)` → `vault.stage_file()` → `index.add_path()` or `index.remove_path()`
6. **Frontend**: Store reloads status via `loadStatus()`

### Commit Flow
1. **Frontend**: User types message and hits Ctrl+Enter or clicks "Commit"
2. **Frontend**: `ChangesTab` calls `useGitStore.commit()`
3. **Store**: Validates message, calls `commands.gitCommit(message)`
4. **IPC**: Tauri invokes `git_commit` command
5. **Backend**: 
   - `vault.commit(message)`
   - Gets staged tree from index
   - Creates signature (or uses default "slateVault User")
   - Finds parent commit if exists
   - Creates commit object
6. **Frontend**: Store clears message, reloads status & log, displays "Committed: <oid>"

### Push/Pull Flow
1. **Frontend**: User clicks Push/Pull button
2. **Frontend**: Remote tab calls `commands.gitPush()` or `commands.gitPull()`
3. **IPC**: Tauri invokes `git_push` or `git_pull` command
4. **Backend**:
   - **push**: Renames local branch to match `remote_branch` config, then `git push -u origin <branch>`
   - **pull**: `git pull origin <branch>`
   - Both use subprocess (not git2 library)
5. **Frontend**: Displays output in textarea

### Auto-Sync Flows
- **On vault open**: If `pull_on_open=true` and remote is set, auto-pulls in background
- **On app close**: If `push_on_close=true` and remote is set, auto-pushes in background
- Both use subprocess commands, not git2

---

## 6. Data Flow Summary

```
Frontend (React/TS)
  ↓
  gitStore.ts (Zustand state)
  ↓
  commands.ts (Tauri IPC)
  ↓
  Tauri Core (commands.rs)
  ↓
  Vault::git_* methods (vault.rs)
  ↓
  git2 library OR git CLI subprocess
```

---

## 7. Current Capabilities

### Supported Operations

| Operation | Implemented | Method | Notes |
|-----------|-------------|--------|-------|
| Stage files | ✅ | git2 | Via index operations |
| Unstage files | ✅ | git2 | Smart HEAD restoration |
| Stage all | ✅ | git2 | Loop + stage_path |
| View status | ✅ | git2 | With staged/unstaged sep. |
| Create commits | ✅ | git2 | With custom signature |
| View log | ✅ | git2 | Time-sorted, limit 50 |
| Configure remote | ✅ | git2 | Origin only |
| Push | ✅ | CLI | With branch rename |
| Pull | ✅ | CLI | Auto on open |
| Clone | ✅ | CLI | Repo creation only |
| Auto-push on close | ✅ | CLI | Configurable |

### Missing Operations (Gaps)

| Operation | Status | Impact |
|-----------|--------|--------|
| Branch creation | ❌ | Users stuck on main/configured branch |
| Branch switching | ❌ | No multi-branch workflows |
| Branch deletion | ❌ | No branch cleanup |
| Branch listing | ❌ | Can't see available branches |
| Diff viewing | ❌ | No visual change preview |
| Merge/rebase | ❌ | No conflict resolution UI |
| Stash | ❌ | No temporary change saving |
| Reset | ❌ | No commit history rewinding |
| Tag creation | ❌ | No release marking |
| Cherry-pick | ❌ | No selective commit application |
| Blame | ❌ | No line-by-line history |
| Fetch | ❌ | No remote tracking without pull |
| SSH key management | ✅ | Configurable in UI (path only) |
| Conflict detection | ❌ | No merge conflict indicators |

---

## 8. Architecture Notes

### Hybrid Approach: git2 + CLI

The implementation mixes two approaches:
- **git2 library** (Rust): Used for local index operations (stage, unstage, status, commit, log)
- **Git CLI subprocess** (external): Used for remote operations (push, pull, clone)

**Rationale**:
- git2 is reliable for index/tree/object operations
- CLI is simpler for remote auth (SSH keys, credentials)
- git2 can struggle with authentication flows

### Configuration Storage

Git config is stored in `vault.toml`:
```toml
[sync]
remote_url = "https://github.com/user/repo.git"
remote_branch = "main"
pull_on_open = true
push_on_close = false
ssh_key_path = "/home/user/.ssh/id_rsa"
```

**Note**: SSH key path is stored but only used in subprocess commands. No validation that it exists.

### Error Handling

- **git2 errors**: Converted to `CoreError::Git` via thiserror
- **CLI errors**: Captured from subprocess stderr, returned as string error messages
- **Frontend**: Displays errors in output textarea

---

## 9. Limitations & Constraints

1. **Single branch only**: No branch management UI
2. **Main branch default**: Hardcoded "main" as default in config
3. **Linear history only**: No merge/rebase visualization
4. **Authentication**: SSH key path only, no keychain/agent integration
5. **No conflict resolution**: If pull creates conflicts, user must resolve manually
6. **Auto-sync not visible**: Background pull/push on open/close has no UI feedback
7. **No stash support**: Changed files can't be temporarily saved
8. **Commit signature static**: Uses "slateVault User <user@slatevault.local>" by default
9. **Limit of 50 commits**: Log view hardcoded to 50 commits
10. **No diff viewer**: Changes can't be previewed before staging

---

## 10. Code Locations Summary

| Component | File | Lines | Type |
|-----------|------|-------|------|
| Core git ops | `src-tauri/crates/slatevault-core/src/vault.rs` | 275-453 | Rust |
| Tauri commands | `src-tauri/src/commands.rs` | 187-355 | Rust |
| Tauri entry | `src-tauri/src/lib.rs` | 33-87 | Rust |
| GitPanel component | `src/components/git/GitPanel.tsx` | 1-40 | React/TS |
| Changes tab | `src/components/git/ChangesTab.tsx` | 1-152 | React/TS |
| History tab | `src/components/git/HistoryTab.tsx` | 1-60 | React/TS |
| Remote tab | `src/components/git/RemoteTab.tsx` | 1-148 | React/TS |
| Git store | `src/stores/gitStore.ts` | 1-107 | Zustand |
| Command wrappers | `src/lib/commands.ts` | 83-126 | TypeScript |
| MCP server | `src-tauri/crates/slatevault-mcp/src/server.rs` | 1-270 | Rust |
| Config types | `src-tauri/crates/slatevault-core/src/config.rs` | 1-95 | Rust |

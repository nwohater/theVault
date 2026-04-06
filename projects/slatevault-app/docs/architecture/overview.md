---
id: 2b87ba8a-ba64-4763-9eee-4f433f4bbc42
title: slateVault Architecture Overview
author: ai
tags:
- architecture
- design
- overview
created: 2026-04-05T14:58:55.882860700Z
modified: 2026-04-05T14:58:55.882860700Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# slateVault Architecture Overview

slateVault is a local-first, AI-native markdown document vault built with Tauri (Rust backend) and Next.js/React (TypeScript frontend). It provides document management, full-text search, git integration, and MCP (Model Context Protocol) server capabilities.

## Tech Stack

**Frontend:**
- Next.js 15.3 (React 19) with TypeScript
- Zustand for state management
- Tailwind CSS for styling
- CodeMirror 6 for markdown editing
- xterm for terminal emulation
- gray-matter for frontmatter parsing

**Backend:**
- Tauri 2.10.3 (Rust)
- slatevault-core crate (document/vault logic)
- slatevault-mcp crate (MCP server implementation)
- git2 for git operations
- SQLite with FTS5 for full-text search
- portable-pty for terminal emulation
- rmcp for MCP protocol implementation

## Architecture Layers

### 1. Desktop Application (Tauri)
- Window management and native OS integration
- Inter-process communication via invoke handlers
- Plugin system (shell, dialog)
- Icon/resources management

**Entry Point:** `/src-tauri/src/lib.rs` - configures Tauri builders and command handlers

### 2. Rust Backend (Multi-crate)

#### Main App Crate: `/src-tauri/src/`
- `lib.rs` - Tauri builder configuration
- `commands.rs` - Tauri command handlers (17KB file)
- `mcp_manager.rs` - MCP server lifecycle management
- `terminal.rs` - PTY-based terminal implementation

#### Core Crate: `/src-tauri/crates/slatevault-core/src/`
- `vault.rs` - Main Vault struct and operations (478 lines)
- `document.rs` - Document struct, parsing, serialization
- `project.rs` - Project struct and file management
- `config.rs` - TOML configuration structures
- `search.rs` - SQLite FTS5 search index
- `error.rs` - Error types

#### MCP Crate: `/src-tauri/crates/slatevault-mcp/src/`
- `server.rs` - MCP server handler with tool routing
- `tools.rs` - MCP tool parameter definitions
- `main.rs` - Binary entry point

### 3. React Frontend: `/src/`

#### State Management (Zustand)
- `stores/vaultStore.ts` - Vault and project state (162 lines)
- `stores/editorStore.ts` - Document editing state (77 lines)
- `stores/gitStore.ts` - Git operations state (107 lines)
- `stores/uiStore.ts` - UI layout and theme state (84 lines)

#### Components (21 components total)

**Shell & Layout:**
- `components/AppShell.tsx` - Main application container with layout
- `components/StatusBar.tsx` - Status bar component
- `components/Onboarding.tsx` - Initial setup flow

**Sidebar:**
- `components/sidebar/Sidebar.tsx` - Main sidebar with tabs
- `components/sidebar/FileTree.tsx` - Project/document tree
- `components/sidebar/TreeNode.tsx` - Tree node renderer
- `components/sidebar/SearchBar.tsx` - Project/document search

**Editor:**
- `components/editor/EditorPane.tsx` - Main editor wrapper
- `components/editor/CodeMirrorEditor.tsx` - CodeMirror integration
- `components/editor/FrontMatterBar.tsx` - Frontmatter editor

**Preview:**
- `components/preview/MarkdownPreview.tsx` - Markdown rendering

**Git:**
- `components/git/GitPanel.tsx` - Git interface
- `components/git/ChangesTab.tsx` - Staging/unstaging
- `components/git/HistoryTab.tsx` - Commit history
- `components/git/RemoteTab.tsx` - Remote configuration

**Search:**
- `components/search/SearchView.tsx` - Full-text search interface

**Settings:**
- `components/settings/SettingsPanel.tsx` - Vault settings

**Other:**
- `components/vault/VaultPicker.tsx` - Vault selection/creation
- `components/terminal/TerminalPanel.tsx` - Terminal emulation
- `components/shared/EmptyState.tsx` - Empty state UI
- `components/shared/ResizeHandle.tsx` - Resizable divider

#### Library Code
- `lib/commands.ts` - Tauri invoke wrappers (216 lines)
- `lib/frontmatter.ts` - Frontmatter parsing helper

#### Type Definitions
- `types/index.ts` - All TypeScript interfaces

#### Pages
- `app/layout.tsx` - Root layout
- `app/page.tsx` - Main page (delegates to AppShell)
- `app/globals.css` - Global styles

## Data Flow Architecture

### Vault Lifecycle

```
1. User selects/creates vault
   ↓
2. openVault() command → Vault::open() (Rust)
   ↓
3. Stores active vault in ~/.slatevault/active-vault
   ↓
4. Load projects via listProjects() command
   ↓
5. Auto-start MCP server (if enabled)
   ↓
6. Rebuild search index on first open
```

### Document Editing Flow

```
1. User clicks document in sidebar
   ↓
2. EditorStore.openDocument()
   ↓
3. readDocument() command → Vault.read_document()
   ↓
4. Parse frontmatter with gray-matter
   ↓
5. Display in CodeMirrorEditor
   ↓
6. User edits content (tracked by isDirty flag)
   ↓
7. On save: EditorStore.saveDocument()
   ↓
8. writeDocument() command → Vault.write_document()
   ↓
9. Update search index and git stage (if auto_stage_ai_writes)
```

### Search Flow

```
User enters query in SearchBar
   ↓
searchDocuments() command
   ↓
Vault.search_documents() → SearchIndex.search()
   ↓
SQLite FTS5 query with optional project filter
   ↓
Return snippet-highlighted results
   ↓
Results displayed in SearchView
```

### Git Operations Flow

```
User actions in Git panel → GitStore methods
   ↓
git_* commands (stage, unstage, commit, push, pull)
   ↓
Vault methods for git2 integration
   ↓
Update GitStore with status/log
   ↓
Display in ChangesTab/HistoryTab/RemoteTab
```

## Command Handler Registry

All commands defined in `src-tauri/src/lib.rs` invoke_handler:

**Vault Commands:**
- `create_vault`, `open_vault`
- `create_project`, `list_projects`, `delete_project`, `rename_project`
- `write_document`, `read_document`, `list_documents`, `delete_document`, `rename_document`
- `search_documents`, `get_project_context`
- `rebuild_index`, `vault_stats`

**Git Commands:**
- `git_commit`, `git_status`, `git_stage`, `git_unstage`, `git_log`
- `git_remote_config`, `git_set_remote_config`
- `git_clone`, `git_push`, `git_pull`

**Config Commands:**
- `get_vault_config`, `set_vault_config`

**File Commands:**
- `show_in_folder`

**Terminal Commands:**
- `spawn_terminal`, `write_terminal`, `resize_terminal`, `close_terminal`

**MCP Commands:**
- `start_mcp_server`, `stop_mcp_server`, `mcp_server_status`

## MCP Server Architecture

The MCP server runs as a separate process managed by McpProcessState.

**Tools Exposed (via slatevault-mcp crate):**
- `create_project` - Create new project
- `list_projects` - List all projects with metadata
- `get_project_context` - Load pinned AI context files
- `write_document` - Create/overwrite document with auto-staged git support
- `read_document` - Read document with frontmatter
- `list_documents` - List documents with optional tag filter
- `search_documents` - Full-text search (FTS5 syntax)

**Server Info:** Includes instructions for AI tools on proper usage

**Auto-Staging:** When `auto_stage_ai_writes=true` in config, documents written with `ai_tool` set are automatically staged for git commit.

## State Management Pattern

All stores use Zustand with async actions:

```typescript
interface Store {
  // State
  property: Type;
  
  // Actions (async or sync)
  action: () => Promise<void>;
}

create<Store>((set, get) => ({
  // Implementations use set() and get()
}))
```

**Key Stores:**
- `vaultStore` - Vault path, projects, documents, expanded state
- `editorStore` - Active document, content, frontmatter, dirty flag
- `gitStore` - File status, commits, remote config
- `uiStore` - Layout dimensions, visibility toggles, theme

## Configuration System

### Vault Config (vault.toml)
```toml
[vault]
name = "My Vault"
version = "0.1.0"

[sync]
remote_url = "https://..."
remote_branch = "main"
pull_on_open = true
push_on_close = false

[mcp]
enabled = true
port = 3742
auto_stage_ai_writes = true
```

### Project Config (projects/{name}/project.toml)
```toml
[project]
name = "project-name"
description = "..."
tags = ["tag1", "tag2"]
ai_context_files = ["docs/spec.md"]
```

### Document Frontmatter (YAML)
```yaml
---
id: uuid
title: "Document Title"
author: ai|human|both
tags: [tag1, tag2]
created: ISO8601
modified: ISO8601
project: project-name
status: draft|review|final
ai_tool: "claude-code" (optional)
---
```

## Disk Structure

```
vault-root/
├── vault.toml              # Vault configuration
├── .gitignore             # Git ignore file
├── index.db               # SQLite FTS5 search index
├── .git/                  # Git repository
└── projects/
    └── {project-name}/
        ├── project.toml   # Project configuration
        └── docs/
            ├── doc1.md
            ├── doc2.md
            └── subdir/
                └── doc3.md

~/.slatevault/
└── active-vault           # Path to currently open vault
```

## Key Design Patterns

1. **Rust Struct Wrapping** - Each domain (Vault, Project, Document) is a Rust struct with associated methods

2. **Error Propagation** - `Result<T>` types with custom `CoreError` enum for detailed error handling

3. **Search Index Management** - Automatic indexing on document write; rebuild available on vault open

4. **Lazy Loading** - Projects/documents loaded on demand from filesystem

5. **Git Integration** - Leverages git2 library; automatic pull/push based on config

6. **Terminal as Subprocess** - PTY-based terminal runs parallel to Tauri app; output emitted via events

7. **MCP as Sidecar** - MCP server runs as separate process (stdio mode); lifecycle managed by Tauri

8. **Frontmatter as Metadata** - All document metadata stored as YAML frontmatter; enables offline access

## Performance Characteristics

- **Search:** FTS5 index enables fast queries; snippet generation via SQLite
- **Document Load:** Direct file read; frontmatter parsed with gray-matter
- **Git Status:** Uses libgit2 for efficient status queries
- **Indexing:** Incremental on write; full rebuild available

## Security & Permissions

- No user authentication (local-only application)
- Git credentials handled by system Git integration
- SSH key paths configurable in vault config
- All operations respect filesystem permissions

## Extension Points

1. **New Commands** - Add to `commands.rs` and register in `lib.rs`
2. **New MCP Tools** - Add to `slatevault-mcp/tools.rs` and router in `server.rs`
3. **New Store State** - Add new Zustand store in `stores/`
4. **New UI Components** - Add to `components/` with store integration
5. **New Config Options** - Extend `config.rs` structs and handle in commands

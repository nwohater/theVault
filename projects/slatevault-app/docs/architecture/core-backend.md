---
id: bbd62e01-ce9b-461a-8b22-6849014a7388
title: Core Backend Architecture (Rust)
author: ai
tags:
- backend
- rust
- core
- architecture
created: 2026-04-05T15:01:38.383935Z
modified: 2026-04-05T15:01:38.383935Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# Core Backend Architecture (Rust)

Detailed architecture of the Rust backend components and data structures.

## Workspace Structure

**Location:** `/src-tauri/`

**Crates:**
```
slatevault-app (main app)
├── src/
│   ├── lib.rs             # Tauri setup
│   ├── commands.rs        # Command handlers
│   ├── mcp_manager.rs     # MCP process lifecycle
│   └── terminal.rs        # PTY terminal
└── crates/
    ├── slatevault-core/   # Document/vault logic
    └── slatevault-mcp/    # MCP server binary
```

**Cargo.toml Workspace:** `/src-tauri/Cargo.toml:1-3`

---

## Main App: `slatevault-app`

### Entry Point: `lib.rs`

**File:** `/src-tauri/src/lib.rs` (92 lines)

**Tauri Builder Configuration:**
```rust
pub fn run() {
    tauri::Builder::default()
        .manage(VaultState(Mutex::new(None)))
        .manage(PtyState::new())
        .manage(McpProcessState::new())
        .plugin(tauri_plugin_shell::init())
        .plugin(tauri_plugin_dialog::init())
        .setup(|app| { /* window setup */ })
        .invoke_handler(tauri::generate_handler![...])
        .on_window_event(|window, event| { /* cleanup */ })
        .run(tauri::generate_context!())
}
```

**Managed State:**
1. `VaultState(Mutex<Option<Vault>>)` - Currently open vault
2. `PtyState` - Terminal session
3. `McpProcessState` - MCP server process

**Plugins:**
- `tauri_plugin_shell` - Shell command execution
- `tauri_plugin_dialog` - File dialogs
- `tauri_plugin_log` - Logging (debug builds)

**Invoke Handler:** Registers 27 commands from `commands.rs`

**Window Event:** Auto-push on close if configured

---

### Command Handlers: `commands.rs`

**File:** `/src-tauri/src/commands.rs` (517 lines)

**VaultState Wrapper:**
```rust
pub struct VaultState(pub Mutex<Option<Vault>>);

fn with_vault<F, T>(state: &State<'_, VaultState>, f: F) -> CmdResult<T>
where
    F: FnOnce(&Vault) -> Result<T, slatevault_core::CoreError>,
```

**Helper:** `with_vault()` provides ergonomic access to vault with error handling

**Command Groups:**

**Vault Commands (4):**
- `create_vault()` - Lines 21-25
- `open_vault()` - Lines 28-51 (includes pull on open)
- Create/list projects
- Delete/rename projects

**Document Commands (5):**
- `write_document()` - Lines 89-102
- `read_document()` - Lines 116-125
- `list_documents()` - Lines 128-148
- `delete_document()`
- `rename_document()`

**Search Commands (2):**
- `search_documents()` - Lines 159-177
- `get_project_context()` - Lines 180-185

**Git Commands (8):**
- `git_commit()`, `git_status()`, `git_stage()`, `git_unstage()`
- `git_log()`, `git_remote_config()`, `git_set_remote_config()`
- `git_clone()`, `git_push()`, `git_pull()`

**Config Commands (2):**
- `get_vault_config()`
- `set_vault_config()` - Lines 266-293

**Terminal Commands (4):**
- `spawn_terminal()`, `write_terminal()`, `resize_terminal()`, `close_terminal()`

**MCP Commands (3):**
- `start_mcp_server()`, `stop_mcp_server()`, `mcp_server_status()`

**Utility Commands (2):**
- `show_in_folder()`
- `rebuild_index()`, `vault_stats()`

---

### MCP Manager: `mcp_manager.rs`

**File:** `/src-tauri/src/mcp_manager.rs` (143 lines)

**McpProcessState:**
```rust
pub struct McpProcessState(Arc<Mutex<Option<McpProcess>>>);

struct McpProcess {
    child: Child,
    vault_path: String,
    port: u16,
}
```

**Binary Discovery:** `find_mcp_binary()` - Lines 34-60

Checks:
1. Next to current executable
2. In cargo target/debug directory
3. On system PATH

**Starting:** `start_mcp_server()` - Lines 63-97

- Kills existing process
- Spawns new with stdio piped
- Sets `SLATEVAULT_PATH` environment variable
- Returns in stdio mode (not network)

**Stopping:** `stop_mcp_server()` - Lines 100-111

- Kills process and waits for cleanup

**Status:** `mcp_server_status()` - Lines 122-142

Returns: `McpServerStatus { running, vault_path, port, binary_found }`

**Important:** Port is stored but not actively used (stdio mode doesn't use network)

---

### Terminal: `terminal.rs`

**File:** `/src-tauri/src/terminal.rs` (143 lines)

**PTY State:**
```rust
struct PtySession {
    master: Box<dyn MasterPty + Send>,
    writer: Box<dyn Write + Send>,
    child: Box<dyn portable_pty::Child + Send>,
}

pub struct PtyState(Arc<Mutex<Option<PtySession>>>);
```

**Spawning:** `spawn_terminal()` - Lines 21-91

1. Open PTY with portable-pty
2. Determine shell (cmd.exe on Windows, /bin/sh on others)
3. Spawn shell in PTY
4. Start reader thread emitting `terminal:output` events
5. Store master/writer/child in state

**Writing:** `write_terminal()` - Lines 94-109

- Write to PTY writer
- Flush to ensure transmission

**Resizing:** `resize_terminal()` - Lines 112-129

- Resize PTY to new rows/cols
- Affects running terminal session

**Closing:** `close_terminal()` - Lines 132-142

- Kill child process
- Clear state

**Event Loop:** Reader thread continuously reads from PTY and emits to Tauri event system

---

## Core Crate: `slatevault-core`

**Location:** `/src-tauri/crates/slatevault-core/src/`

**Purpose:** Document management, search, and git operations

### Module: `vault.rs`

**File:** `/src-tauri/crates/slatevault-core/src/vault.rs` (478 lines)

**Vault Struct:**
```rust
pub struct Vault {
    pub root: PathBuf,           // Vault directory
    pub config: VaultConfig,     // Configuration
    pub search: SearchIndex,     // FTS5 search
    repo: git2::Repository,      // Git repo
}
```

**Key Methods:**

**Creation:** `Vault::create()` - Lines 17-57

```rust
pub fn create(root: &Path, name: &str) -> Result<Self> {
    // Create directories
    // Write vault.toml
    // Write .gitignore
    // Initialize git repo
    // Initialize search index
}
```

**Opening:** `Vault::open()` - Lines 59-78

```rust
pub fn open(root: &Path) -> Result<Self> {
    // Read vault.toml
    // Open git repo
    // Open search index
}
```

**Indexing:** `rebuild_index()` - Lines 84-104

```rust
pub fn rebuild_index(&self) -> Result<usize> {
    // Walk all projects
    // Index all documents
    // Return count
}
```

**Statistics:** `stats()` - Lines 106-123

```rust
pub fn stats(&self) -> Result<VaultStats> {
    // Count projects and documents
    // Return stats
}
```

**Project Operations (3 methods):**
- `create_project()` - Lines 127-134
- `open_project()` - Lines 136-138
- `list_projects()` - Lines 140-142

**Document Operations (4 methods):**

**Write:** `write_document()` - Lines 146-181

```rust
pub fn write_document(
    &self,
    project_name: &str,
    path: &str,
    title: &str,
    content: &str,
    tags: Vec<String>,
    ai_tool: Option<String>,
) -> Result<Document> {
    // Create Document with frontmatter
    // Write to file
    // Index for search
    // Auto-stage if configured
}
```

**Read:** `read_document()` - Lines 183-196

```rust
pub fn read_document(&self, project_name: &str, path: &str) -> Result<Document>
```

**List:** `list_documents()` - Lines 198-213

```rust
pub fn list_documents(
    &self,
    project_name: &str,
    tag_filter: Option<&[String]>,
) -> Result<Vec<Document>> {
    // Recursive walk of docs directory
    // Filter by tags if provided
    // Parse each document
}
```

**Search:** `search_documents()` - Line ~355

Delegates to `search.search()`

**Get Project Context:** `get_project_context()` - Line ~380

Reads files listed in `project.toml` ai_context_files

**Git Operations (7 methods):**
- `commit()` - Creates commit
- `status()` - File status
- `stage_path()` / `stage_file()` - Stage file
- `unstage_file()` - Unstage file
- `log()` - Commit history
- `set_git_remote()` - Configure remote
- `pull()` / `push()` - Sync with remote
- `save_config()` - Write vault.toml

**Git Auto-Actions:**
- `pull_on_open` - Auto-pull when vault opens (in commands.rs:34)
- `push_on_close` - Auto-push when app closes (in lib.rs:76)
- `auto_stage_ai_writes` - Auto-stage documents written with ai_tool (vault.rs:176)

---

### Module: `document.rs`

**File:** `/src-tauri/crates/slatevault-core/src/document.rs` (119 lines)

**Frontmatter Struct:**
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct FrontMatter {
    pub id: Uuid,
    pub title: String,
    pub author: Author,
    pub tags: Vec<String>,
    pub created: DateTime<Utc>,
    pub modified: DateTime<Utc>,
    pub project: String,
    pub status: DocStatus,
    pub ai_tool: Option<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub enum Author {
    Human,
    Ai,
    Both,
}

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub enum DocStatus {
    Draft,
    Review,
    Final,
}
```

**Document Struct:**
```rust
#[derive(Debug, Clone)]
pub struct Document {
    pub front_matter: FrontMatter,
    pub content: String,
    pub path: String,
}
```

**Parsing:** `Document::parse()` - Lines 46-54

```rust
pub fn parse(raw: &str, path: &str) -> Result<Self> {
    // Split frontmatter and content
    // Deserialize YAML frontmatter
    // Return Document
}
```

**Serialization:** `Document::to_string()` - Lines 56-59

```rust
pub fn to_string(&self) -> Result<String> {
    // Serialize frontmatter to YAML
    // Combine with content
    // Return markdown with frontmatter header
}
```

**Creation:** `Document::new()` - Lines 61-90

```rust
pub fn new(
    title: String,
    content: String,
    project: String,
    path: String,
    tags: Vec<String>,
    ai_tool: Option<String>,
) -> Self {
    // Generate UUID
    // Set timestamps to now
    // Determine author from ai_tool
    // Return Document
}
```

**Format:**
```markdown
---
id: {uuid}
title: "Title"
author: human|ai|both
tags: [tag1, tag2]
created: {ISO8601}
modified: {ISO8601}
project: {project}
status: draft|review|final
ai_tool: "tool-name"  # optional
---

{content}
```

**Frontmatter Splitting:** `split_front_matter()` - Lines 93-110

- Finds `---` delimiters
- Extracts YAML between them
- Returns (frontmatter_str, content_str)

---

### Module: `project.rs`

**File:** `/src-tauri/crates/slatevault-core/src/project.rs` (72 lines)

**Project Struct:**
```rust
pub struct Project {
    pub config: ProjectConfig,
    pub root: PathBuf,
}
```

**Creation:** `Project::create()` - Lines 12-33

```rust
pub fn create(
    projects_dir: &Path,
    name: &str,
    description: &str,
    tags: Vec<String>,
) -> Result<Self> {
    // Create projects/{name}/ directory
    // Create projects/{name}/docs/ subdirectory
    // Write project.toml config
}
```

**Opening:** `Project::open()` - Lines 35-45

```rust
pub fn open(projects_dir: &Path, name: &str) -> Result<Self> {
    // Read projects/{name}/project.toml
    // Deserialize config
}
```

**Docs Directory:** `docs_dir()` - Line 47

Returns: `root.join("docs")`

**Listing:** `Project::list_all()` - Lines 51-71

```rust
pub fn list_all(projects_dir: &Path) -> Result<Vec<ProjectConfig>> {
    // Iterate projects_dir
    // For each directory with project.toml
    // Parse and collect config
}
```

---

### Module: `config.rs`

**File:** `/src-tauri/crates/slatevault-core/src/config.rs` (95 lines)

**VaultConfig:**
```rust
pub struct VaultConfig {
    pub vault: VaultMeta,
    pub sync: SyncConfig,
    pub mcp: McpConfig,
}

pub struct VaultMeta {
    pub name: String,
    pub version: String,
}
```

**SyncConfig:**
```rust
pub struct SyncConfig {
    pub remote_url: Option<String>,
    pub remote_branch: String,      // default: "main"
    pub pull_on_open: bool,         // default: true
    pub push_on_close: bool,        // default: false
    pub ssh_key_path: Option<String>,
}
```

**McpConfig:**
```rust
pub struct McpConfig {
    pub enabled: bool,               // default: true
    pub port: u16,                   // default: 3742
    pub auto_stage_ai_writes: bool,  // default: true
}
```

**ProjectConfig:**
```rust
pub struct ProjectConfig {
    pub project: ProjectMeta,
}

pub struct ProjectMeta {
    pub name: String,
    pub description: String,
    pub tags: Vec<String>,
    pub ai_context_files: Vec<String>,
}
```

**Serialization:** All use `serde` with TOML format

**vault.toml Example:**
```toml
[vault]
name = "My Vault"
version = "0.1.0"

[sync]
remote_url = null
remote_branch = "main"
pull_on_open = true
push_on_close = false

[mcp]
enabled = true
port = 3742
auto_stage_ai_writes = true
```

**project.toml Example:**
```toml
[project]
name = "my-project"
description = "My project"
tags = []
ai_context_files = ["docs/spec.md"]
```

---

### Module: `search.rs`

**File:** `/src-tauri/crates/slatevault-core/src/search.rs` (103 lines)

**SearchIndex:**
```rust
pub struct SearchIndex {
    conn: Connection,
}

pub struct SearchResult {
    pub project: String,
    pub path: String,
    pub title: String,
    pub snippet: String,
    pub rank: f64,
}
```

**Initialization:** `open()` - Lines 19-32

```rust
pub fn open(db_path: &Path) -> Result<Self> {
    // Create/open SQLite db
    // Create virtual FTS5 table
}
```

**FTS5 Table:**
```sql
CREATE VIRTUAL TABLE IF NOT EXISTS documents_fts USING fts5(
    project,
    path,
    title,
    content,
    tags,
    tokenize='porter'
);
```

**Indexing:** `index_document()` - Lines 34-53

```rust
pub fn index_document(
    &self,
    project: &str,
    path: &str,
    title: &str,
    content: &str,
    tags: &[String],
) -> Result<()> {
    // Delete existing entry
    // Insert new entry
}
```

**Searching:** `search()` - Lines 55-103

```rust
pub fn search(
    &self,
    query: &str,
    project: Option<&str>,
    limit: usize,
) -> Result<Vec<SearchResult>> {
    // Build FTS5 query
    // Execute with optional project filter
    // Generate snippets
    // Return results ordered by rank
}
```

**Snippet Function:** Uses SQLite `snippet(table, context_lines, open, close, ellipsis, max_len)`

Example: `snippet(documents_fts, 3, '<b>', '</b>', '...', 32)` - 3 context lines, bold tags, 32 char max

**FTS5 Syntax Supported:**
- AND, OR, NOT operators
- Phrase search with quotes
- Prefix search with *
- Column-specific search with col:term

---

### Module: `error.rs`

**File:** `/src-tauri/crates/slatevault-core/src/error.rs` (1002 lines - likely large)

**CoreError Enum:**

```rust
pub enum CoreError {
    VaultAlreadyExists(String),
    VaultNotFound(String),
    ProjectAlreadyExists(String),
    ProjectNotFound(String),
    DocumentNotFound(String),
    InvalidFrontMatter(String),
    Io(io::Error),
    TomlError(toml::de::Error),
    GitError(git2::Error),
    SqliteError(rusqlite::Error),
    YamlError(serde_yaml::Error),
    // ... more variants
}
```

**Trait Implementations:**
- `From<io::Error>`
- `From<toml::de::Error>`
- `From<git2::Error>`
- `From<rusqlite::Error>`
- `Display` for error messages

**Result Type:** `pub type Result<T> = std::result::Result<T, CoreError>;`

---

## MCP Crate: `slatevault-mcp`

**Location:** `/src-tauri/crates/slatevault-mcp/src/`

**Purpose:** MCP server binary with tool implementations

### Main: `main.rs`

**File:** `/src-tauri/crates/slatevault-mcp/src/main.rs` (70 lines)

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Get SLATEVAULT_PATH from env
    // Initialize SlateVaultMcpServer
    // Run server on stdio with rmcp
}
```

**Server Initialization:**
```rust
let vault_path = std::env::var("SLATEVAULT_PATH")
    .expect("SLATEVAULT_PATH environment variable not set");

let server = SlateVaultMcpServer::new(PathBuf::from(vault_path));
StdioServerTransport::new(stdin(), stdout()).run(server).await?
```

---

### Server: `server.rs`

**File:** `/src-tauri/crates/slatevault-mcp/src/server.rs` (271 lines)

**SlateVaultMcpServer Struct:**
```rust
#[derive(Clone)]
pub struct SlateVaultMcpServer {
    vault_path: PathBuf,
    tool_router: ToolRouter<Self>,
}
```

**Tool Router:** Auto-generated from `#[tool_router]` macro

**ServerHandler Implementation:**
```rust
#[tool_handler]
impl ServerHandler for SlateVaultMcpServer {
    fn get_info(&self) -> ServerInfo {
        // Return server info with instructions
        // Enable tools capability
    }
}
```

**Helper:**
```rust
fn open_vault(&self) -> Result<Vault, McpError> {
    Vault::open(&self.vault_path)
        .map_err(|e| McpError::internal_error(format!(...), None))
}
```

**Tool Methods (7):**
Each decorated with `#[tool(description = "...")]`

1. `create_project()` - Lines 37-55
2. `list_projects()` - Lines 57-89
3. `get_project_context()` - Lines 91-114
4. `write_document()` - Lines 116-151
5. `read_document()` - Lines 153-175
6. `list_documents()` - Lines 177-207
7. `search_documents()` - Lines 209-235

**Return Type:** All return `Result<CallToolResult, McpError>`

**Success Pattern:**
```rust
Ok(CallToolResult::success(vec![Content::text(output)]))
```

**Error Pattern:**
```rust
Err(McpError::internal_error(format!("Error: {}", e), None))
```

---

### Tools: `tools.rs`

**File:** `/src-tauri/crates/slatevault-mcp/src/tools.rs` (68 lines)

**Parameter Structs:** 7 structs for tool parameters

Each implements:
- `#[derive(Debug, Deserialize, JsonSchema)]`
- `#[schemars(description = "...")]` on struct and fields

**Parameter Structs:**
1. `CreateProjectParams`
2. `GetProjectContextParams`
3. `WriteDocumentParams`
4. `ReadDocumentParams`
5. `ListDocumentsParams`
6. `SearchDocumentsParams`

**JSON Schema Generation:** Automatic via `rmcp::schemars`

---

## Dependencies

**Core Dependencies** (from workspace Cargo.toml:10-23):

- `serde` / `serde_json` - Serialization
- `serde_yaml` - YAML parsing
- `toml` - TOML config
- `uuid` - Document IDs
- `chrono` - Timestamps
- `thiserror` - Error handling
- `anyhow` - Error context
- `log` - Logging
- `rusqlite` - SQLite FTS5
- `git2` - Git operations
- `tokio` - Async runtime

**Tauri-Specific:**
- `tauri` - Desktop framework
- `tauri-plugin-log` - Logging
- `tauri-plugin-shell` - Shell integration
- `tauri-plugin-dialog` - File dialogs
- `portable-pty` - Terminal emulation

**MCP-Specific:**
- `rmcp` - MCP protocol implementation

---

## Data Flow: Document Write

```
Frontend (writeDocument command)
    ↓ invoke
Tauri (commands.rs::write_document)
    ↓ Vault::write_document()
Core (vault.rs::write_document)
    ↓ Document::new()
Core (document.rs)
    ├─ Generate UUID
    ├─ Set timestamps
    ├─ Determine author from ai_tool
    └─ Create Document struct
    ↓
Core (vault.rs)
    ├─ Create file path
    ├─ Create parent directories
    ├─ Serialize document to YAML frontmatter
    ├─ Write file
    ├─ Index in SearchIndex
    └─ Auto-stage in git (if configured)
    ↓
Frontend (editorStore updates)
```

---

## Data Flow: Search

```
Frontend (searchDocuments command)
    ↓
Tauri (commands.rs::search_documents)
    ↓ Vault::search_documents()
Core (vault.rs)
    ↓
SearchIndex::search()
Core (search.rs)
    ├─ Build FTS5 SQL query
    ├─ Execute against documents_fts table
    ├─ Generate snippets with SQLite snippet()
    └─ Order by rank
    ↓
SearchResult[]
    ↓
Frontend (SearchView renders)
```

---

## Thread Safety & Concurrency

- **VaultState:** Wrapped in `Mutex<Option<Vault>>`
- **PtyState:** Wrapped in `Arc<Mutex<Option<PtySession>>>`
- **McpProcessState:** Wrapped in `Arc<Mutex<Option<McpProcess>>>`
- **Vault Operations:** No internal locking (single writer expected)
- **Search Index:** Single SQLite connection (SQLite handles locking)
- **Git Operations:** git2 library handles internal locking

---

## Error Handling Pattern

All Rust functions return `Result<T, CoreError>`:

1. Try operation
2. Map errors via `.map_err(|e| e.to_string())` for Tauri
3. Frontend receives string error message
4. Frontend displays in UI or console

No panics in core operations - all errors propagate as Result types

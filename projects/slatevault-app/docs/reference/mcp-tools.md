---
id: 99f7f833-c2df-4415-a10d-fcbceafe188e
title: MCP Tools Reference
author: ai
tags:
- mcp
- tools
- reference
- api
created: 2026-04-05T15:00:02.420517700Z
modified: 2026-04-05T15:00:02.420517700Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# MCP Tools Reference

Complete reference for all MCP tools exposed by the slatevault-mcp server.

## Overview

The MCP server is implemented in `/src-tauri/crates/slatevault-mcp/src/`:
- `server.rs` - Tool routing and handler (271 lines)
- `tools.rs` - Parameter definitions with JSON schemas (68 lines)
- `main.rs` - Binary entry point

**Server Handler:** `SlateVaultMcpServer` struct implementing `ServerHandler` trait

**Tool Router:** Uses `#[tool_router]` macro to register tools automatically

## Startup Workflow (from ServerInfo)

The MCP server provides these instructions to AI tools:

1. Call `list_projects` to see what projects exist
2. If relevant project exists, call `get_project_context` to load pinned AI context files
3. If no relevant project exists, ask user which project to use or whether to create one

## Best Practices (from ServerInfo)

**Writing Documents:**
- Use `write_document` to save documentation, specs, decisions, or notes
- Choose descriptive paths: `'architecture.md'` or `'decisions/001-use-tauri.md'`
- Always set `ai_tool` to your tool name (e.g. `'claude-code'`) for attribution
- Documents are auto-staged for git commit when `ai_tool` is set

**Reading and Searching:**
- Use `search_documents` before writing to check if a document exists
- Use `read_document` to load existing docs for context or updates
- Use `list_documents` to see all docs in a project

**Organization:**
- Organize docs by type: `specs/`, `decisions/`, `guides/`, `notes/`
- Update existing documents rather than creating duplicates
- Use tags to categorize documents for easy filtering
- Do NOT create projects without asking the user first

---

## Tool: `create_project`

Creates a new project folder in the vault.

**Location:** `/src-tauri/crates/slatevault-mcp/src/server.rs:37-55`

**Definition:** `/src-tauri/crates/slatevault-mcp/src/tools.rs:5-14`

```typescript
Parameters {
  name: string         // Project folder name in slug format (e.g. 'my-app')
  description?: string // Short description of the project
  tags?: string[]      // Initial tags for the project
}

Returns: CallToolResult  // "Project '{name}' created successfully"
```

**Rust Implementation:**
```rust
fn create_project(
    &self,
    Parameters(params): Parameters<CreateProjectParams>,
) -> Result<CallToolResult, McpError>
```

**Creates:**
- Directory: `projects/{name}/`
- Subdirectory: `projects/{name}/docs/`
- Config file: `projects/{name}/project.toml`

**Error Cases:**
- Project already exists
- Invalid name format
- Filesystem permissions

---

## Tool: `list_projects`

Returns all projects with metadata and document counts.

**Location:** `server.rs:57-89`

**Parameters:** None

**Returns:** Text output listing all projects

```
## {ProjectName}
- Description: {...}
- Tags: [tag1, tag2]

## {AnotherProject}
...
```

**Empty Case:** Returns "No projects found in vault."

**Rust Implementation:**
```rust
fn list_projects(&self) -> Result<CallToolResult, McpError>
```

**Usage:** Always call this first to understand vault structure

---

## Tool: `get_project_context`

Returns content of all AI context files pinned for a project.

**Location:** `server.rs:91-114`

**Definition:** `tools.rs:16-21`

```typescript
Parameters {
  project: string  // Project name
}

Returns: Text with concatenated file contents
```

**Output Format:**
```
# {file_path}

{file content}

---

# {another_path}

{file content}

---
```

**Configuration:** Set `ai_context_files` in `projects/{name}/project.toml`:

```toml
[project]
name = "my-project"
ai_context_files = ["docs/architecture.md", "docs/spec.md"]
```

**Empty Case:** Returns message suggesting to add files to `project.toml`

**Use Case:** Load project documentation as context before writing new documents

---

## Tool: `write_document`

Creates or overwrites a markdown document in a project.

**Location:** `server.rs:116-151`

**Definition:** `tools.rs:23-38`

```typescript
Parameters {
  project: string       // Target project name
  path: string          // Path relative to docs/ folder
  title: string         // Document title (used in front matter)
  content: string       // Full markdown body (no front matter)
  tags?: string[]       // Tags to apply
  ai_tool?: string      // Name of AI tool calling this (e.g. 'claude-code')
}

Returns: Text success message with document metadata
```

**Output Example:**
```
Document written: my-project/docs/architecture.md
- ID: {uuid}
- Author: Ai
- Status: Draft
- Auto-staged for git commit
```

**Auto-Generated Frontmatter:**
```yaml
---
id: {uuid}
title: "Document Title"
author: ai  # (or 'human' or 'both' based on ai_tool)
tags: [tag1, tag2]
created: {ISO8601}
modified: {ISO8601}
project: my-project
status: draft
ai_tool: "claude-code"
---
```

**Auto-Staging:** When `mcp.auto_stage_ai_writes = true` in `vault.toml` and `ai_tool` is provided, the document is automatically staged for git commit

**File Structure:**
- Path can include directories: `"decisions/001-use-tauri.md"` creates subdirectories as needed
- Must be relative to `projects/{project}/docs/`
- File extension must be `.md`

**Example:**
```
write_document({
  project: "my-app",
  path: "decisions/001-use-tauri.md",
  title: "Decision: Use Tauri for Desktop",
  content: "We chose Tauri because...",
  tags: ["decision", "architecture"],
  ai_tool: "claude-code"
})
```

---

## Tool: `read_document`

Reads a document by project and path.

**Location:** `server.rs:153-175`

**Definition:** `tools.rs:40-47`

```typescript
Parameters {
  project: string  // Project name
  path: string     // Path relative to docs/ folder
}

Returns: Text with full document (frontmatter + content)
```

**Output Format:**
```
---
title: Document Title
author: Ai
status: Draft
tags: [tag1, tag2]
created: {ISO8601}
modified: {ISO8601}
---

{document content}
```

**Error Cases:**
- Document not found
- Invalid project name
- Malformed frontmatter

**Usage:** Load existing documents to check current state or use as context

---

## Tool: `list_documents`

Lists all documents in a project with optional tag filtering.

**Location:** `server.rs:177-207`

**Definition:** `tools.rs:49-56`

```typescript
Parameters {
  project: string    // Project name
  tags?: string[]    // Filter to docs with ALL these tags
}

Returns: Text listing all documents
```

**Output Format:**
```
- **Document Title** (`path/to/doc.md`)
  Author: Ai | Status: Draft | Tags: [tag1, tag2]

- **Another Document** (`another.md`)
  Author: Human | Status: Final | Tags: [tag3]
```

**Tag Filtering:** If `tags = ["architecture"]` specified, only returns documents tagged with "architecture"

**Empty Case:** Returns "No documents found in project '{project}'."

**Recursive:** Walks all subdirectories in `projects/{project}/docs/`

**Usage:** Discover existing documents before writing new ones to avoid duplicates

---

## Tool: `search_documents`

Full-text search across the vault or a single project using SQLite FTS5.

**Location:** `server.rs:209-235`

**Definition:** `tools.rs:58-67`

```typescript
Parameters {
  query: string     // Search query (SQLite FTS5 syntax supported)
  project?: string  // Scope search to a single project (optional)
  limit?: number    // Maximum results to return (default: 20)
}

Returns: Text with search results
```

**Output Format:**
```
Found 2 result(s):

- **Document Title** (`project-name/docs/path.md`)
  Search snippet with <b>highlighted</b> matches...

- **Another Title** (`other-project/docs/file.md`)
  Another snippet with matches...
```

**FTS5 Query Syntax:**
- `term` - Search for term
- `"exact phrase"` - Search for exact phrase
- `term1 OR term2` - Either term
- `term1 AND term2` - Both terms
- `NOT term` - Exclude term
- `col:term` - Search in specific column (project, path, title, content, tags)
- `term*` - Prefix search
- `"phrase"*` - Phrase prefix

**Columns Indexed:** project, path, title, content, tags

**Tokenizer:** Porter stemmer for better matching

**Search Scope:**
- Without `project`: Searches entire vault across all projects
- With `project`: Searches only that project

**Snippet Generation:** SQLite generates 3-line snippets with HTML bold tags

**Empty Results:** Returns "No results found for query: {query}"

**Example Queries:**
```
"architecture decisions"     # Phrase search
vault AND design            # Both terms
specification* OR spec*     # Prefix matching
title:decision              # Search in title column
NOT deprecated              # Exclude term
```

**Usage:** Find relevant existing documents before writing; understand what's already documented

---

## Tool: Parameter Types

All tool parameter types are defined in `/src-tauri/crates/slatevault-mcp/src/tools.rs`

Each uses `#[schemars(description = "...")]` for JSON schema generation

**ImportingParams:** Uses `rmcp::schemars` for automatic schema generation

```rust
use rmcp::schemars;
use schemars::JsonSchema;
use serde::Deserialize;

#[derive(Debug, Deserialize, JsonSchema)]
#[schemars(description = "Tool description")]
pub struct ToolParams {
    #[schemars(description = "Field description")]
    pub field: String,
}
```

---

## ServerInfo & Capabilities

**Location:** `server.rs:238-270`

**Implements:** `ServerHandler` trait

**Functions:**
- `get_info()` - Returns `ServerInfo` with instructions and capabilities
- `tool_router()` - Returns configured `ToolRouter<Self>`

**Capabilities Enabled:** Tools only

**Instructions:** Detailed startup workflow and best practices (included in tool descriptions above)

---

## Error Handling

All tools use `Result<CallToolResult, McpError>` pattern:

```rust
// Success
Ok(CallToolResult::success(vec![Content::text(output)]))

// Error
Err(McpError::internal_error(format!("Error message"), None))
```

**Common Errors:**
- Vault not found/accessible
- Project does not exist
- Document not found
- Invalid parameters
- Filesystem I/O errors

---

## Integration with Vault Core

All tools delegate to `slatevault_core::Vault` struct methods:

**Vault Methods Called:**
- `create_project()` - Project creation
- `list_projects()` - Project enumeration
- `get_project_context()` - Load context files
- `write_document()` - Document creation/update
- `read_document()` - Document reading
- `list_documents()` - Document enumeration with filtering
- `search_documents()` - FTS5 search via SearchIndex

**Vault Initialization:**
```rust
fn open_vault(&self) -> Result<Vault, McpError> {
    Vault::open(&self.vault_path)
}
```

**Vault Path:** Read from `SLATEVAULT_PATH` environment variable set by `mcp_manager.rs:80`

---

## Process Lifecycle

**Started By:** `start_mcp_server` Tauri command (mcp_manager.rs:63)

**Binary:** `slatevault-mcp` executable (found via `find_mcp_binary()`)

**Environment:** `SLATEVAULT_PATH` points to active vault root

**Communication:** Stdio mode (JSON-RPC 2.0 over stdin/stdout)

**Lifecycle Management:** `McpProcessState` in `mcp_manager.rs`

**On Vault Close:** MCP server stopped automatically

**On App Close:** MCP server killed via `window::on_close_event`

---

## Usage Examples

### Example 1: Create Project with Context Files

```
1. create_project({
     name: "my-docs",
     description: "Documentation project",
     tags: ["docs"]
   })

2. write_document({
     project: "my-docs",
     path: "architecture.md",
     title: "System Architecture",
     content: "Our system is built with...",
     tags: ["architecture"],
     ai_tool: "claude-code"
   })

3. Update project.toml to add:
   ai_context_files = ["docs/architecture.md"]
```

### Example 2: Research and Update

```
1. list_projects()
   # Find "my-project"

2. get_project_context({
     project: "my-project"
   })
   # Load existing documentation

3. search_documents({
     query: "database schema",
     project: "my-project"
   })
   # Find related docs

4. read_document({
     project: "my-project",
     path: "schema.md"
   })
   # Load full document

5. write_document({
     project: "my-project",
     path: "schema.md",
     title: "Database Schema",
     content: "Updated schema info...",
     ai_tool: "claude-code"
   })
   # Update with new information
```

### Example 3: Search Across Vault

```
search_documents({
  query: "deployment OR infrastructure",
  limit: 10
})
# Returns top 10 results across all projects
```

---

## Configuration for MCP

**Vault Config** (`vault.toml`):
```toml
[mcp]
enabled = true
port = 3742
auto_stage_ai_writes = true
```

**Project Config** (`project.toml`):
```toml
[project]
name = "my-project"
ai_context_files = ["docs/spec.md", "docs/architecture.md"]
```

**Environment:**
- `SLATEVAULT_PATH` - Path to active vault (set by Tauri app)

---

## Performance Characteristics

- **Search:** FTS5 provides sub-second queries for typical vaults
- **List:** Directory walk for all projects/documents
- **Read/Write:** Direct file I/O
- **Context Loading:** Stream file contents from disk
- **Index Creation:** Automatic on document write

---

## Thread Safety

- MCP server runs as separate process, no shared state with Tauri app
- All Vault operations are thread-safe (use `&self` references)
- Vault struct manages git2::Repository safely

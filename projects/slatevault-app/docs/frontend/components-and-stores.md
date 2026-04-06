---
id: 4e6942ea-6187-40e0-90f6-fd2b84766466
title: Frontend Components & Stores
author: ai
tags:
- frontend
- components
- react
- stores
created: 2026-04-05T15:00:48.753159200Z
modified: 2026-04-05T15:00:48.753159200Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# Frontend Components & Stores

Complete reference for React components and Zustand stores in the slateVault frontend.

**Location:** `/src/` (Next.js 15.3 + React 19 + TypeScript)

---

## Zustand Stores

All stores are in `/src/stores/` and use the Zustand state management library with async actions.

### Store: `vaultStore`

**File:** `/src/stores/vaultStore.ts` (162 lines)

**Purpose:** Manages vault and project state

**State:**
```typescript
{
  isOpen: boolean                           // Vault open/closed
  vaultPath: string | null                  // Current vault directory
  vaultName: string | null                  // Vault display name
  projects: ProjectInfo[]                   // All projects in vault
  documents: Record<string, DocumentInfo[]> // Docs by project
  expandedProjects: Set<string>             // Expanded tree nodes
  stats: VaultStatsInfo | null              // Vault statistics
}
```

**Actions:**
- `openVault(path)` - Open vault, load projects, auto-start MCP
- `createVault(path, name)` - Create and open vault
- `loadProjects()` - Reload projects list
- `loadDocuments(project)` - Load docs for project
- `loadStats()` - Load vault statistics
- `closeVault()` - Close vault, stop MCP
- `createProject(name, description?, tags?)` - Create project
- `deleteProject(name)` - Delete project and reload
- `deleteDocument(project, path)` - Delete document
- `renameDocument(project, oldPath, newPath)` - Rename document
- `renameProject(oldName, newName)` - Rename project
- `toggleProject(name)` - Toggle project expansion and load docs

**Key Flows:**
1. On open: Rebuild index → Load projects → Load stats → Start MCP
2. On close: Stop MCP server
3. Project expansion: Auto-loads documents when expanded

**Used By:**
- AppShell (main layout)
- Sidebar (vault header, projects list)
- FileTree (project tree rendering)

---

### Store: `editorStore`

**File:** `/src/stores/editorStore.ts` (77 lines)

**Purpose:** Manages document editing state

**State:**
```typescript
{
  activeProject: string | null  // Current project name
  activePath: string | null     // Current document path
  content: string               // Full document with frontmatter
  frontMatter: FrontMatter | null // Parsed frontmatter
  isDirty: boolean              // Document has unsaved changes
}
```

**Actions:**
- `openDocument(project, path)` - Open document (auto-saves previous if dirty)
- `updateContent(content)` - Update editor content and parse frontmatter
- `saveDocument()` - Save to backend via writeDocument command
- `closeDocument()` - Clear editor state

**Frontmatter Handling:**
- Parsed with `gray-matter` library
- Extracted from content on read
- Reconstructed on save
- Full content (with frontmatter) stored during edit

**Auto-Save:** When opening a new document, automatically saves the previous one if it was modified

**Used By:**
- EditorPane (document wrapper)
- CodeMirrorEditor (content editor)
- FrontMatterBar (metadata editor)
- AppShell (keyboard shortcuts, dirty state)

---

### Store: `gitStore`

**File:** `/src/stores/gitStore.ts` (107 lines)

**Purpose:** Manages git operations and state

**State:**
```typescript
{
  files: FileStatus[]           // Changed files with status
  commits: CommitInfo[]         // Commit history
  remoteConfig: RemoteConfig | null // Remote settings
  commitMessage: string         // Draft commit message
  loading: boolean              // Operation in progress
  output: string | null         // Last operation result
}
```

**Actions:**
- `loadStatus()` - Get current file status
- `stage(path)` - Stage single file and reload status
- `unstage(path)` - Unstage file and reload status
- `stageAll()` - Stage all unstaged files
- `commit()` - Commit staged changes with message
- `loadLog(limit?)` - Load commit history
- `loadRemoteConfig()` - Get remote configuration
- `setRemoteConfig(config)` - Update remote settings
- `setCommitMessage(msg)` - Update draft message
- `clearOutput()` - Clear operation result

**File Status Types:**
- `"staged_new"` - New file (staged)
- `"staged_modified"` - Modified file (staged)
- `"staged_deleted"` - Deleted file (staged)
- `"new"` - New file (unstaged)
- `"modified"` - Modified file (unstaged)
- `"deleted"` - Deleted file (unstaged)

**Used By:**
- GitPanel (git interface)
- ChangesTab (staging/unstaging)
- HistoryTab (commit history)
- RemoteTab (remote configuration)

---

### Store: `uiStore`

**File:** `/src/stores/uiStore.ts` (84 lines)

**Purpose:** Manages UI layout and theme

**State:**
```typescript
{
  sidebarWidth: number      // Sidebar width (180-500px)
  showEditor: boolean       // Show/hide editor pane
  showPreview: boolean      // Show/hide preview pane
  previewRatio: number      // Editor/preview split (0.2-0.8)
  activeView: ActiveView    // "editor" | "search"
  showTerminal: boolean     // Show/hide terminal
  terminalHeight: number    // Terminal height (100-600px)
  theme: Theme              // "dark" | "light" | "olive" | "deepblue"
}
```

**Type: ActiveView:**
```typescript
type ActiveView = "editor" | "search";
```

**Type: Theme:**
```typescript
type Theme = "dark" | "light" | "olive" | "deepblue";
```

**Actions:**
- `setSidebarWidth(width)` - Set sidebar width with clamping
- `toggleEditor()` - Toggle editor visibility
- `togglePreview()` - Toggle preview visibility
- `setPreviewRatio(ratio)` - Set editor/preview split
- `setActiveView(view)` - Switch between editor and search
- `toggleTerminal()` - Toggle terminal visibility
- `setTerminalHeight(height)` - Set terminal height
- `setTheme(theme)` - Set theme with localStorage persistence

**Theme Persistence:**
- Saved to `localStorage["sv-theme"]`
- Applied to `document.documentElement.dataset.theme`
- Dark theme removes attribute; others set it

**Dimension Clamping:**
- Sidebar: 180px min, 500px max
- Terminal: 100px min, 600px max
- Preview ratio: 0.2-0.8

**Used By:**
- AppShell (layout container, keyboard shortcuts)
- ResizeHandle (sidebar and terminal resizing)
- TerminalPanel (terminal visibility)
- Settings (theme selection)

---

## React Components

Components are in `/src/components/` and organized by feature.

### Component: `AppShell`

**File:** `/src/components/AppShell.tsx` (full component)

**Purpose:** Main application container and layout orchestration

**Structure:**
```
AppShell (flex column, full height)
├─ Sidebar (fixed width, resizable)
├─ ResizeHandle (sidebar/content separator)
├─ Main Content (flex, split between editor/preview)
│  ├─ Editor Section (flex-shrink based on preview)
│  ├─ ResizeHandle (editor/preview separator)
│  └─ Preview Section (flex-shrink based on editor)
├─ Terminal (fixed bottom, resizable)
└─ StatusBar (fixed bottom)
```

**Key Features:**
- Shows `VaultPicker` if no vault is open
- Keyboard shortcuts:
  - `Ctrl+Shift+F` - Toggle search/editor view
  - `Ctrl+T` - Toggle terminal
- Restores theme from localStorage on mount
- Calculates flex ratios for split panes
- Auto-saves dirty documents on close

**State Management:**
- Uses all 4 stores (vault, editor, git, ui)
- Coordinates between stores on actions

**Used By:** Root layout

---

### Component: `Sidebar`

**File:** `/src/components/sidebar/Sidebar.tsx` (80+ lines)

**Purpose:** Side panel with project/file browser and settings

**Tabs:**
1. **Files** - Project tree and documents (FileTree + SearchBar)
2. **Git** - Git interface (GitPanel)
3. **Settings** - Vault settings (SettingsPanel)

**Features:**
- Switch between tabs
- Create new project (with input field)
- Close/switch vault
- Show vault name in header

**State:**
- Current tab view
- New project input
- Uses vaultStore for projects

**Used By:** AppShell

---

### Component: `FileTree`

**File:** `/src/components/sidebar/FileTree.tsx`

**Purpose:** Hierarchical display of projects and documents

**Features:**
- Expand/collapse projects
- Click document to open
- Show document metadata (author, status)
- Right-click context menu (delete, rename)
- Drag-and-drop (future feature)

**State:**
- Uses vaultStore for projects, documents, expandedProjects
- Uses editorStore for active document highlighting

---

### Component: `TreeNode`

**File:** `/src/components/sidebar/TreeNode.tsx`

**Purpose:** Individual tree node renderer

**Renders:**
- Folder icons for projects
- File icons for documents
- Expansion toggles
- Selection highlighting

---

### Component: `SearchBar`

**File:** `/src/components/sidebar/SearchBar.tsx`

**Purpose:** Search projects and documents

**Features:**
- Filter documents by name/content
- Debounced search
- Click result to open

---

### Component: `EditorPane`

**File:** `/src/components/editor/EditorPane.tsx` (full component)

**Purpose:** Wrapper for editor with frontmatter editor

**Structure:**
```
EditorPane
├─ FrontMatterBar
└─ CodeMirrorEditor
```

**Shows:** Empty state if no document is open

**Used By:** AppShell (main content area)

---

### Component: `CodeMirrorEditor`

**File:** `/src/components/editor/CodeMirrorEditor.tsx`

**Purpose:** Main markdown editor using CodeMirror 6

**Features:**
- Syntax highlighting for markdown
- CodeMirror extensions:
  - Language support (markdown)
  - Theme (One Dark)
  - Commands (undo, redo, etc.)
- Real-time content updates to editorStore
- Auto-format support

**Key Events:**
- On change → `editorStore.updateContent()`
- On save (Ctrl+S) → `editorStore.saveDocument()`

**Configuration:**
- Dark theme (One Dark)
- Gutter for line numbers
- Horizontal scrollbar

**Lazy Loaded:** Dynamically imported to avoid SSR issues

---

### Component: `FrontMatterBar`

**File:** `/src/components/editor/FrontMatterBar.tsx`

**Purpose:** Editor for document metadata

**Editable Fields:**
- Title
- Author selector (Human/Ai/Both)
- Status (Draft/Review/Final)
- Tags (add/remove)

**Display Fields:**
- Document ID
- Created/Modified timestamps
- Project name

**Updates:** Reconstructs document content with updated frontmatter on save

---

### Component: `MarkdownPreview`

**File:** `/src/components/preview/MarkdownPreview.tsx`

**Purpose:** Rendered markdown preview pane

**Features:**
- Real-time preview of editor content
- Uses `react-markdown` with plugins:
  - Frontmatter stripping
  - GitHub-flavored markdown (tables, strikethrough, etc.)
- Syntax highlighting for code blocks
- Responsive layout

**Updates:** Re-renders when editor content changes

---

### Component: `GitPanel`

**File:** `/src/components/git/GitPanel.tsx` (full component)

**Purpose:** Git interface with three tabs

**Tabs:**
1. **Changes** (ChangesTab) - Stage/unstage files
2. **History** (HistoryTab) - View commits
3. **Remote** (RemoteTab) - Configure remote

**State:** Uses gitStore

---

### Component: `ChangesTab`

**File:** `/src/components/git/ChangesTab.tsx`

**Purpose:** Stage/unstage files interface

**Features:**
- List changed files with status indicators
- Click to stage/unstage individual files
- "Stage All" button
- Commit message input
- Commit button

**File Status Indicators:**
- Color-coded by status
- Icons for new/modified/deleted

**Used By:** GitPanel

---

### Component: `HistoryTab`

**File:** `/src/components/git/HistoryTab.tsx`

**Purpose:** View commit history

**Features:**
- List commits with author, message, date
- Limit display (default 50)
- Formatted timestamps

**Used By:** GitPanel

---

### Component: `RemoteTab`

**File:** `/src/components/git/RemoteTab.tsx`

**Purpose:** Configure remote repository

**Fields:**
- Remote URL
- Remote branch
- Pull on open (checkbox)
- Push on close (checkbox)

**Actions:**
- Update remote config
- Push/pull buttons

**Used By:** GitPanel

---

### Component: `SearchView`

**File:** `/src/components/search/SearchView.tsx`

**Purpose:** Full-text search interface

**Features:**
- Search input with query builder (optional)
- Project filter dropdown
- Results list with snippet preview
- Click result to open document

**Uses:** Vault search command for FTS5 queries

**Used By:** AppShell (when activeView === "search")

---

### Component: `SettingsPanel`

**File:** `/src/components/settings/SettingsPanel.tsx`

**Purpose:** Vault configuration interface

**Settings:**
- Vault name
- MCP enabled (toggle)
- MCP port
- Auto-stage AI writes (toggle)
- SSH key path
- Sync settings (pull/push on open/close)

**Actions:**
- Update settings via setVaultConfig command
- Test remote connection

**Used By:** Sidebar (Settings tab)

---

### Component: `VaultPicker`

**File:** `/src/components/vault/VaultPicker.tsx`

**Purpose:** Vault selection and creation

**Features:**
- Recent vaults list
- Browse for vault directory
- Create new vault (name + path)
- Open vault

**Shows:** When `vaultStore.isOpen === false`

**Used By:** AppShell (conditional render)

---

### Component: `TerminalPanel`

**File:** `/src/components/terminal/TerminalPanel.tsx`

**Purpose:** Terminal emulation

**Features:**
- XTerm.js integration
- PTY communication with backend
- Input/output handling
- Terminal resizing
- Font size controls

**Events:**
- Listens for `terminal:output` Tauri events
- Sends input via `writeTerminal` command

**Lazy Loaded:** Dynamic import to avoid browser-only XTerm issues

---

### Component: `StatusBar`

**File:** `/src/components/StatusBar.tsx`

**Purpose:** Bottom status bar

**Displays:**
- Vault name and path
- Active document path
- Document dirty state
- Git status summary
- File count statistics

---

### Component: `Onboarding`

**File:** `/src/components/Onboarding.tsx`

**Purpose:** Initial setup guide

**Features:**
- Welcome screen
- Create first vault
- Import existing vault
- Quick start tips

---

### Component: `EmptyState`

**File:** `/src/components/shared/EmptyState.tsx`

**Purpose:** Placeholder for empty states

**Shows:**
- "No vault open" → VaultPicker
- "No document selected" → EditorPane
- "No search results" → SearchView

**Props:**
```typescript
{
  title: string;
  description: string;
  icon?: ReactNode;
}
```

---

### Component: `ResizeHandle`

**File:** `/src/components/shared/ResizeHandle.tsx`

**Purpose:** Draggable divider for resizing panes

**Features:**
- Drag to resize
- Double-click to reset to default
- Visual feedback (highlights on hover)
- Configurable direction (horizontal/vertical)

**Used By:**
- AppShell (sidebar/content separator, editor/preview separator, terminal separator)

---

## Type Definitions

**File:** `/src/types/index.ts`

**Core Types:**
```typescript
// Project info from backend
interface ProjectInfo {
  name: string;
  description: string;
  tags: string[];
}

// Document info from backend
interface DocumentInfo {
  title: string;
  path: string;
  author: string;      // "Human" | "Ai" | "Both"
  status: string;      // "Draft" | "Review" | "Final"
  tags: string[];
  created: string;     // ISO8601
  modified: string;    // ISO8601
}

// Search result
interface SearchResultInfo {
  project: string;
  path: string;
  title: string;
  snippet: string;     // HTML with <b> tags
}

// File status in git
interface FileStatus {
  path: string;
  status: string;      // staging status
}

// Commit info
interface CommitInfo {
  oid: string;
  message: string;
  author: string;
  date: string;        // ISO8601
}

// Remote configuration
interface RemoteConfig {
  remote_url: string | null;
  remote_branch: string;
  pull_on_open: boolean;
  push_on_close: boolean;
}

// Vault settings
interface VaultSettings {
  name: string;
  path: string;
  mcp_enabled: boolean;
  mcp_port: number;
  auto_stage_ai_writes: boolean;
  ssh_key_path: string | null;
  remote_url: string | null;
  remote_branch: string;
}

// Vault statistics
interface VaultStatsInfo {
  project_count: number;
  doc_count: number;
  mcp_enabled: boolean;
  mcp_port: number;
  remote_branch: string;
  remote_url: string | null;
}

// MCP server status
interface McpServerStatus {
  running: boolean;
  vault_path: string | null;
  port: number | null;
  binary_found: boolean;
}

// Document frontmatter
interface FrontMatter {
  id: string;
  title: string;
  author: "human" | "ai" | "both";
  tags: string[];
  created: string;
  modified: string;
  project: string;
  status: "draft" | "review" | "final";
  ai_tool?: string;
}
```

---

## Styling

**Technology:** Tailwind CSS v4.0

**Files:**
- `app/globals.css` - Global styles with Tailwind directives
- Component inline styles using `className` prop

**Color Scheme:**
- Dark background: `neutral-950`, `neutral-900`, `neutral-800`
- Text: `neutral-100` to `neutral-500`
- Accents: `blue-400`, `blue-500`
- Borders: `neutral-800`

**Custom Themes:**
- Dark (default)
- Light
- Olive
- Deep Blue

**Theme Application:** `document.documentElement.dataset.theme` attribute

---

## Library Integration

**gray-matter:** Parse YAML frontmatter

```typescript
import matter from "gray-matter";
const { data, content } = matter(raw);
```

**react-markdown:** Render markdown with plugins

```typescript
import ReactMarkdown from "react-markdown";
import remarkFrontmatter from "remark-frontmatter";
import remarkGfm from "remark-gfm";
```

**CodeMirror:** Code editor

```typescript
import { EditorView } from "@codemirror/view";
import { markdown } from "@codemirror/lang-markdown";
```

**xterm:** Terminal emulation

```typescript
import { Terminal } from "@xterm/xterm";
```

---

## Keyboard Shortcuts

**Global:**
- `Ctrl+Shift+F` - Toggle search/editor view
- `Ctrl+T` - Toggle terminal

**Editor (CodeMirror):**
- `Ctrl+S` - Save document
- Standard CodeMirror shortcuts (Ctrl+Z undo, Ctrl+Y redo, etc.)

---

## Performance Optimization

- **Lazy Loading:** Terminal, CodeMirror loaded dynamically via `next/dynamic`
- **Memoization:** Components memoized where appropriate
- **Store Selectors:** Zustand selectors for granular updates
- **Image Optimization:** Next.js Image component (where applicable)

---

## Accessibility

- Semantic HTML elements
- ARIA labels on interactive elements
- Keyboard navigation support
- Color contrast compliance
- Focus management

---

## Next.js Configuration

**Config:** `/next.config.ts`

- Output mode: `export` (static)
- Asset optimization

**Layout:** `/src/app/layout.tsx` - Root layout with providers

**Styling:** PostCSS with Tailwind 4.0

---

## Component Composition Pattern

All components follow this pattern:

```typescript
"use client";

import { useXStore } from "@/stores/xStore";
import { useOtherComponent } from "./OtherComponent";

export function MyComponent() {
  const state = useXStore((s) => s.state);
  const action = useXStore((s) => s.action);

  return (
    <div className="...">
      {/* render */}
    </div>
  );
}
```

**Key Principles:**
- Client-side rendering (`"use client"`)
- Zustand hooks for state
- Tailwind classes for styling
- Event handlers for interactivity

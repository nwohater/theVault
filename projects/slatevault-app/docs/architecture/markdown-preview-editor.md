---
id: 57aa8675-cd9a-4fb9-814a-cb17aa4d3312
title: Markdown Preview and Editor Components Architecture
author: ai
tags:
- architecture
- components
- markdown
- editor
- preview
created: 2026-04-05T15:54:53.991544900Z
modified: 2026-04-05T15:54:53.991544900Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

## Overview

The slateVault app implements a split-pane markdown editor with live preview using React, CodeMirror, and react-markdown. The components are organized hierarchically with the AppShell managing the overall layout.

## Component Hierarchy

```
AppShell
├── Sidebar (FileTree + SearchBar)
├── EditorPane
│   ├── FrontMatterBar
│   └── CodeMirrorEditor
├── MarkdownPreview
├── TerminalPanel (optional)
└── StatusBar
```

## 1. MarkdownPreview Component

**Location:** `/src/components/preview/MarkdownPreview.tsx`

### Rendering Approach
- Uses **react-markdown** (v10.1.0) for markdown parsing and rendering
- Applies **Tailwind CSS typography plugin** for styled rendering
- Uses **Remark plugins** for extended markdown support

### Dependencies
```typescript
import ReactMarkdown from "react-markdown";
import remarkGfm from "remark-gfm";          // GitHub Flavored Markdown
import remarkFrontmatter from "remark-frontmatter";  // YAML frontmatter support
import { useEditorStore } from "@/stores/editorStore";
```

### Key Features
- **Dark theme optimized**: Uses dark backgrounds (bg-neutral-950) with inverted prose styling
- **Prose styling**: Applied via Tailwind's `@tailwindcss/typography` plugin
- **Custom color scheme**:
  - Headings: `prose-headings:text-neutral-100`
  - Body text: `prose-p:text-neutral-300`
  - Links: `prose-a:text-blue-400`
  - Inline code: `prose-code:text-emerald-400` with dark background
  - Code blocks: `prose-pre:bg-neutral-900`
  - Strong text: `prose-strong:text-neutral-200`
- **Max width control**: `max-w-none` removes prose width constraints
- **Scroll behavior**: `overflow-y-auto` for vertical scrolling

### Plugin Configuration
```typescript
<ReactMarkdown
  remarkPlugins={[remarkGfm, remarkFrontmatter]}
>
  {content}
</ReactMarkdown>
```

The plugins handle:
- **remark-gfm**: Tables, strikethrough, task lists, etc.
- **remark-frontmatter**: Skips YAML/TOML frontmatter (doesn't render it in preview)

### State Management
- Gets `content` from editor store: `useEditorStore((s) => s.content)`
- Gets `activePath` to determine if document is open
- Shows `EmptyState` when no document is selected

## 2. EditorPane Component

**Location:** `/src/components/editor/EditorPane.tsx`

### Architecture
- **Layout**: Vertical flex container with FrontMatterBar on top, CodeMirrorEditor below
- **Dynamic import**: CodeMirrorEditor is dynamically imported with `ssr: false` (client-only)
- **Structure**:
  ```tsx
  <div className="flex flex-col h-full">
    <FrontMatterBar />
    <div className="flex-1 min-h-0">
      <CodeMirrorEditor />
    </div>
  </div>
  ```

### FrontMatterBar
**Location:** `/src/components/editor/FrontMatterBar.tsx`

Displays document metadata:
- **Title** (truncated)
- **Dirty indicator** (blue dot when unsaved)
- **Status badge**: draft/review/final with color coding
- **Author label**: human/ai/both
- **Tags**: Shows first 3 tags, displays +N for overflow

Status colors:
- `draft`: yellow-900/yellow-300
- `review`: blue-900/blue-300
- `final`: green-900/green-300

### CodeMirrorEditor
**Location:** `/src/components/editor/CodeMirrorEditor.tsx`

#### Libraries & Configuration
- **@codemirror/view**: DOM rendering
- **@codemirror/state**: Editor state management
- **@codemirror/lang-markdown**: Markdown syntax highlighting
- **@codemirror/theme-one-dark**: Dark theme (matches app aesthetics)
- **@codemirror/commands**: Undo/redo and default keybindings

#### Editor Configuration
```typescript
EditorState.create({
  doc: content,
  extensions: [
    markdown({ base: markdownLanguage, codeLanguages: languages }),
    oneDark,
    history(),
    keymap.of([
      ...defaultKeymap,
      ...historyKeymap,
      {
        key: "Mod-s",
        run: () => {
          saveDocument();
          return true;
        },
      },
    ]),
    EditorView.updateListener.of((update) => {
      if (update.docChanged) {
        updateContent(update.state.doc.toString());
      }
    }),
  ]
})
```

#### Styling
- Custom theme with full height editor
- Font: monospace (SFMono, Menlo, Consolas)
- Font size: 13px
- Gutters: transparent background with dark border
- Synchronized with store on every keystroke

#### Keyboard Shortcuts
- **Ctrl+S / Cmd+S**: Save document
- **Standard CodeMirror defaults**: Undo, redo, navigation
- **History support**: Full undo/redo with history extension

## 3. AppShell Layout Component

**Location:** `/src/components/AppShell.tsx`

### Layout Structure
Three-level layout:

1. **Sidebar** (left panel)
   - Width: configurable 180-500px (default 260px)
   - Contains FileTree and SearchBar
   - Resizable with vertical handle

2. **Main Content Area** (flex-1)
   - **Toolbar**: Search/Editor/Preview toggles + Terminal toggle + Save button
   - **Editor/Preview Pane** (flex-1):
     - **Editor** (flex: editorFlex ratio)
     - **Vertical Resize Handle** (if both visible)
     - **Preview** (flex: previewFlex ratio)
   - **Terminal** (optional, bottom):
     - Resizable height (100-600px default)
     - Custom drag handle implementation
   - **Status Bar** (bottom)

### Flex Ratio Calculation
```typescript
const editorFlex = showEditor ? (showPreview ? previewRatio : 1) : 0;
const previewFlex = showPreview ? (showEditor ? 1 - previewRatio : 1) : 0;
```

Allows:
- Editor only (flex 1)
- Preview only (flex 1)
- Split view with adjustable ratio (default 0.5 = 50/50)
- Minimum 0.2, maximum 0.8 ratio

### View Modes
- **activeView: "editor"** - Shows editor and optional preview
- **activeView: "search"** - Shows full-width SearchView
- **Terminal**: Toggle via Ctrl+T or button

### Keyboard Shortcuts
- **Ctrl+Shift+F**: Toggle search view
- **Ctrl+T**: Toggle terminal

### Resizing Implementation
Two types of resize handles:
1. **Horizontal (sidebar)**: `direction="vertical"` updates sidebarWidth
2. **Vertical (editor/preview)**: Updates previewRatio based on delta
3. **Terminal height**: Custom pointer event handler for row-resize

## 4. Editor Store (State Management)

**Location:** `/src/stores/editorStore.ts`

State variables:
- `activeProject`: Currently open project name
- `activePath`: Path to current document
- `content`: Full markdown content (with frontmatter)
- `frontMatter`: Parsed metadata (FrontMatter interface)
- `isDirty`: Whether unsaved changes exist

### Key Methods
- **openDocument(project, path)**: Load document, auto-save previous if dirty
- **updateContent(content)**: Update on keystroke, parse frontmatter
- **saveDocument()**: Write to backend via commands.writeDocument
- **closeDocument()**: Clear editor state

Auto-save on document switch:
```typescript
openDocument: async (project: string, path: string) => {
  const state = get();
  if (state.isDirty && state.activeProject && state.activePath) {
    await state.saveDocument();  // Auto-save before switching
  }
  // Load new document
}
```

## 5. UI Store (Layout State)

**Location:** `/src/stores/uiStore.ts`

Manages:
- `sidebarWidth`: 180-500px range
- `showEditor`, `showPreview`: Boolean toggles
- `previewRatio`: 0.2-0.8 range (split pane ratio)
- `activeView`: "editor" | "search"
- `showTerminal`, `terminalHeight`: Terminal visibility and height
- `theme`: "dark" | "light" | "olive" | "deepblue"

Theme persistence via localStorage (`sv-theme`).

## 6. Styling System

### Tailwind CSS Setup
- **Version**: 4.0.0 with PostCSS integration
- **Plugins**: `@tailwindcss/typography` for prose styling
- **Colors**: Custom CSS variables for theme support

### Theme System
Uses HTML `data-theme` attribute:
```html
<!-- Dark (default) -->
<html>

<!-- Light theme -->
<html data-theme="light">

<!-- Olive theme -->
<html data-theme="olive">

<!-- Deep Blue theme -->
<html data-theme="deepblue">
```

Each theme overrides CSS variables like `--color-neutral-950`, `--color-neutral-900`, etc.

### Global CSS
**Location:** `/src/app/globals.css`

Includes:
- Tailwind imports with typography plugin
- Theme color variable definitions
- Custom scrollbar styling (webkit)
- CodeMirror overrides:
  - Editor height: 100%
  - Line padding: 16px horizontal
  - Content padding: 8px vertical

## 7. Export and Print Functionality

**Current Status: NOT IMPLEMENTED**

No export/print features found in:
- AppShell toolbar
- EditorPane context menus
- MarkdownPreview
- StatusBar
- Editor store actions

Potential implementation areas:
1. **Toolbar button** in AppShell (next to Save/Terminal buttons)
2. **Preview pane** could add export menu
3. **Browser print (Ctrl+P)**: Would capture whatever pane is visible
4. **File format support**: Could export to HTML, PDF, plain text

The architecture supports this via:
- Direct access to preview HTML via MarkdownPreview component
- Raw content available in editorStore
- Tauri API available for file system operations

## Dependencies Summary

### Core Rendering
- `react-markdown@10.1.0` - Markdown to React components
- `remark-gfm@4.0.1` - GitHub-flavored markdown support
- `remark-frontmatter@5.0.0` - YAML frontmatter parsing

### Editor
- `@codemirror/view@6.40.0` - DOM rendering
- `@codemirror/state@6.6.0` - State management
- `@codemirror/lang-markdown@6.5.0` - Markdown syntax
- `@codemirror/commands@6.10.3` - Keyboard shortcuts
- `@codemirror/theme-one-dark@6.1.3` - Dark theme
- `@codemirror/language-data@6.5.2` - Language detection

### Styling
- `tailwindcss@4.0.0` - Utility CSS framework
- `@tailwindcss/typography@0.5.19` - Prose component styling
- `@tailwindcss/postcss@4.0.0` - PostCSS integration

### UI Framework
- `react@19.1.0`
- `react-dom@19.1.0`
- `next@15.3.0` (App Router)

### State Management
- `zustand@5.0.12` - Lightweight store management

## File Paths Reference

| Component | Path |
|-----------|------|
| MarkdownPreview | `/src/components/preview/MarkdownPreview.tsx` |
| EditorPane | `/src/components/editor/EditorPane.tsx` |
| CodeMirrorEditor | `/src/components/editor/CodeMirrorEditor.tsx` |
| FrontMatterBar | `/src/components/editor/FrontMatterBar.tsx` |
| AppShell | `/src/components/AppShell.tsx` |
| EditorStore | `/src/stores/editorStore.ts` |
| UIStore | `/src/stores/uiStore.ts` |
| Global CSS | `/src/app/globals.css` |
| Package config | `/package.json` |

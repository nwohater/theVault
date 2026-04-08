---
id: 0bc3f218-a104-4cb2-9956-118502965c15
title: Local LLM Integration - Design
author: ai
tags:
- feature
- ai
- llm
- design
created: 2026-04-08T15:44:51.017079700Z
modified: 2026-04-08T15:44:51.017079700Z
project: slatevault-app
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Local LLM Integration

## Status: Design

## Overview
Add OpenAI-compatible LLM integration to slateVault. The app connects to any local or cloud LLM (Ollama, LM Studio, OpenAI, Anthropic) via the standard /v1/chat/completions API. The app orchestrates context assembly — no MCP needed for the LLM.

## Architecture
```
User asks question in AI chat
  -> App reads relevant docs from vault (existing build_context_bundle)
  -> App reads source files from project's source_folder
  -> App sends context + question to LLM API
  -> LLM responds
  -> User can save response as a vault document
```

## Phase 1: Backend Config

### AiConfig (vault.toml)
```toml
[ai]
enabled = true
endpoint_url = "http://localhost:11434/v1"
model = ""
```

### ProjectMeta addition
```
source_folder: Option<String>  # path to project source code
```

### Credentials (~/. slatevault/credentials.toml)
```
ai_api_key: Option<String>
```

## Phase 2: Backend AI Module (ai.rs)

### API call
- POST to {endpoint_url}/chat/completions (OpenAI-compatible)
- GET {endpoint_url}/models for model discovery
- Uses reqwest::blocking (already a dependency)
- Non-streaming first (streaming can be added later)

### Context assembly
- assemble_context(vault, project, user_message, include_source)
- Uses existing get_project_context for pinned files
- Uses existing build_context_bundle for search-based context
- Reads source code files if source_folder is set (50k char limit)
- Skips: node_modules, target, .git, dist, build, __pycache__

### Source code reader
- Recursive walker with extension filter (rs, ts, tsx, js, py, go, etc.)
- 50k character total limit
- Returns files as markdown code blocks with relative paths

## Phase 3: Tauri Commands

### ai_chat(message, project, include_context, include_source, history)
- Assembles context from vault + source
- Builds messages: system (with context) + history + user message
- Calls chat_completion endpoint
- Returns: content, model, token usage

### ai_list_models()
- GET /v1/models from configured endpoint
- Returns list of available model names

### set_project_source_folder(project, source_folder)
### get_project_source_folder(project)

### Extended vault config commands
- ai_enabled, ai_endpoint_url, ai_model added to VaultSettings

## Phase 4: Frontend

### Chat Store (chatStore.ts)
- messages: array of role/content pairs
- isLoading, error, lastModel, lastUsage
- includeContext (default true), includeSource (default false)
- Actions: sendMessage, clearChat, toggles

### AI Chat Panel (components/ai/AiChatPanel.tsx)
- Activity bar icon (sparkles) between Playbooks and Settings
- Header: project name, clear button
- Options: "Include docs" and "Include source" checkboxes
- Scrollable message area with auto-scroll
- Input textarea with Enter-to-send
- Empty state with suggestions

### Message Bubble (components/ai/AiMessageBubble.tsx)
- Assistant messages rendered as markdown
- "Copy" and "Save to vault" buttons on assistant messages
- User messages: plain text, right-aligned

### Settings Panel additions
- AI Assistant section: enabled toggle, endpoint URL, model dropdown (from ai_list_models), API key, Test Connection button

### FileTree additions
- Right-click project -> "Set Source Folder..." (directory picker)
- Badge on projects with source_folder configured

## Key Decisions
1. Non-streaming first (simpler, can add SSE streaming later)
2. Context assembly in Rust (single round-trip, not multiple)
3. API key in credentials.toml (not vault.toml)
4. Stateless backend (frontend owns conversation history)
5. Model discovery via /v1/models (works with Ollama, LM Studio, OpenAI)
6. 50k char source code limit with extension/directory filtering

## Files to Create
- src-tauri/crates/slatevault-core/src/ai.rs
- src/components/ai/AiChatPanel.tsx
- src/components/ai/AiMessageBubble.tsx
- src/stores/chatStore.ts

## Files to Modify
- src-tauri/crates/slatevault-core/src/config.rs (AiConfig, source_folder)
- src-tauri/crates/slatevault-core/src/credentials.rs (ai_api_key)
- src-tauri/crates/slatevault-core/src/lib.rs (add ai module)
- src-tauri/src/commands.rs (ai_chat, ai_list_models, source folder commands)
- src-tauri/src/lib.rs (register commands)
- src/types/index.ts (AiChatMessage, AiChatResponse, VaultSettings)
- src/lib/commands.ts (command wrappers)
- src/components/sidebar/Sidebar.tsx (AI icon + panel)
- src/components/settings/SettingsPanel.tsx (AI section)
- src/components/sidebar/FileTree.tsx (source folder in context menu)

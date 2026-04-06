---
id: b66ff054-0ecc-430d-97cd-f9adc04ba318
title: 'Idea: Plugin System'
author: ai
tags:
- idea
- plugins
- v2
created: 2026-04-06T15:35:46.665695700Z
modified: 2026-04-06T15:35:46.665695700Z
project: vibe2
status: draft
ai_tool: claude-code
---

# Idea: Plugin System

## Concept
Allow users to extend slateVault with custom plugins that can add new document types, sidebar panels, export formats, and editor features.

## Possible Plugin Types
- **Document processors** — custom frontmatter fields, auto-tagging, content validation
- **Export formats** — DOCX, HTML site, Confluence wiki
- **Editor extensions** — custom CodeMirror plugins, snippets, auto-complete
- **Sidebar widgets** — word count dashboard, project analytics, activity feed
- **MCP extensions** — additional tools exposed to AI clients

## Technical Approach
- Plugins as npm packages or local JS files
- Plugin manifest with capabilities declaration
- Sandboxed execution for security
- Event system for hooks (on-save, on-create, on-export)

## Questions to Resolve
- How to handle plugin updates?
- Should plugins have access to the Rust backend or only the frontend?
- What's the security model for community plugins?

## Priority
Low — nice to have for v2, not needed for MVP.

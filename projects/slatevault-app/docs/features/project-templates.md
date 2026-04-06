---
id: 94cc2c2e-3b0b-4ea9-8fc1-846f70826afd
title: Project Templates - Implemented
author: ai
tags:
- feature
- templates
- project-creation
- completed
created: 2026-04-06T15:04:52.080626700Z
modified: 2026-04-06T15:04:52.080626700Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# Project Templates

## Status: Complete

## Overview
When creating a new project, users select a template from a dropdown that pre-scaffolds the folder structure and starter docs. Templates are customizable via `templates.json` in the vault root.

## Default Templates

### Software Development (default)
- `specs/` — Technical specifications
- `features/` — Feature documentation
- `decisions/` — Architecture Decision Records
- `guides/` — How-to guides and tutorials
- `runbooks/` — Operational procedures
- `notes/` — Scratch space

Each folder includes an `_about.md` starter file explaining its purpose.

### Minimal
Empty `docs/` folder, no scaffolding.

## Customization
Users edit `templates.json` in Settings > Project Templates. The JSON format supports:
- Multiple named templates with display labels
- Custom folder lists per template
- Starter files with content per template
- Setting a default template

## Implementation
- `src-tauri/crates/slatevault-core/src/template.rs` — TemplateConfig, ProjectTemplate structs, load/save/apply logic
- `vault.rs` — create_project accepts optional template param, templates.json created on vault create/open
- 3 Tauri commands: `list_templates`, `get_templates_config`, `save_templates_config`
- Sidebar.tsx — template dropdown in project creation form
- SettingsPanel.tsx — JSON editor for templates.json
- MCP server updated to pass None for template (uses default)

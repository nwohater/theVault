---
id: b25b4f9a-4cfd-4558-8e62-945343c6f6f0
title: Themes and Appearance
author: ai
tags:
- themes
- appearance
- ui
- settings
created: 2026-03-31T17:16:32.655579900Z
modified: 2026-03-31T17:16:32.655579900Z
project: getting-started
status: draft
ai_tool: claude-code
---

# Themes and Appearance

slateVault supports four UI themes, all switchable from **Settings → Theme**.

## Available Themes

### Dark (default)
The standard dark theme using neutral grays. Easy on the eyes for long writing sessions.

### Light
A warm off-white theme with dark text. Good for daytime use or if you prefer a document-like feel.

### Olive
A deep forest green palette. Earthy and distinctive — great if you spend a lot of time in the editor.

### Deep Blue
A navy-to-steel blue palette. Feels like writing at the bottom of the ocean.

## How Themes Work

Themes are implemented via CSS custom property overrides on the `<html>` element. The Tailwind neutral color palette is remapped per theme, so all UI elements respond automatically.

Your selected theme is saved to `localStorage` and restored on next launch.

## The Editor

The CodeMirror markdown editor always uses the **One Dark** theme regardless of the UI theme selected. This is intentional — a dark editor is generally easier to write in even on a light UI.

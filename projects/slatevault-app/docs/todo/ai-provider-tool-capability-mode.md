---
id: 0e64dc92-4e69-45d5-8ef6-063afc340732
title: 'TODO: AI Provider Tool Capability Mode'
author: human
tags:
- todo
- ai
- providers
- ux
- tooling
created: 2026-04-15T17:14:20.230492Z
modified: 2026-04-15T17:14:20.230492Z
project: slatevault-app
status: draft
canonical: false
protected: false
---

# TODO: AI Provider Tool Capability Mode

## Problem
The built-in AI assistant currently assumes tool-based document writes are available when the configured endpoint/model may only support text responses. This creates ambiguous behavior for users on providers or accounts that do not expose compatible tool/function calling.

## Goal
Make the assistant explicitly aware of whether the current provider/model supports direct tool writes, and present a clear fallback mode when it does not.

## Proposed Work
- Detect tool capability per configured provider/model more explicitly.
- Surface assistant mode in UI:
  - `Tool write enabled`
  - `Draft-only mode`
- If tool writes are unavailable, avoid implying that the assistant can directly save/update docs.
- Adjust built-in assistant prompts/UX so text-only providers behave intentionally rather than ambiguously.
- Consider provider-specific handling for Claude-account-style setups versus OpenAI-compatible local endpoints.

## Acceptance Criteria
- Users can tell at a glance whether direct tool writes are available.
- Text-only models do not present misleading save/update expectations.
- Tool-capable models continue to use the hardened write path.
- Provider differences are handled explicitly rather than inferred indirectly.

## Notes
This follows recent hardening work around AI doc writes, including path validation, malformed content normalization, and tool-write confirmation behavior.
---
id: d58ce1bf-d16d-4fcc-a67f-1f481b5ad73d
title: Playbooks - AI Workflow Templates
author: ai
tags:
- feature
- playbooks
- ai
- workflows
- prompts
created: 2026-04-07T04:43:29.027455800Z
modified: 2026-04-07T04:43:29.027455800Z
project: slatevault-app
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Playbooks — AI Workflow Templates

Playbooks are pre-defined AI prompt templates stored in the vault root at `playbooks.json`. They give users a curated set of AI-assisted documentation workflows that can be launched against any project.

## How It Works

On startup or project selection, the app reads `playbooks.json` from the vault root. Each playbook is a named prompt template with two interpolation variables:

| Variable | Replaced with |
|---|---|
| `{{project}}` | The active project name |
| `{{folders_section}}` | A listing of the project's folder structure |

When a user selects a playbook, the rendered prompt is injected as the system prompt (or initial user message) for the AI session, scoping the agent's behavior to that workflow.

## Data Format

`playbooks.json` lives at the vault root alongside `vault.toml` and `templates.json`.

```json
{
  "playbooks": [
    {
      "id": "unique-slug",
      "label": "Human-readable name",
      "description": "One-line description shown in the UI",
      "prompt": "Full system prompt with {{project}} and {{folders_section}} placeholders"
    }
  ]
}
```

## Built-in Playbooks

| ID | Label | Purpose |
|---|---|---|
| `document-as-you-go` | Document as You Go | Continuous documentation during development — AI documents decisions, specs, and changes in real time |
| `reverse-engineer` | Reverse Engineer Project | AI analyzes the codebase and creates comprehensive architecture, feature, and guide docs |
| `resume-session` | Resume Session | Brings an AI agent up to speed on what changed since the last session |
| `code-review-docs` | Code Review Documentation | Documents a PR's changes, decisions, and architectural impact |
| `sprint-release-notes` | Sprint / Release Notes | Generates a changelog and release notes from recent commits and doc changes |
| `onboard-team-member` | Onboard Team Member | Creates a comprehensive onboarding guide from existing vault docs |
| `architecture-audit` | Architecture Audit | Reviews all documentation for gaps, staleness, and inconsistencies |

## Prompt Structure Conventions

All built-in playbook prompts follow a consistent structure:

1. **MCP context line** — establishes slateVault access and active project
2. **Startup steps** — what the agent should do first (e.g. `get_project_context`, `list_documents`)
3. **Task instructions** — the specific workflow steps
4. **End-of-session action** — usually writing a summary doc to `changelog/`

## Storage

- `playbooks.json` is stored at the vault root (same level as `vault.toml`)
- It is not project-scoped — playbooks are available across all projects
- The file is user-editable; custom playbooks can be added following the same schema

## Related

- `templates.json` — project folder structure templates (separate from playbooks)
- `vault.toml` — vault-level config
- `reference/mcp-tools.md` — MCP tools that playbook prompts reference

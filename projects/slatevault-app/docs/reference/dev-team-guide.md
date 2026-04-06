---
id: 27cf900d-7aad-4a11-8b49-8602e22fe623
title: Dev Team Usage Guide — Documentation Workflows & Git Strategies
author: ai
tags:
- reference
- guide
- team
- git-strategy
- workflow
created: 2026-04-05T15:50:55.837731300Z
modified: 2026-04-05T15:50:55.837731300Z
project: slatevault-app
status: draft
ai_tool: claude-code
---

# Dev Team Usage Guide

A practical guide for dev managers and teams adopting slateVault as their documentation system.

---

## Why slateVault for Dev Teams

Traditional docs live in wikis (Confluence, Notion) that disconnect from the code. AI-generated docs vanish in chat logs. slateVault solves both: docs live alongside your workflow, are version-controlled with git, and AI tools write directly to the vault via MCP.

---

## Recommended Project Structure

Organize your vault into projects that mirror your team's domains:

```
vault/
├── platform/           # Core platform docs
│   └── docs/
│       ├── architecture/
│       │   ├── system-overview.md
│       │   └── data-flow.md
│       ├── decisions/
│       │   ├── 001-use-postgres.md
│       │   ├── 002-event-driven-arch.md
│       │   └── 003-switch-to-rust.md
│       ├── runbooks/
│       │   ├── deploy-production.md
│       │   └── incident-response.md
│       └── specs/
│           ├── api-v2.md
│           └── auth-flow.md
├── frontend/           # Frontend team docs
│   └── docs/
│       ├── components/
│       ├── decisions/
│       └── style-guide.md
├── onboarding/         # New hire docs
│   └── docs/
│       ├── setup-guide.md
│       ├── codebase-tour.md
│       └── who-owns-what.md
└── incidents/          # Post-mortems
    └── docs/
        ├── 2026-03-15-db-outage.md
        └── 2026-04-01-auth-regression.md
```

### Naming Conventions
- **Projects**: lowercase, kebab-case (`platform`, `frontend`, `data-pipeline`)
- **Folders**: category name (`decisions/`, `specs/`, `runbooks/`)
- **Decision docs**: numbered prefix (`001-use-postgres.md`) for chronological ordering
- **Incident docs**: date prefix (`2026-03-15-db-outage.md`)

---

## Documentation Workflows

### Architecture Decision Records (ADRs)

Use the `decisions/` folder in each project for ADRs. Every significant technical decision gets a doc:

**Template:**
```markdown
# ADR-NNN: Decision Title

## Status
Proposed | Accepted | Deprecated | Superseded by ADR-NNN

## Context
What is the problem or situation that requires a decision?

## Decision
What was decided and why?

## Consequences
What are the trade-offs? What becomes easier? What becomes harder?

## Participants
Who was involved in this decision?
```

**Rule**: No code PR that introduces a new dependency, changes a data model, or alters system architecture ships without a corresponding ADR.

### Specs and Design Docs

Before building a feature, write the spec in slateVault. The PR review workflow makes this powerful:

1. Dev creates a branch: `feature/user-auth-spec`
2. Writes `specs/user-auth.md` with the design
3. Creates a PR for the spec — team reviews the design before code is written
4. After approval, merge the spec, then begin implementation

This creates a paper trail: the spec was reviewed, approved, and is now searchable in the vault.

### Runbooks

Operational runbooks belong in `runbooks/`. These are the docs on-call engineers reach for at 2am:

- Keep them step-by-step with exact commands
- Tag with `runbook` and the service name
- Review quarterly — stale runbooks are dangerous
- AI context files: pin your most critical runbooks so AI tools always have them

### AI-Generated Documentation

When team members use Claude Code or other AI tools with the MCP server connected:

- AI writes docs directly to the vault with `ai_tool` attribution
- Docs are auto-staged for git
- The `author: ai` metadata makes it clear what was AI-generated
- Team reviews AI docs before merging (use the PR workflow)

**Rule**: AI-generated docs must go through PR review before merging to main. No direct commits of AI content to the default branch.

---

## Git Strategies

### Branch Model

**Recommended: Trunk-based with short-lived doc branches**

```
main (protected)
├── docs/add-auth-spec          # spec for new feature
├── docs/update-deploy-runbook  # runbook update
├── docs/adr-004-caching        # new decision record
└── ai/claude-session-042       # AI-generated batch
```

- `main` is the source of truth — always deployable, always reviewed
- Doc branches are prefixed: `docs/` for human-authored, `ai/` for AI-generated batches
- Branches are short-lived: write, review, merge, delete

### Branch Protection Rules

Configure these in GitHub/Azure DevOps:

| Rule | Setting |
|------|---------|
| Require PR for merge to main | Yes |
| Minimum reviewers | 1 (2 for architecture/decisions) |
| Require up-to-date branch | Yes |
| Auto-delete branches after merge | Yes |

### Commit Message Convention

```
docs(project): short description

Longer explanation if needed.

- What changed and why
- Any context for reviewers
```

Examples:
```
docs(platform): add ADR-004 for Redis caching strategy
docs(onboarding): update setup guide for new CI pipeline
docs(incidents): add post-mortem for 2026-04-01 auth regression
ai(platform): Claude-generated API v2 spec draft
```

### PR Review Guidelines for Docs

**For human-authored docs:**
- Is the information accurate and current?
- Does it follow the project's naming conventions?
- Is it in the right folder/project?
- Are tags set correctly?

**For AI-generated docs:**
- Is the content factually correct? (AI can hallucinate)
- Does it match the team's voice and terminology?
- Is the scope appropriate? (AI sometimes over-generates)
- Is the `ai_tool` attribution set?

---

## Team Roles & Permissions

### Doc Owner (Dev Manager / Tech Lead)
- Maintains project structure and naming conventions
- Reviews and merges PRs to main
- Sets branch protection rules
- Conducts quarterly doc audits (delete stale docs, update runbooks)

### Contributors (All Engineers)
- Write docs on feature branches
- Create PRs for review
- Keep AI-generated docs factual
- Tag documents appropriately

### AI Context Curators
- Decide which docs to pin as AI context files per project
- This controls what AI tools "know" about the project
- Review and rotate context files as the project evolves

---

## Vault Configuration for Teams

### vault.toml recommendations

```toml
[vault]
name = "eng-docs"

[sync]
remote_url = "git@github.com:yourorg/eng-docs.git"
remote_branch = "main"
pull_on_open = true    # Always start with latest
push_on_close = false  # Don't auto-push — use PRs

[mcp]
enabled = true
auto_stage_ai_writes = true  # AI docs auto-staged
```

Key settings:
- **pull_on_open = true**: Every team member starts with the latest docs
- **push_on_close = false**: Force the PR workflow — no direct pushes
- **auto_stage_ai_writes = true**: AI content is staged but still requires branch + PR to reach main

### Credentials

Each team member configures their own PAT in Settings > PR Credentials. PATs are stored in `~/.slatevault/credentials.toml` on their machine — never committed to the vault.

---

## Getting Started Checklist

1. Create the vault and connect to your team's git remote
2. Set up project structure (see recommended layout above)
3. Configure branch protection on GitHub/Azure DevOps
4. Have each team member clone and open the vault
5. Pin key docs as AI context files in each project
6. Connect Claude Code via MCP: `claude mcp add -s user slatevault -- slatevault-mcp`
7. Write your first ADR as a team to establish the workflow
8. Set a quarterly calendar reminder to audit and prune docs

---
id: b3be0929-b817-4d5f-8950-111a8fe4513a
title: Getting Started — slateVault User Guide
author: ai
tags:
- guide
- onboarding
- workflow
- getting-started
created: 2026-04-06T20:02:27.552167100Z
modified: 2026-04-06T20:02:27.552167100Z
project: slatevault-app
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Getting Started with slateVault

A practical guide to setting up slateVault and using it effectively as a version-controlled memory system for AI-assisted development.

---

## What is slateVault?

slateVault is not a note-taking app. It's the shared brain between you and your AI coding agents. It stores your project documentation as plain markdown files, version-controls them with git, and exposes 19 MCP tools so AI agents can read, write, search, and reason about your knowledge base — with safety guardrails and human-in-the-loop review.

---

## Step 1: Create Your Vault

When you first open slateVault, you'll see the vault picker. You have three options:

- **Create Vault** — start fresh with a new vault at a folder you choose
- **Open Vault** — open an existing vault folder
- **Clone Repo** — clone a git repo that contains a vault

Choose **Create Vault**, pick a folder, and give it a name (e.g. `eng-docs`).

> **Tip:** Put your vault somewhere that syncs if you want it backed up — OneDrive, Dropbox, or a git remote.

---

## Step 2: Create Your First Project

After creating a vault, you'll see the onboarding screen. Click **Create First Project**.

1. Enter a project name (e.g. `my-app`, `platform`, `api`)
2. Pick a template:
   - **Software Development** — specs, features, decisions, guides, runbooks, notes
   - **Agile Development** — backlog, sprints, retrospectives, ceremonies, epics, definitions
   - **Vibe Coding** — PRD, todo, prompts, context, changelog, bugs, ideas
   - **Minimal** — empty project, no scaffolding
3. Click **Create Project**

The template pre-creates folders with starter `_about.md` files that explain what goes where. These files are also pinned as AI context, so when an agent connects via MCP, it immediately understands your project structure.

---

## Step 3: Connect Your AI Agent (MCP)

This is what makes slateVault different. Click through to the MCP setup screen, or go to **Settings > MCP Server**.

Run this once in your terminal:

```
claude mcp add -s user slatevault -- slatevault-mcp
```

That's it. The MCP server automatically connects to whichever vault is open in the app. When you start a Claude Code session, it has full access to your vault — 19 tools including search, write, context bundling, and safe edits.

### How it works:
1. You open a vault → slateVault writes the path to `~/.slatevault/active-vault`
2. Claude Code starts → the MCP server reads that file and connects to the same vault
3. Your AI agent can now read, write, search, and bundle context from your docs

---

## Step 4: Write Your First Documents

Expand your project in the sidebar and start creating docs:

- Click the **+** button on a project to create a new document
- Documents go inside folders — put specs in `specs/`, decisions in `decisions/`, etc.
- Every doc gets YAML frontmatter automatically: title, author, status, tags, timestamps

### Document Statuses
- **Draft** — work in progress, not yet reviewed
- **Review** — ready for team review
- **Final** — approved and stable

### Special Flags (right-click a doc to toggle)
- **Canonical ★** — marks the doc as source of truth. Canonical docs appear first in agent briefs and context bundles
- **Protected 🔒** — blocks AI agents from overwriting. They must use `propose_doc_update` or `append_to_doc` instead

---

## Step 5: Set Up Git

Switch to the **Git** tab in the sidebar.

1. Go to the **Remote** tab and enter your git remote URL
2. Configure your branch (default: `main`)
3. Enable **Pull on open** so you always start with the latest docs
4. Optionally enable **Push on close** for auto-sync

### Recommended workflow:
- Work on the `main` branch for day-to-day docs
- Create feature branches for larger doc changes or AI-proposed updates
- Use the **PR** tab to push branches and create pull requests on GitHub or Azure DevOps
- Review and merge via your normal code review process

### Credentials:
Go to **Settings > PR Credentials** to add your GitHub or Azure DevOps Personal Access Token. These are stored in `~/.slatevault/credentials.toml` — never committed to your vault.

---

## Daily Workflow

Here's how a typical day looks with slateVault:

### Morning: Catch up
1. Open slateVault → it auto-pulls the latest from your remote
2. Check the **Changes** tab for any uncommitted work from yesterday
3. Click **Agent Brief** in the preview toolbar to generate a project brief — paste it into your AI session to bring the agent up to speed

### During development: Document as you go
1. Writing a new feature? Create a spec in `specs/`
2. Making a technical decision? Add an ADR in `decisions/`
3. Hit a bug? Log it in `bugs/` with repro steps
4. Quick thought? Drop it in `notes/` — you can clean it up later

### With your AI agent:
1. The agent reads your vault via MCP — it knows your project structure
2. Ask it to write docs: *"Write a spec for the auth flow and save it to specs/auth-flow.md"*
3. Ask it to search: *"What do our docs say about the database schema?"*
4. Ask it to propose changes: *"Update the architecture overview to include the new caching layer"* — it uses `propose_doc_update` to write on a branch
5. Generate a brief: *"Generate an agent brief for this project"* — it assembles canonical docs, gaps, and suggested actions

### End of day: Commit and sync
1. Switch to the **Changes** tab
2. Stage your changes (or click Stage All)
3. Write a commit message and commit
4. Push to remote

---

## Key Features to Know

### Agent Brief (preview toolbar)
Click the sparkle icon in the preview toolbar to generate a full project brief and copy it to your clipboard. Paste it into any AI session to instantly bring the agent up to speed. The brief includes:
- Project summary with doc counts
- Key documents (canonical docs, full content)
- Current focus (recently modified)
- Known gaps and warnings
- Constraints and suggested actions

### Copy as Prompt (preview toolbar)
Click the clipboard icon to copy the current document's content (without frontmatter) formatted for pasting into an AI conversation.

### Context Bundling (MCP)
Your agent can call `build_context_bundle` with a query like "authentication" — it searches your vault, ranks results, and returns a concatenated briefing with canonical docs first.

### Propose Doc Update (MCP)
When an agent needs to update a protected or canonical doc, it calls `propose_doc_update`. This creates a branch, writes the change, and returns a diff. You review it in the Git panel and merge when ready.

### PDF Export
- **Single doc:** Click Export PDF in the preview toolbar
- **Full project:** Right-click a project → Export to PDF. Creates sections per folder with proper ordering.

### Backlinks & Related Docs
The preview pane shows:
- **Linked From** — other docs that reference the current doc
- **Related Documents** — docs that share tags

### Secret Detection
The editor warns you if it detects API keys, tokens, or credentials in your document content. Don't commit secrets to the vault.

### Templates
Go to **Settings > Edit Project Templates** to customize the folder structures and starter files for new projects. Templates are stored in `templates.json` at the vault root.

---

## Suggested Project Structure

```
vault/
├── platform/              # Core platform docs
│   └── docs/
│       ├── specs/         # Technical specifications
│       ├── features/      # Feature documentation
│       ├── decisions/     # Architecture Decision Records
│       ├── guides/        # How-to guides
│       ├── runbooks/      # Operational procedures
│       └── notes/         # Quick captures
├── frontend/              # Frontend team docs
│   └── docs/
│       ├── specs/
│       ├── decisions/
│       └── components/
└── onboarding/            # New hire docs
    └── docs/
        ├── setup-guide.md
        └── codebase-tour.md
```

### Naming Conventions
- **Projects:** lowercase, kebab-case (`platform`, `frontend`, `data-pipeline`)
- **Folders:** category name (`decisions/`, `specs/`, `runbooks/`)
- **Decision docs:** numbered prefix (`001-use-postgres.md`)
- **Incident docs:** date prefix (`2026-03-15-db-outage.md`)

---

## Tips

1. **Mark your most important docs as canonical** — this ensures AI agents always read them first
2. **Protect docs you don't want AI to modify** — agents will use `propose_doc_update` instead
3. **Use the Agent Brief before every AI session** — it's the fastest way to bring an agent up to speed
4. **Commit often** — your vault is git-backed, so every commit is a checkpoint you can return to
5. **Use templates** — customize `templates.json` to match your team's workflow
6. **Let AI write first drafts** — then review, refine, and promote to canonical
7. **Check for stale docs periodically** — the MCP `detect_stale_docs` tool flags docs not updated in 30+ days

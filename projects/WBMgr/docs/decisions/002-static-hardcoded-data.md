---
id: 10f94e4e-115e-4c57-b216-09557de1c6c1
title: 002 — Static Hardcoded Data
author: ai
tags:
- decision
- data
- architecture
created: 2026-04-07T15:16:23.942442700Z
modified: 2026-04-07T15:16:23.942442700Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# 002 — Static Hardcoded Data

**Status:** Accepted  
**Inferred from:** `src/data/players.ts`, `src/data/teams.ts`, `src/data/schedule.ts`, `src/data/leaders.ts`

---

## Decision

All league data (players, teams, standings, schedule, historical results, and leaderboards) is hardcoded as TypeScript constants in the `src/data/` directory. There is no database, no file I/O, and no Tauri backend data layer.

---

## Rationale

This is appropriate for the current stage of the project. The app is a UI prototype / demo of a wiffle ball league manager. Using static data allows rapid iteration on the UI without needing to design or maintain a persistence schema. All 40 players with full stats, 3 years of career history, and a complete 12-week schedule were pre-authored.

---

## Consequences

- **Updating stats requires code changes**: To change a player's stats or add a new game result, a developer must edit the data files and rebuild. There is no admin UI.
- **Leaders list can drift from player stats**: `src/data/leaders.ts` is maintained separately from `src/data/players.ts`. If a player's stats change, the leaders file must be updated manually.
- **No multi-league or multi-user support**: `MY_TEAM` is a hardcoded constant in `teams.ts`. There is no concept of user accounts.
- **Easy to version**: All data is in source control, making it trivial to roll back or diff.

---

## Future Path

To make data editable at runtime, the natural next step would be persisting the entire data layer to a file (JSON/SQLite) via Tauri's file system or database plugins, replacing the static imports with a data store.

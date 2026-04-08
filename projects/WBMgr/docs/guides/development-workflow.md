---
id: 751443e3-f0f7-4ccc-a63e-c77f5cd83202
title: Development Workflow
author: ai
tags:
- guide
- development
- workflow
created: 2026-04-07T15:17:20.952573Z
modified: 2026-04-07T15:17:20.952573Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Development Workflow

---

## Project Structure at a Glance

```
src/          ← All React/TypeScript (the app)
src-tauri/    ← Rust/Tauri backend (window shell only)
```

Most day-to-day work happens entirely inside `src/`.

---

## Adding or Changing Static Data

All league data lives in `src/data/`. Changes here are reflected immediately on next HMR reload.

| File | What to change |
|---|---|
| `players.ts` | Player stats, profiles, bios, career history |
| `teams.ts` | Team records (W/L/RF/RA), team names, MY_TEAM |
| `schedule.ts` | Weekly matchups, dates, past results, current week pointer |
| `leaders.ts` | Top-5 leaderboard lists (must be updated manually to match player stats) |

**Important:** `leaders.ts` is not derived from `players.ts` at runtime. If you update a player's stats in `players.ts`, also update the relevant entry in `leaders.ts`.

---

## Updating Player Stats

In `src/data/players.ts`, each `Player` object has optional batting and pitching fields. A player is treated as a pitcher if `ip` (innings pitched) is non-null.

```typescript
// Batter-only example
{ name: "Jake Reimer", team: "Rip City Rockets", ab:82, h:27, d:5, t:2, hr:4, rbi:19, bso:18, bbb:7 }

// Two-way player (batter + pitcher)
{ name: "Sam Kowalski", team: "Rip City Rockets", ab:44, h:14, ..., w:9, l:2, ip:62.1, er:13, pso:68, pbb:14 }
```

Career history is in `PROFILES[name].seasons` as `SeasonStats[]` objects — add a new entry per year.

---

## Changing the Current Week

In `src/data/schedule.ts`:
```typescript
export const CURRENT_WEEK = 9; // change this to advance the season
```

Past results for weeks 1–8 are in `PAST_RESULTS`. To add results for week 9+, append to that array and increment `CURRENT_WEEK`.

---

## Adding a New View

See the dedicated guide: [How to Add a New Feature](adding-a-feature.md).

---

## Styling

All styles are in `src/styles.css` — a single flat file with no preprocessor. There are no CSS modules or styled-components. Class names are descriptive (e.g. `.roster-table`, `.matchup-card`, `.bo-row`).

---

## TypeScript

All shared types are in `src/types/index.ts`. Keep type definitions there rather than co-locating them in component files, so they can be imported from anywhere without circular dependencies.

---

## localStorage in Development

Player roster, batting order, and AI settings are persisted in `localStorage`. During development, changes you make through the UI will persist across page reloads. To reset to defaults, open DevTools → Application → Local Storage → clear the `wbmgr_*` and `rc-*` keys.

---

## No Tests

There are no test files in the project. The TypeScript compiler (`tsc`) is the primary correctness check. Run `npm run build` to catch type errors before shipping.

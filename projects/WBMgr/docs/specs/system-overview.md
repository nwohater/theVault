---
id: a77b0ec8-67a5-42c0-a4c4-2076feb33ee8
title: System Overview
author: ai
tags:
- architecture
- overview
- canonical
created: 2026-04-07T15:14:39.521185800Z
modified: 2026-04-07T15:14:39.521185800Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# WBMgr — System Overview

Wiffle Ball Manager is a desktop application for managing a fictional wiffle ball league. It tracks standings, player statistics, schedules, and allows the user to manage their own team's roster and batting order. An optional Local AI integration supports in-app chat for strategy and insights.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Desktop shell | Tauri 2 (Rust) |
| Frontend framework | React 18 + TypeScript |
| Build tool | Vite 5 (dev port 1420) |
| Styling | Plain CSS (`src/styles.css`) |
| Persistence | `localStorage` (browser-side) |
| AI integration | OpenAI-compatible REST API (local — Ollama, LM Studio, etc.) |

No external UI libraries, no routing library, no state management library beyond React's built-in hooks.

---

## Component Diagram

```
┌──────────────────────────────────────────────────────────────┐
│  Tauri window  (1024×700, min 800×600)                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  App.tsx                                               │  │
│  │  ┌─────────────┐  ┌──────────────────────────────┐    │  │
│  │  │  Sidebar     │  │  <main> — active view        │    │  │
│  │  │  nav buttons │  │                              │    │  │
│  │  │  Dashboard   │  │  Dashboard / MyTeam /        │    │  │
│  │  │  My Team     │  │  Schedule / Standings /      │    │  │
│  │  │  Schedule    │  │  Statistics / Leaders /      │    │  │
│  │  │  Standings   │  │  Settings                    │    │  │
│  │  │  Statistics  │  │                              │    │  │
│  │  │  Leaders     │  └──────────────────────────────┘    │  │
│  │  │  ─────────   │                                      │  │
│  │  │  Settings    │  ┌──────────────────────────────┐    │  │
│  │  └─────────────┘  │  PlayerModal (portal overlay) │    │  │
│  │                    └──────────────────────────────┘    │  │
│  │  PlayerCtx (React Context) — propagates openModal()    │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## Source Tree

```
src/
├── main.tsx                  # React root mount
├── App.tsx                   # Shell: sidebar nav + view switcher + PlayerCtx provider
├── styles.css                # All application styles
│
├── types/
│   └── index.ts              # Shared TypeScript interfaces and union types
│
├── data/                     # Static, hardcoded seed data
│   ├── players.ts            # PLAYERS[] + PROFILES{} (40 players across 8 teams)
│   ├── teams.ts              # MY_TEAM constant, DIVISIONS[] with standings records
│   ├── schedule.ts           # SCHEDULE_WEEKS, WEEK_DATES, CURRENT_WEEK, PAST_RESULTS
│   └── leaders.ts            # Pre-computed top-5 lists for AVG/HR/RBI/ERA/W/K
│
├── utils/
│   └── stats.ts              # calcStats(), avg(), era(), loadStrArr(), loadOrder(), g()
│
├── context/
│   └── PlayerContext.tsx     # PlayerCtx — React context for triggering the player modal
│
├── components/
│   ├── WiffleBall.tsx        # SVG wiffle ball icon (sidebar logo)
│   ├── SortIcon.tsx          # ▲/▼ sort direction indicator
│   ├── PlayerLink.tsx        # Clickable player name that opens PlayerModal
│   ├── PlayerModal.tsx       # Full-screen modal: bio, career batting, career pitching
│   ├── DivisionTable.tsx     # Standings table for one division
│   ├── MatchupCard.tsx       # Single matchup: home vs away, 3-game scores, sim button
│   └── LeaderTable.tsx       # Top-5 leaderboard card
│
└── views/
    ├── Dashboard.tsx         # Welcome screen (static stat cards, not yet wired up)
    ├── MyTeam.tsx            # Roster table, batting order editor, roster move UI
    ├── Schedule.tsx          # Week-by-week schedule browser with sim/play actions
    ├── Standings.tsx         # Two-division standings
    ├── Statistics.tsx        # Full-league batting/pitching stat tables (sortable)
    ├── Leaders.tsx           # League leaders in 6 categories
    └── Settings.tsx          # Local AI URL/model config + in-app AI chat modal

src-tauri/
├── src/
│   ├── main.rs               # Tauri entry point
│   └── lib.rs                # Tauri commands (only `greet` placeholder so far)
├── tauri.conf.json           # App metadata, window config, bundle targets
└── Cargo.toml                # Rust dependencies
```

---

## Data Flow

```
Static data files (data/)
        │
        ▼
Views pull data directly via ES module imports
        │
        ├─► utils/stats.ts  — calculates derived values (AVG, ERA, GB, PCT, run diff)
        │
        └─► localStorage    — persists user mutations (active roster, reserves, batting order,
                               AI URL, AI model name)

PlayerCtx (React Context)
        │
        ├─► Provided by App.tsx (root level)
        └─► Consumed by any component to call setModalPlayer(name)
                │
                └─► triggers PlayerModal overlay in App.tsx
```

---

## Key Abstractions

### `Player` (types/index.ts)
Central data shape. Optional batting fields (`ab`, `h`, `d`, `t`, `hr`, `rbi`, `bso`, `bbb`) and optional pitching fields (`w`, `l`, `ip`, `er`, `pso`, `pbb`) on the same object. A player is a pitcher if `ip != null`.

### `PlayerProfile` (types/index.ts)
Separate from `Player` — holds position, scouting bio, and historical `SeasonStats[]`. Keyed by name in `PROFILES` record.

### `LineupEntry` (types/index.ts)
`{ role: "F" | "P" | "DH"; player: string }` — a slot in the batting order. Role constrains which player pool is available in the editor.

### `GS` (types/index.ts)
`{ h: number; a: number }` — a single game's home/away score. Three `GS` objects represent a 3-game series.

### `calcStats()` (utils/stats.ts)
Derives `pct`, `gb` (games back), and `run_diff` from raw `TeamRecord[]`. Sorts by wins descending.

---

## Rust / Tauri Backend

The backend is essentially a scaffold. `lib.rs` registers a single `greet` command that is not called by the frontend. All application logic runs entirely in the React layer. The Tauri shell provides the native window, OS integration, and bundling.

---

## League Structure

- **8 teams** split into two 4-team divisions (East and West)
- **My team**: Rip City Rockets (hard-coded as `MY_TEAM` in `data/teams.ts`)
- **Season**: 12 weeks, 3 games per matchup, 4 matchups per week (24 games)
- **Current week**: Week 9 (first week where results are not yet pre-baked)
- **Rip City Rockets record** at current week: 18–6

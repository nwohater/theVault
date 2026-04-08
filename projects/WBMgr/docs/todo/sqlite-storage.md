---
id: bcf66bd9-d13a-4b6a-9675-bdc7b40ba1ed
title: 'TODO: SQLite Storage Migration'
author: ai
tags:
- todo
- storage
- sqlite
- architecture
created: 2026-04-07T15:22:52.052945700Z
modified: 2026-04-07T15:22:52.052945700Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# TODO: SQLite Storage Migration

The current app uses static TypeScript data files for everything and `localStorage` for the handful of user-editable fields. To support a real sim engine, player progression, and season history, we need a proper database layer. SQLite via Tauri is the right call — it's embedded, zero-config, and ships inside the app bundle.

---

## Why This Has to Come First

The sim engine and player progression systems are pointless without storage. If sim results aren't saved, every session starts from scratch. If ratings and progression aren't persisted, development arcs don't survive a restart. SQLite is the foundation everything else builds on.

---

## Tauri Integration

Use the official `tauri-plugin-sql` plugin, which wraps SQLite (via `sqlx`) and exposes it to the frontend through Tauri commands. The frontend calls async Tauri commands; the Rust layer executes queries.

```toml
# src-tauri/Cargo.toml
tauri-plugin-sql = { version = "2", features = ["sqlite"] }
```

Alternatively: write custom Rust commands using `rusqlite` directly for more control. The plugin is simpler to start; custom commands are better if we need complex queries or transactions the plugin doesn't support cleanly.

---

## Schema Design

### `seasons`
```sql
CREATE TABLE seasons (
  id        INTEGER PRIMARY KEY,
  year      INTEGER NOT NULL,
  is_active INTEGER NOT NULL DEFAULT 1
);
```

### `teams`
```sql
CREATE TABLE teams (
  id        INTEGER PRIMARY KEY,
  name      TEXT NOT NULL UNIQUE,
  division  TEXT NOT NULL  -- 'East' | 'West'
);
```

### `players`
```sql
CREATE TABLE players (
  id        INTEGER PRIMARY KEY,
  name      TEXT NOT NULL,
  team_id   INTEGER REFERENCES teams(id),
  position  TEXT NOT NULL,
  age       INTEGER NOT NULL,
  -- Ratings (see player-ratings.md)
  r_contact  INTEGER NOT NULL DEFAULT 50,
  r_power    INTEGER NOT NULL DEFAULT 50,
  r_disc     INTEGER NOT NULL DEFAULT 50,  -- plate discipline
  r_speed    INTEGER NOT NULL DEFAULT 50,
  r_stuff    INTEGER NOT NULL DEFAULT 50,  -- pitcher: raw stuff/K ability
  r_control  INTEGER NOT NULL DEFAULT 50,  -- pitcher: walk avoidance
  r_stamina  INTEGER NOT NULL DEFAULT 50,  -- pitcher: IP capacity
  -- Potential (hidden ceiling — see player-progression.md)
  p_contact  INTEGER NOT NULL DEFAULT 50,
  p_power    INTEGER NOT NULL DEFAULT 50,
  p_disc     INTEGER NOT NULL DEFAULT 50,
  p_speed    INTEGER NOT NULL DEFAULT 50,
  p_stuff    INTEGER NOT NULL DEFAULT 50,
  p_control  INTEGER NOT NULL DEFAULT 50,
  p_stamina  INTEGER NOT NULL DEFAULT 50,
  is_pitcher INTEGER NOT NULL DEFAULT 0,
  bio        TEXT
);
```

### `season_stats`
Accumulated counting stats per player per season. The sim engine writes to this after each game.

```sql
CREATE TABLE season_stats (
  id         INTEGER PRIMARY KEY,
  player_id  INTEGER REFERENCES players(id),
  season_id  INTEGER REFERENCES seasons(id),
  -- Batting
  ab INTEGER DEFAULT 0, h INTEGER DEFAULT 0,
  d  INTEGER DEFAULT 0, t INTEGER DEFAULT 0,
  hr INTEGER DEFAULT 0, rbi INTEGER DEFAULT 0,
  bso INTEGER DEFAULT 0, bbb INTEGER DEFAULT 0,
  -- Pitching
  w INTEGER DEFAULT 0, l INTEGER DEFAULT 0,
  ip_outs INTEGER DEFAULT 0,  -- store as outs (×3), convert to IP.f on read
  er INTEGER DEFAULT 0,
  pso INTEGER DEFAULT 0, pbb INTEGER DEFAULT 0,
  UNIQUE(player_id, season_id)
);
```

> Storing IP as integer outs-count (×3) avoids the `.1`, `.2` fractional innings math — just divide by 3 when displaying.

### `schedule`
```sql
CREATE TABLE schedule (
  id         INTEGER PRIMARY KEY,
  season_id  INTEGER REFERENCES seasons(id),
  week       INTEGER NOT NULL,
  home_id    INTEGER REFERENCES teams(id),
  away_id    INTEGER REFERENCES teams(id)
);
```

### `games`
```sql
CREATE TABLE games (
  id          INTEGER PRIMARY KEY,
  matchup_id  INTEGER REFERENCES schedule(id),
  game_num    INTEGER NOT NULL,  -- 1, 2, or 3 in the series
  home_score  INTEGER,
  away_score  INTEGER,
  played_at   TEXT    -- ISO timestamp
);
```

### `roster`
Active vs reserve status, managed per team per season.

```sql
CREATE TABLE roster (
  id        INTEGER PRIMARY KEY,
  player_id INTEGER REFERENCES players(id),
  team_id   INTEGER REFERENCES teams(id),
  season_id INTEGER REFERENCES seasons(id),
  status    TEXT NOT NULL DEFAULT 'active',  -- 'active' | 'reserve'
  bat_order INTEGER  -- null if not in lineup; 1-based position
);
```

---

## Migration Strategy

1. **Seed script**: On first launch (or when no DB exists), run a migration that inserts all current static data (40 players, 8 teams, 12-week schedule, historical results). This preserves the current season's data without requiring re-entry.

2. **Replace static imports**: Views currently import directly from `src/data/*.ts`. These become async Tauri command calls. A React data layer (Context or a lightweight store) fetches and caches the data, exposing it to views.

3. **Retire localStorage**: The `rc-active`, `rc-reserves`, and `rc-batting-order` keys move into the `roster` table. AI settings (`wbmgr_ai_url`, `wbmgr_ai_model`) can stay in localStorage — they're truly device-local config, not game data.

---

## React Data Layer

The frontend will need a data access pattern. Options:

**Option A — Tauri command hooks**: Custom `useTeams()`, `usePlayers()`, `useSchedule()` hooks that call `invoke()` and cache results in component state. Simple but leads to lots of loading states and prop-passing.

**Option B — Context-based store**: A `LeagueCtx` context that loads all season data once on mount and exposes it + mutation functions. Views read from context; mutations call `invoke()` and update context state. This mirrors the current architecture (static data → context) with minimal refactor.

**Option C — TanStack Query**: If async complexity grows, a query library handles caching, invalidation, and loading states cleanly. Probably overkill until the data layer is more complex.

Start with **Option B** — it's the most natural evolution of the current static-data pattern.

---

## Open Questions

- Do we support **multiple active seasons** (e.g., browse last year's stats)? Schema supports it; UI needs season-switcher.
- **Import/export**: Should users be able to back up or share their league as a JSON/CSV export? Easy to add once data is in SQLite.
- **Schema migrations**: Need a versioning strategy (e.g., `pragma user_version`) for when the schema changes across app updates.

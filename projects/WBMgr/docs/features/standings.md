---
id: c6d25468-4118-440d-bd09-78d9cf314fd5
title: 'Feature: Standings'
author: ai
tags:
- feature
- standings
created: 2026-04-07T15:15:06.987370400Z
modified: 2026-04-07T15:15:06.987370400Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: Standings

The Standings view displays current win/loss records for all 8 teams across two divisions, with derived stats calculated at render time.

---

## Key Files

| File | Role |
|---|---|
| `src/views/Standings.tsx` | View — renders two DivisionTable components |
| `src/components/DivisionTable.tsx` | Renders one division's standings table |
| `src/data/teams.ts` | `DIVISIONS[]` with raw `TeamRecord` data |
| `src/utils/stats.ts` | `calcStats()` — derives PCT, GB, run diff |

---

## Data

`DIVISIONS` is a static array of two objects:

```
East Division: Rip City Rockets (18-6), Harbor Hawks (15-9), Eastside Eagles (11-13), Bayside Bandits (5-19)
West Division: Desert Drifters (17-7), Canyon Crushers (14-10), Sunset Sluggers (12-12), Pacific Phantoms (7-17)
```

Each `TeamRecord` has: `name`, `w` (wins), `l` (losses), `rf` (runs for), `ra` (runs against).

---

## Derived Stats (calcStats)

`calcStats(teams)` sorts teams by wins (desc), then derives:

| Column | Formula |
|---|---|
| `pct` | `w / (w + l)` |
| `gb` | `((leaderW - teamW) + (teamL - leaderL)) / 2` |
| `run_diff` | `rf - ra` |

GB is displayed as `"-"` for the division leader.

---

## Display

- Each division is rendered in its own `DivisionTable` block.
- The division leader row receives the CSS class `leader-row`.
- Run differential is colored: green (`pos`) if positive, red (`neg`) if negative.
- Columns: Team · W · L · PCT · GB · RF · RA · DIFF

---
id: b8b5ebb4-5f37-4712-a26c-e3ba5f6f2697
title: 'Feature: League Leaders'
author: ai
tags:
- feature
- leaders
created: 2026-04-07T15:15:29.761747700Z
modified: 2026-04-07T15:15:29.761747700Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: League Leaders

The Leaders view shows pre-computed top-5 leaderboards for 6 statistical categories split into Batting and Pitching sections.

---

## Key Files

| File | Role |
|---|---|
| `src/views/Leaders.tsx` | View — renders 6 LeaderTable components |
| `src/components/LeaderTable.tsx` | Reusable top-5 card component |
| `src/data/leaders.ts` | Static top-5 arrays for each category |

---

## Categories

**Batting**
| Category | Stat column |
|---|---|
| Batting Average | AVG (formatted `.NNN`) |
| Home Runs | HR |
| RBI | RBI |

**Pitching**
| Category | Stat column |
|---|---|
| ERA | ERA (formatted `N.NN`) |
| Wins | W |
| Strikeouts | K |

---

## Data

All six lists are hardcoded in `src/data/leaders.ts` as `BatterRow[]` or `PitcherRow[]` arrays of 5 entries each. They are not derived from the main `PLAYERS[]` array at runtime — they are maintained separately and may diverge if player stats are updated without updating the leaders file.

---

## LeaderTable Component

Generic component that accepts:
- `title: string` — card heading
- `rows: BatterRow[] | PitcherRow[]` — the five entries
- `col: LeaderCol<T>` — descriptor with `{ label, key, fmt }` for the stat column

Renders a small card with rank, player name, team, and stat value.

---

## Notable Leaders (current season snapshot)

- **AVG**: Marcus Webb (Rip City Rockets) — .412
- **HR**: Ray Sutton (Desert Drifters) — 14
- **RBI**: Ray Sutton (Desert Drifters) — 38
- **ERA**: Sam Kowalski (Rip City Rockets) — 1.84
- **W**: Sam Kowalski (Rip City Rockets) — 9
- **K**: Tré Okonkwo (Desert Drifters) — 74

---
id: 8fdbb03e-5832-4ad3-87be-e63a64f02242
title: 'Feature: Statistics'
author: ai
tags:
- feature
- statistics
created: 2026-04-07T15:15:20.186685400Z
modified: 2026-04-07T15:15:20.186685400Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: Statistics

The Statistics view shows full-league batting and pitching stats for all players in a sortable table. All 40 players across all 8 teams are included.

---

## Key Files

| File | Role |
|---|---|
| `src/views/Statistics.tsx` | View — tab switcher, sortable tables |
| `src/data/players.ts` | `PLAYERS[]` — source data |
| `src/components/PlayerLink.tsx` | Clickable name that opens PlayerModal |
| `src/components/SortIcon.tsx` | ▲/▼ sort direction icon |

---

## Tabs

Two tabs: **Batting** and **Pitching**.

- Switching tabs resets the sort key to the most meaningful default:
  - Batting → sort by `avg` descending
  - Pitching → sort by `era` ascending

---

## Batting Table

Filters `PLAYERS` to entries where `ab != null`. Derives `avg = h / ab` on the fly.

Columns: Player · Team · AB · H · 2B · 3B · HR · RBI · SO · BB · **AVG** (highlighted)

Sortable by all columns. `avg` column uses `key: "avg"` on the derived field.

---

## Pitching Table

Filters `PLAYERS` to entries where `ip != null`. Derives `era` using the standard formula:

```
ERA = (ER / IP_decimal) × 9
IP_decimal = whole_innings + (fractional_part / 3)
```

Columns: Player · Team · W · L · IP · ER · SO · BB · **ERA** (highlighted)

---

## Sorting

`sortRows()` is a generic utility inside the view. It handles both string (localeCompare) and numeric comparisons. `asc`/`desc` direction is toggled by clicking the active column header. Clicking a new column sets a fresh direction (desc for most stats, asc for ERA).

---

## Player Links

Player names in both tables are rendered as `<PlayerLink>` components, which consume `PlayerCtx` to open the `PlayerModal` overlay on click.

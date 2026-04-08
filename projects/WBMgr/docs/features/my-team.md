---
id: 147875a3-b9a7-4a38-b359-649d4c57793c
title: 'Feature: My Team'
author: ai
tags:
- feature
- roster
- lineup
created: 2026-04-07T15:14:43.408307300Z
modified: 2026-04-07T15:14:43.408307300Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: My Team

The My Team view is the most interactive screen in the app. It lets the user manage the Rip City Rockets roster and batting order. All changes are persisted to `localStorage`.

---

## Key Files

| File | Role |
|---|---|
| `src/views/MyTeam.tsx` | Entire view — roster table, batting order, swap UI |
| `src/data/players.ts` | `PLAYERS[]`, `RC_ACTIVE_DEFAULT`, `RC_RESERVES_DEFAULT` |
| `src/data/teams.ts` | `MY_TEAM` constant |
| `src/utils/stats.ts` | `avg()`, `era()`, `loadStrArr()`, `loadOrder()` |
| `src/context/PlayerContext.tsx` | `PlayerCtx` — consumed to open player modal |

---

## Roster Table

Displays all active players for the Rip City Rockets in a combined batting + pitching stats table.

**Columns:** Player name · AVG · AB · H · 2B · 3B · HR · RBI · SO · W-L · ERA · K · IP

- Pitching columns are muted (greyed) for non-pitchers.
- Players who appear in the batting order show a `#N` badge next to their name.
- Non-batting pitchers show a `P` role tag.
- All columns are sortable; clicking the active column reverses direction.
- Reserve players appear below active players with `—` stats and a **Reserve** tag.

**localStorage keys:**
- `rc-active` — JSON array of active player names
- `rc-reserves` — JSON array of reserve player names

Default active roster: Marcus Webb, Sam Kowalski, Marcus Tull, Jake Reimer, T.J. Bancroft, Eli Cross  
Default reserves: Nate Greer, Jonah Wise

---

## Roster Moves (Promote / Demote)

Each active player has a **Demote** button; each reserve has a **Promote** button.

Clicking either enters an inline swap UI within the same table row:
1. A `<select>` dropdown appears with the other group's players.
2. Confirm (✓) or Cancel (✗) buttons complete or abort the swap.
3. On confirm, `doSwap()` updates both lists and clears the swapped player's batting order slot.
4. All three localStorage keys are updated atomically.

Only one swap UI can be open at a time (controlled by the `swapping` state object).

---

## Batting Order

Displays a 6-slot batting order. Each slot has a role badge:
- **F** (Fielder) — blue badge
- **P** (Pitcher) — red badge  
- **DH** (Designated Hitter) — yellow badge

**View mode**: Shows player name (clickable → opens PlayerModal) and key stats (AVG/HR/RBI for batters, W-L/ERA/K for pitchers). Empty slots show "— Not set —".

**Edit mode** (activated by "Edit Lineup" button):
- Each slot becomes a `<select>` filtered to the appropriate pool (pitchers only for P slots; all active players for F/DH slots).
- Already-used players are disabled in other slots' dropdowns (shown with ✓).
- Up (▲) / Down (▼) arrow buttons reorder slots.
- **Save Order** persists to `localStorage` key `rc-batting-order`.
- **Cancel** discards draft changes.

Default order: Marcus Webb (F), Jake Reimer (F), Eli Cross (F), T.J. Bancroft (DH), Sam Kowalski (P), empty (F).

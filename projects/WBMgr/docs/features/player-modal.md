---
id: eb2256df-1dc5-4fc7-a83e-1c033a977d96
title: 'Feature: Player Modal'
author: ai
tags:
- feature
- player
- modal
created: 2026-04-07T15:15:42.009847700Z
modified: 2026-04-07T15:15:42.009847700Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: Player Modal

The Player Modal is a full-screen overlay that shows a detailed profile for any player. It is accessible from any view that renders player names — Statistics, Leaders, My Team, and the batting order.

---

## Key Files

| File | Role |
|---|---|
| `src/components/PlayerModal.tsx` | Modal UI — bio, career batting, career pitching |
| `src/components/PlayerLink.tsx` | Wrapper that calls `PlayerCtx` to open modal |
| `src/context/PlayerContext.tsx` | `PlayerCtx` — React Context carrying the `openModal(name)` setter |
| `src/App.tsx` | Holds `modalPlayer` state; renders `PlayerModal` when set |
| `src/data/players.ts` | `PLAYERS[]` (current stats) + `PROFILES{}` (bio + history) |

---

## How It's Triggered

1. `PlayerCtx` is a React Context of type `(name: string) => void`, provided at the root in `App.tsx`.
2. Any component that needs to open the modal calls `useContext(PlayerCtx)` to get `openModal`, then calls `openModal(playerName)`.
3. `App.tsx` stores the name in `modalPlayer` state and renders `<PlayerModal>` when it's non-null.
4. Clicking the overlay backdrop or the ✕ button calls `onClose`, resetting `modalPlayer` to null.

---

## Modal Content

**Header**: Player name, position badge (e.g. `OF`, `SP`, `C`), team name.

**Scout Assessment**: Free-text scouting bio from `PROFILES[name].bio`.

**Career Batting table** (shown if player has `ab != null`):
- Columns: Year · AB · H · 2B · 3B · HR · RBI · SO · BB · **AVG**
- Rows: historical seasons from `PROFILES[name].seasons` + current 2026 season from `PLAYERS[]`.
- Current season row has CSS class `row-current`.

**Career Pitching table** (shown if player has `ip != null`):
- Columns: Year · W · L · IP · ER · SO · BB · **ERA**
- Same season merge logic as batting.

---

## Player Coverage

Profiles (`PROFILES`) exist for all 40 players in the league. Each has:
- 2 historical seasons (2024, 2025)
- A current 2026 season derived live from `PLAYERS[]`

Players with both batting and pitching stats (e.g. Sam Kowalski, Marcus Tull) show both tables.

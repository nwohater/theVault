---
id: bec56125-4ec0-41a3-8f9c-48728e85a847
title: 'Feature: Schedule'
author: ai
tags:
- feature
- schedule
- simulation
created: 2026-04-07T15:14:58.320986400Z
modified: 2026-04-07T15:14:58.320986400Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: Schedule

The Schedule view shows the full 12-week season schedule, allows the user to browse any week, view completed results, and simulate or play current-week matchups.

---

## Key Files

| File | Role |
|---|---|
| `src/views/Schedule.tsx` | View — week navigation, matchup grid, sim state |
| `src/data/schedule.ts` | `SCHEDULE_WEEKS`, `WEEK_DATES`, `CURRENT_WEEK`, `PAST_RESULTS` |
| `src/components/MatchupCard.tsx` | Individual matchup card rendering |
| `src/data/teams.ts` | Team name constants and `MY_TEAM` |

---

## Schedule Structure

- **12 weeks**, each with **4 matchups** (every team plays every week).
- Each matchup is a **best-of-3 series** (3 individual games).
- `SCHEDULE_WEEKS[wIdx][mIdx]` → `[home: string, away: string]`.
- `WEEK_DATES[wIdx]` → human-readable date range string (e.g. `"Mar 9–11"`).
- `CURRENT_WEEK = 9` — first week with no pre-baked results.

---

## Week Navigation

- Left/right arrow buttons step through weeks 1–12.
- Active week shows the date range and a badge: **Current Week** or **Completed**.
- State is local (`viewWeek` useState), not persisted.

---

## Matchup Cards

Each week renders 4 `<MatchupCard>` components in a grid. A card shows:
- Home team vs Away team (user's team name highlighted).
- Three game rows (G1, G2, G3) with home/away scores and win/loss styling.
- Series result line (e.g. "Rip City Rockets wins series 2-1") if results exist.
- **Play** button (current week, user's game only) and/or **Sim** button (current week, any game without results).

---

## Results Logic

| Week state | Results source |
|---|---|
| Past (< current week) | Pre-baked `PAST_RESULTS[wIdx][mIdx]` — static data |
| Current week | `simmed` state (`Record<string, GS[]>`) — random per button press |
| Future week | `null` — no scores shown |

Simulated results are random: `Math.floor(Math.random() * 6) + 1` runs for each team per game. There is no difference between "Play" and "Sim" — both call the same `simSeries()` function. The "Play" button is cosmetically distinct but functionally identical.

Simulated results for the current week are **not persisted** — they reset on page navigation or refresh.

---

## My Team Highlighting

`isMyGame` is true when `home === MY_TEAM || away === MY_TEAM`. The card receives a CSS class `my-game` and the team name is styled with `my-team-name`.

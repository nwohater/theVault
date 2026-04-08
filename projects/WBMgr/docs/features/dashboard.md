---
id: b24e0f94-ed7a-48ba-baa0-cf9ebe2e2161
title: 'Feature: Dashboard'
author: ai
tags:
- feature
- dashboard
created: 2026-04-07T15:16:00.902336800Z
modified: 2026-04-07T15:16:00.902336800Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Feature: Dashboard

The Dashboard is the default landing view shown when the app launches. It is currently a static placeholder.

---

## Key Files

| File | Role |
|---|---|
| `src/views/Dashboard.tsx` | Entire view (26 lines) |

---

## Current State

Renders a welcome heading and four stat cards (Teams, Players, Games Played, Games Scheduled) all showing `0`. The cards are styled but not wired to any real data.

---

## Intended Purpose

The dashboard is meant to provide a quick at-a-glance summary of the league. The stat cards are the obvious extension point — they should eventually pull live values:

| Card | Data source |
|---|---|
| Teams | `DIVISIONS` — count all teams (currently 8) |
| Players | `PLAYERS.length` (currently 40) |
| Games Played | Sum completed games from `PAST_RESULTS` |
| Games Scheduled | Remaining games in `SCHEDULE_WEEKS` |

---

## Gap

This is the least-developed screen in the app. All other views are functional; the Dashboard is purely cosmetic scaffolding.

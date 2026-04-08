---
id: eb03907f-cc4d-4b19-ae0a-6bf6660e6427
title: Product Requirements Document — Wiffle Ball Manager
author: ai
tags:
- prd
- product
- requirements
- canonical
created: 2026-04-08T15:30:43.893302500Z
modified: 2026-04-08T15:30:43.893302500Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Wiffle Ball Manager — Product Requirements Document

## 1. Product Overview

**Wiffle Ball Manager (wbmgr)** is a single-player desktop simulation game for managing a Wiffle Ball league. The player takes the role of general manager for the **Rip City Rockets**, making roster decisions, setting batting lineups, and watching their team compete across a full season. The app blends sports management gameplay with optional local AI integration for scouting insights and strategic advice.

**Platform:** Windows, macOS, Linux (Tauri 2 desktop app)
**Target Audience:** Wiffle ball enthusiasts, sports sim fans, casual strategy gamers

---

## 2. Vision & Goals

### Vision
A lightweight, polished desktop game that captures the fun of managing a wiffle ball team — roster moves, lineup strategy, and season-long competition — without the complexity of full baseball sims.

### Goals
1. **Playable season loop** — Manage a team through a 12-week season with meaningful decisions
2. **Accessible depth** — Simple enough to pick up immediately, deep enough to reward strategy
3. **Offline-first** — Fully functional with no internet; optional local AI adds flavor
4. **Native performance** — Fast startup, small footprint via Tauri

---

## 3. League Structure

| Element | Detail |
|---------|--------|
| Teams | 8 teams across 2 divisions (East & West) |
| Roster size | 6 active + 2 reserves per team |
| Season length | 12 weeks, 4 matchups per week |
| Series format | 3-game series per matchup |
| User team | Rip City Rockets (RC) |

### Teams
- **East Division:** Rip City Rockets, Hillcrest Hitters, Eastside Eagles, Bayview Bombers
- **West Division:** Diamond Dogs, Crescent City Cobras, Sunset Sluggers, Parkside Pythons

---

## 4. Core Features

### 4.1 Dashboard
- **Purpose:** At-a-glance league status
- **Content:** Stat cards showing total teams, players, games played, games remaining
- **Priority:** P1 (implemented — needs live data hookup)

### 4.2 My Team
- **Purpose:** Primary gameplay screen for managing the Rip City Rockets
- **Roster Table:** Sortable by all batting/pitching stats; color-coded pitcher rows; role tags (P, Reserve)
- **Batting Order Editor:** 9-slot lineup with role designations (F/P/DH); drag-to-reorder; save/cancel
- **Roster Moves:** Promote reserves to active / demote active to reserves via swap UI
- **Persistence:** Roster and lineup changes saved to localStorage (future: SQLite)
- **Priority:** P0

### 4.3 Schedule
- **Purpose:** View and simulate the season week-by-week
- **Past Weeks:** Display completed series results with winner highlighting
- **Current Week:** Simulate button generates random 3-game series results
- **Future Weeks:** TBD placeholder until simulated
- **My-team highlighting:** Rip City matchups visually distinct
- **Priority:** P0

### 4.4 Standings
- **Purpose:** Division standings with computed stats
- **Columns:** Team, W, L, PCT, GB, RF, RA, DIFF
- **Behavior:** Leader row highlighted; GB calculated dynamically
- **Priority:** P1

### 4.5 Statistics
- **Purpose:** League-wide stat tables
- **Tabs:** Batting | Pitching
- **Batting columns:** Player, Team, AB, H, 2B, 3B, HR, RBI, SO, BB, AVG
- **Pitching columns:** Player, Team, W, L, IP, ER, SO, BB, ERA
- **Sortable** by any column with visual indicators
- **Priority:** P1

### 4.6 League Leaders
- **Purpose:** Quick view of top performers
- **Batting categories:** AVG, HR, RBI (top 5 each)
- **Pitching categories:** ERA, W, K (top 5 each)
- **Priority:** P2

### 4.7 Player Profiles (Modal)
- **Purpose:** Deep-dive on any player
- **Content:** Name, position, team, scout bio, career stats (multi-season)
- **Access:** Click any player name anywhere in the app
- **Priority:** P1

### 4.8 Settings & Local AI
- **Purpose:** Configure optional local AI assistant
- **AI providers:** Ollama (port 11434), LM Studio (port 1234), Jan, or any OpenAI-compatible endpoint
- **Chat modal:** Full conversation UI with message history
- **Priority:** P2

---

## 5. Planned Features (Roadmap)

### Phase 1 — Simulation Engine (P0)
- Replace random score generation with stats-based simulation
- Player ratings influence at-bat outcomes (contact, power, discipline, speed)
- Pitcher ratings affect opposing batters (stuff, control, stamina)
- Box scores generated per game with individual stat lines
- Season stats update after each simulated series

### Phase 2 — Persistent Storage (P0)
- Migrate from hardcoded data + localStorage to SQLite via Tauri
- Schema: teams, players, seasons, games, at_bats, roster_moves
- Save/load game state
- Support multiple save slots

### Phase 3 — Player Ratings & Progression (P1)
- Hidden ratings system: contact, power, discipline, speed (batting); stuff, control, stamina (pitching)
- Player potential vs. current ability
- Age-based progression: young players develop, veterans decline
- Injuries and fatigue

### Phase 4 — Advanced Management (P1)
- Draft system for new players between seasons
- Free agency and trades
- Multi-season franchise mode
- Season awards (MVP, Cy Young equivalent)

### Phase 5 — AI Integration (P2)
- Local AI scouting reports generated from player data
- Lineup optimization suggestions
- Trade analysis and recommendations
- Natural language game recaps

---

## 6. Data Model

### Core Entities

**Player**
- Identity: name, team
- Batting stats: AB, H, 2B, 3B, HR, RBI, SO, BB
- Pitching stats: W, L, IP, ER, SO, BB
- Profile: position, scout bio, career season history

**Team**
- Identity: name, abbreviation, division
- Record: W, L, RF, RA

**Season / Schedule**
- 12 weeks × 4 matchups × 3 games = 144 total games
- Series results: array of `{ home_score, away_score }` per game

**Lineup**
- 9 batting order slots
- Each slot: player name + role (F / P / DH)

---

## 7. Technical Requirements

### Stack
| Layer | Technology |
|-------|-----------|
| Desktop runtime | Tauri 2 (Rust) |
| Frontend | React 18 + TypeScript |
| Build | Vite 5 |
| Styling | Pure CSS with CSS custom properties (dark theme) |
| Storage (current) | localStorage |
| Storage (planned) | SQLite via Tauri plugin |
| AI (optional) | OpenAI-compatible local endpoints |

### Performance
- App startup: < 2 seconds
- Game simulation: < 500ms per series
- Window: 1024×700 default, 800×600 minimum

### Platform Support
- Windows 10+ (primary)
- macOS 12+ (secondary)
- Linux (best-effort)

---

## 8. UI / UX Requirements

### Theme
- Dark mode only (dark navy backgrounds, blue accent `#4f8ef7`)
- High contrast text (`#e2e8f0` on `#0f1117`)
- Fixed 220px sidebar navigation with icons

### Layout
- Sidebar: 7 nav items (Dashboard, My Team, Schedule, Standings, Statistics, Leaders, Settings)
- Main content area: fluid width, scrollable
- Custom wiffle ball SVG icon in header

### Interaction Patterns
- Click player name → modal opens (global via React Context)
- Sort columns → click header toggles asc/desc
- Roster moves → select from each list, confirm swap
- Lineup edit → toggle edit mode, reorder with arrows, save/cancel
- Simulate → button click, results appear inline

---

## 9. Non-Functional Requirements

- **Offline-first:** No internet required for core gameplay
- **Single-user:** No auth, no accounts, no server
- **Save integrity:** Game state must not corrupt on unexpected close
- **Accessibility:** Keyboard navigable; sufficient color contrast
- **Bundle size:** < 10 MB installer

---

## 10. Success Metrics

| Metric | Target |
|--------|--------|
| Full season playable end-to-end | Yes |
| Stats update after simulation | Accurate to box score |
| Save/load without data loss | 100% reliability |
| App startup time | < 2s |
| User can complete a season in | ~30 minutes |

---

## 11. Out of Scope (v1)

- Multiplayer or online play
- Real player data or licensed content
- Mobile or web versions
- Cloud saves or sync
- Modding or custom league creation

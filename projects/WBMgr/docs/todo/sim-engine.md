---
id: c0125011-567c-438d-9649-f33490ed9fbe
title: 'TODO: Game Simulation Engine'
author: ai
tags:
- todo
- simulation
- engine
- gameplay
created: 2026-04-07T15:23:38.644894400Z
modified: 2026-04-07T15:23:38.644894400Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# TODO: Game Simulation Engine

The current "sim" is a random number generator — `Math.floor(Math.random() * 6) + 1` per team per game. It produces scores with no relationship to player skill, team quality, or matchup dynamics. The sim engine needs to replace this with play-by-play simulation driven by player ratings.

---

## Design Philosophy

Wiffle ball sim should be **fast, fun, and slightly unpredictable** — not a rigid probability calculator. The engine should:

- Reflect player quality meaningfully (good hitters get more hits, good pitchers walk fewer batters)
- Still allow upsets and variance (a weak team can win on any given day)
- Run an entire 3-game series in milliseconds
- Produce realistic-looking stat lines that accumulate into believable season totals

We don't need pitch-by-pitch granularity. **At-bat resolution** (one outcome per plate appearance) is the right level.

---

## Core Loop

```
Game:
  innings = 0..8 (9 innings, 0-indexed)
  for each half-inning:
    outs = 0
    bases = [false, false, false]  // 1B, 2B, 3B occupied
    batter_idx = team.current_batter_slot  // carries across innings
    while outs < 3:
      result = simulate_at_bat(batter, pitcher)
      apply_result(result, bases, outs, score)
      advance_batter_idx()
  if tie after 9: play extra innings (optional)
```

---

## At-Bat Simulation

Each plate appearance produces one of these outcomes:

| Outcome | Abbrev |
|---|---|
| Strikeout | K |
| Walk | BB |
| Single | 1B |
| Double | 2B |
| Triple | 3B |
| Home Run | HR |
| Out in play | OUT |

The probability of each outcome is derived from a **matchup matrix** between batter and pitcher ratings (see `player-ratings.md`). The key insight: combine batter tendencies with pitcher tendencies to get a per-plate-appearance probability table.

### Probability Derivation

Start with **league-average baseline probabilities** (wiffle ball calibrated):

```
K:   22%
BB:  10%
1B:  18%
2B:   7%
3B:   3%
HR:   5%
OUT: 35%
```

Then apply **batter modifiers** based on ratings:
- High `r_contact` → shifts probability from K/OUT toward 1B/2B
- High `r_power` → shifts probability from 1B/2B toward HR/3B
- High `r_disc` → shifts probability from K toward BB

Then apply **pitcher modifiers** based on ratings:
- High `r_stuff` → shifts batter K probability up
- High `r_control` → shifts batter BB probability down
- Combined: pitcher K-ability vs batter contact is the central tension

The final probability table is the sum of baseline + batter delta + pitcher delta, normalized to 100%.

### Pseudocode

```typescript
function atBatProbabilities(batter: PlayerRatings, pitcher: PlayerRatings): AtBatProbs {
  const league = LEAGUE_AVERAGE_PROBS;

  // Each rating is 0-100; 50 = league average. Delta is linear deviation from baseline.
  const contactAdj  = (batter.r_contact  - 50) / 50;  // -1.0 to +1.0
  const powerAdj    = (batter.r_power    - 50) / 50;
  const discAdj     = (batter.r_disc     - 50) / 50;
  const stuffAdj    = (pitcher.r_stuff   - 50) / 50;
  const controlAdj  = (pitcher.r_control - 50) / 50;

  return normalize({
    K:   league.K  - contactAdj * 0.08 + stuffAdj  * 0.10,
    BB:  league.BB + discAdj    * 0.06 - controlAdj * 0.07,
    H1:  league.H1 + contactAdj * 0.07 - stuffAdj  * 0.05,
    H2:  league.H2 + powerAdj   * 0.04,
    H3:  league.H3 + batter.r_speed / 500,  // speed contributes a little
    HR:  league.HR + powerAdj   * 0.04,
    OUT: /* remainder after normalization */,
  });
}
```

The tuning constants (0.08, 0.10, etc.) are the critical numbers to calibrate. They control how much ratings actually matter vs luck. Start conservative — too much ratings influence makes outcomes predictable; too little makes ratings meaningless.

---

## Base Running

After a hit, runners advance based on the hit type and a speed modifier:

| Hit type | Runner on 1B | Runner on 2B | Runner on 3B |
|---|---|---|---|
| Single | → 2B (or 3B if fast) | → 3B (or scores if fast) | Scores |
| Double | Scores | Scores | Scores |
| Triple | Scores | Scores | Scores |
| HR | Scores | Scores | Scores |

"Fast" = batter/runner `r_speed` > 65. Simplification: we're not simulating individual baserunner speed — just using the batter's speed as a proxy for aggressive base running.

---

## Pitcher Stamina

Pitchers have an `r_stamina` rating that determines how many at-bats they can face before tiring. Once a pitcher exceeds their stamina threshold, their `r_stuff` and `r_control` ratings degrade linearly for each additional batter faced.

```
max_batters = 15 + (r_stamina / 100) * 15  // 15–30 batters depending on stamina
if batters_faced > max_batters:
  fatigue_factor = (batters_faced - max_batters) * 0.02
  effective_stuff   = r_stuff   * (1 - fatigue_factor)
  effective_control = r_control * (1 - fatigue_factor)
```

Implication: teams need depth. A pitcher-heavy roster with good stamina has an advantage in close late-inning situations.

---

## Batting Order Impact

The batting order already has role slots (F, P, DH). The sim respects this:
- Pitcher slots (`P`) use pitcher batting stats, which are typically weak
- The order cycles continuously — better hitters at the top get more plate appearances per game
- This makes lineup construction matter: stacking good hitters at the top generates more runs

---

## Stat Accumulation

After each simulated at-bat, increment the relevant `season_stats` counters for both the batter and pitcher. Over a full season this produces realistic stat lines — a player's season AVG will converge toward what their ratings imply, with natural variance.

---

## Where the Engine Lives

**Option A — TypeScript (frontend)**: Easiest to prototype. Runs in the browser/webview. Fine for simming individual games on demand (current week's "Sim" button). Risk: running a full season sim or end-of-season processing could block the UI thread.

**Option B — Rust (Tauri command)**: Better for batch operations — simming a full season, running end-of-year progression, generating an entire schedule of results. Requires re-implementing the logic in Rust, but the performance gain is significant and it keeps heavy computation off the UI thread.

**Recommended approach**: Prototype in TypeScript first. Move hot paths to Rust once the logic is stable and we know what's actually slow.

---

## Output

Each simulated game produces:
- Final score (home/away)
- Per-player stat deltas (to write to `season_stats`)
- Optional: play-by-play log (array of strings like "Webb singles to right, Reimer scores") — useful for a game recap view later

---

## Open Questions

- **Home field advantage**: Should home team get a small probability boost? Traditional baseball gives ~54% win rate to home team. For wiffle ball, could be a setting.
- **Tie games**: Does extra innings make sense? Or just replay until a winner?
- **User "playing" vs simming**: When the user clicks "Play" (their game), do they interact with any decisions (e.g., choose to bunt, pull the pitcher)? Or is "Play" just a sim with their team highlighted? This is a big scope question — interactive play is a different product than passive sim.
- **Pitching rotation**: Currently the user sets a static batting order. Do pitchers rotate by game within a series? Who starts game 2 vs game 1? The batting order has a P slot, implying one pitcher per game — does the user pick their rotation, or does the sim auto-assign by `r_stamina`/`r_stuff`?

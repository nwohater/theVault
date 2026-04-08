---
id: 1eaf1bea-1fa8-4303-b1a1-e0af75c4e474
title: 'TODO: Player Progression System'
author: ai
tags:
- todo
- progression
- aging
- potential
- players
created: 2026-04-07T15:25:35.043341800Z
modified: 2026-04-07T15:25:35.043341800Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# TODO: Player Progression System

Progression turns WBMgr from a static snapshot into a living league. At the end of each season, every player's ratings change — young players develop, veterans peak and then decline, and potential determines ceiling. Done well, this creates genuine emotional attachment to players: watching a prospect blossom into a star, or watching a veteran hold on past his prime.

---

## Core Concepts

### Age
Every player has an age. Age advances by 1 at season end. Age governs which phase of the development curve a player is in and when decline begins.

Age needs to be added to the `players` table (see `sqlite-storage.md`). Initial ages for current players should be assigned during the SQLite migration — current stats can hint at career stage (low AB totals and weak stats suggest young/developing; peak counting stats suggest mid-20s).

### Current Ratings
Where the player is *right now*. These are what the sim engine uses. Ratings can go up or down during progression.

### Potential Ratings
Where the player *could reach at their peak*. Each rating (contact, power, disc, speed, stuff, control, stamina) has both a current value and a potential ceiling. Potential is partially hidden — the player might see a vague tier ("High upside", "Limited ceiling") rather than the exact number.

Potential is set at player creation and **never changes**. It's innate ceiling, not trainable.

---

## Development Curve

Players follow an age-based arc. The wiffle ball league skews younger (shorter seasons, less physical wear), so the curve is:

```
Age 17-21: Prospect phase     — significant growth possible, high variance
Age 22-26: Development phase  — steady improvement toward potential
Age 27-30: Peak phase         — full expression of ratings, minimal change
Age 31-33: Early decline      — slow degradation, especially speed and stamina
Age 34+:   Steep decline      — accelerating decline, possible retirement
```

These are soft boundaries — a 32-year-old with exceptional conditioning might barely decline, while a 28-year-old who was always fragile starts fading early.

---

## End-of-Season Progression Algorithm

Run once per player at season end. The result is a new set of current ratings for next season.

### Step 1 — Determine development budget

Based on age, compute a `growth_budget` (maximum total rating points the player can gain across all their ratings) and a `decline_budget` (minimum total rating points they must lose).

```
age 17-21:  growth_budget = 8-15 pts,  decline_budget = 0
age 22-26:  growth_budget = 4-8 pts,   decline_budget = 0
age 27-30:  growth_budget = 0-3 pts,   decline_budget = 0-2 pts
age 31-33:  growth_budget = 0 pts,     decline_budget = 3-6 pts
age 34+:    growth_budget = 0 pts,     decline_budget = 6-12 pts
```

Add randomness: each budget is drawn from a range, not a fixed value. A 24-year-old might improve by 4 points in a quiet year or 8 in a breakout year.

### Step 2 — Distribute growth across ratings

Growth points are distributed weighted by:
1. **Distance from potential** — ratings far below their potential ceiling grow faster (more room to improve)
2. **Position relevance** — contact and power grow faster for hitters; stuff and control grow faster for pitchers
3. **Random variance** — a small random factor means the same player might develop differently in different simulated seasons

```typescript
function distributeGrowth(player: PlayerRatings, budget: number): RatingDelta {
  const delta: RatingDelta = {};
  const relevantRatings = player.isPitcher
    ? ['r_stuff', 'r_control', 'r_stamina']
    : ['r_contact', 'r_power', 'r_disc', 'r_speed'];

  let remaining = budget;
  for (const rating of relevantRatings) {
    const headroom = player.potential[rating] - player.current[rating];
    if (headroom <= 0 || remaining <= 0) continue;

    // More headroom = faster growth, with random variance
    const gain = Math.min(
      Math.round((headroom / 20) * (1 + Math.random() * 0.5)),
      remaining,
      headroom  // can't exceed potential
    );
    delta[rating] = gain;
    remaining -= gain;
  }
  return delta;
}
```

### Step 3 — Apply decline

Decline primarily hits physical tools first: `r_speed` and `r_stamina` degrade before `r_contact` or `r_stuff`. Mental tools (`r_disc`, `r_control`) decline last — veterans often become more disciplined even as their physical skills fade.

Decline order priority:
1. `r_speed` — first to go
2. `r_stamina` — endurance fades
3. `r_power` — bat speed decreases
4. `r_stuff` — velocity/movement diminishes
5. `r_contact` / `r_control` — experience partially compensates
6. `r_disc` — last to decline

### Step 4 — Apply performance modifier

Season stats provide a performance modifier on top of the age-based curve. A player who significantly outperformed their ratings (had a career year) gets a small permanent boost. A player who dramatically underperformed might lose a point or two.

```
modifier = clamp((season_avg - expected_avg) * 10, -2, +3)
```

This means a breakout season can accelerate development, and a disastrous year can set a young player back. But the effect is capped — one bad year doesn't ruin a high-potential prospect.

---

## Potential Tiers (Player-Facing)

Rather than exposing the exact potential rating numbers (which would feel gamey), show scouting tier labels in the UI:

| Potential range | Label |
|---|---|
| 85–100 | "Star potential" |
| 70–84 | "Above-average ceiling" |
| 55–69 | "Solid starter" |
| 40–54 | "Role player" |
| Below 40 | "Fringe roster" |

Each player's profile can show their potential tier per skill category, giving the manager a sense of who to invest in without showing exact numbers.

---

## Special Events

Beyond the standard curve, add occasional one-time events that make individual seasons memorable:

### Breakout Season
A young player (age 19–24, current ratings significantly below potential) has a ~15% chance of a breakout: growth budget doubles for that season. Announced in a news feed: *"Marcus Webb shows flashes of stardom — scouts taking notice."*

### Decline Event
A player 32+ has a ~20% annual chance of an accelerated decline event: decline budget doubles. *"T.J. Bancroft battles through nagging injuries — signs of wear beginning to show."*

### Injury (future)
Could skip a season or reduce a specific rating temporarily or permanently. Not in v1, but the schema should accommodate it (an `injuries` table with player_id, season_id, rating_affected, games_missed).

---

## Multi-Year Arcs

The system naturally produces compelling stories over 4–6 simulated seasons:

- A 19-year-old prospect with "Star potential" is raw and inconsistent. By age 24 he's a rotation cornerstone.
- A 28-year-old at peak is your most reliable asset — but his decline window is approaching.
- A 35-year-old veteran with legendary stats starts to fade. Do you keep him on the active roster out of loyalty, or make the hard call?

These are the kinds of decisions that make sports management games compelling. Progression is what creates the narrative.

---

## Retirement

When a player's overall rating falls below a threshold (e.g., average across all relevant ratings < 30) and they are age 35+, they become eligible for retirement. In an automated sim, they auto-retire. In a managed league, the player/manager decides.

Retired players should remain in the database with their final stats and career history — accessible from a "Hall of Fame" or "Alumni" view.

---

## Prospect Pipeline (Future)

For a fully featured franchise mode, retiring veterans should be replaced by incoming prospects — young players who enter the league at age 17–19 with low current ratings but varying potential ceilings. This requires a prospect generation system:

- Randomly generate name, age, ratings (low), and potential (randomized from a distribution)
- Each team gets 1–2 new prospects per offseason
- Prospects start on reserve and develop into starters over 2–4 seasons

This is a significant feature — not v1, but the schema should not block it.

---

## Open Questions

- **How many seasons should a league "last"?** If progression is aggressive, star players retire in 8–10 seasons. If slow, the league could run 15–20 seasons before roster turnover becomes dramatic. This is a tuning decision.
- **Player trading**: Progression makes trading interesting — do you trade an aging peak player for a young high-potential prospect? Not in scope yet but the data model (roster table with team_id) supports it.
- **Does the user see next year's ratings before the season starts?** Or do they only see them after the progression runs? Showing "offseason projections" before the season could be a satisfying UI moment.
- **Potential visibility settings**: Should potential ever be fully revealed? Could be a late-game unlock or a "full scouting report" purchase in a future premium feature.

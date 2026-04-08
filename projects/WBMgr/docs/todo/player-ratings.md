---
id: 31d71394-4d4c-4259-a6b9-174ec9fe0d4f
title: 'TODO: Player Ratings System'
author: ai
tags:
- todo
- ratings
- players
- simulation
created: 2026-04-07T15:24:23.904670700Z
modified: 2026-04-07T15:24:23.904670700Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# TODO: Player Ratings System

Player ratings are the bridge between player identity and simulation outcomes. They define *how good* a player is at each skill, independent of accumulated stats. Stats are the result of ratings + variance over a season. Ratings are the cause; stats are the effect.

---

## Rating Scale

**1–100 with 50 as league average.**

This is the standard scouting scale in spirit (though traditional baseball uses 20–80). A 100 is theoretically perfect; a 1 is historically bad. In practice, ratings will cluster around 35–75 for most players, with outliers at the extremes.

| Range | Meaning |
|---|---|
| 80–100 | Elite — top of the league |
| 65–79 | Above average — clear strength |
| 50–64 | Average — solid, not a liability |
| 35–49 | Below average — a weakness |
| 20–34 | Poor — a significant problem |
| 1–19 | Historically bad — rare |

---

## Batting Ratings

### `r_contact` — Contact Ability
Governs the probability of putting the ball in play vs striking out. High contact = rarely strikes out, consistently makes contact. Low contact = high strikeout rate, boom-or-bust hitter.

- **Drives**: K rate, overall batting average floor
- **High contact players**: Tony Delgado, Chris Navarro, Devon Price
- **Low contact players**: Luis Ferreira, D.J. Martinez (inferred from high bSO in current stats)

### `r_power` — Raw Power
Governs the probability that contact results in extra bases (XBH) or a home run. High power = hits the ball hard, generates HR and doubles. Low power = weak contact, grounders and shallow flies.

- **Drives**: HR rate, XBH rate, slugging percentage
- **High power players**: Ray Sutton, Marcus Webb
- **Low power players**: D.J. Martinez, Tyler Dunn

### `r_disc` — Plate Discipline
Governs walk rate and the batter's ability to work counts. High discipline = draws walks, rarely chases, forces pitchers into the zone. Low discipline = expands zone, low BB rate, easy outs on bad pitches.

- **Drives**: BB rate, OBP above AVG
- **High discipline players**: Tony Delgado, Devon Price (inferred from high bBB)
- **Low discipline players**: Hector Leal, Cody Walsh

### `r_speed` — Running Speed
Governs base running aggressiveness, triples probability, and the ability to leg out infield hits. Does not affect batting average directly — it's a "what happens after contact" rating.

- **Drives**: Triple rate, stolen bases (future), range in field (future)
- **High speed players**: Terrell Banks, Riley Holt, Sammy Wren (inferred from triple hits)
- **Low speed players**: Ray Sutton, Darnell King (power types, slow-footed)

---

## Pitching Ratings

### `r_stuff` — Stuff / K Ability
Raw pitch quality — the ability to generate swings and misses. A high stuff pitcher has movement, velocity, and deception that batters struggle to square up. Directly counters batter contact rating.

- **Drives**: Strikeout rate, opposing batting average
- **High stuff pitchers**: Tré Okonkwo, Sam Kowalski, D.J. Pruitt
- **Low stuff pitchers**: Hector Leal, Cody Walsh, Jonah Wise

### `r_control` — Command / Control
Ability to throw strikes and avoid free passes. A high control pitcher rarely falls behind in counts and doesn't beat himself with walks. Directly counters batter discipline rating.

- **Drives**: Walk rate (inverse)
- **High control pitchers**: Sam Kowalski, Felix Reyes, Tré Okonkwo
- **Low control pitchers**: Cody Walsh, Jonah Wise, Nate Greer

### `r_stamina` — Durability / Stamina
How deep into a game a pitcher can go before effectiveness degrades. High stamina pitchers are workhorses who go 6–7 innings. Low stamina pitchers are limited to 3–4 innings of effectiveness and need help.

- **Drives**: Max effective innings per start, IP totals over a season
- **High stamina pitchers**: Sam Kowalski (62.1 IP), Marcus Tull (51.0 IP), Tré Okonkwo (58.2 IP)
- **Low stamina pitchers**: Hector Leal (33.2 IP), Jonah Wise (34.0 IP)

---

## Bootstrapping Ratings from Current Stats

The 40 current players already have full stat lines. We can **reverse-engineer initial ratings** from those stats rather than assigning them by hand.

### Batting rating derivations

```
r_contact ≈ normalize(1 - (bso / ab))       // low K rate = high contact
r_power   ≈ normalize((hr + d + t) / ab)    // XBH rate
r_disc    ≈ normalize(bbb / (ab + bbb))     // walk rate as % of PAs
r_speed   ≈ normalize(t / ab * 10)          // triple rate, scaled up (rare events)
```

These produce a 0–1 value which maps to the 1–100 scale relative to the league. The normalization is across all 40 players: the player with the best rate gets ~85–90, worst gets ~15–20, median gets ~50.

### Pitching rating derivations

```
r_stuff   ≈ normalize(pso / (ip_decimal * 3))  // K per out, or K/9 equivalent
r_control ≈ normalize(1 - (pbb / ip_decimal * 3))  // inverse of BB rate
r_stamina ≈ normalize(ip)                           // raw IP total as proxy
```

This bootstrap only needs to run once as a data migration when moving to SQLite. After that, ratings are their own source of truth and stats are derived from simulation.

---

## Rating Display

**Should ratings be visible to the player?**

Two philosophies:
1. **Transparent ratings** — show all ratings on the player profile. Feels like a video game (FIFA style). Players know exactly what they're working with.
2. **Scouting grades** — show letter grades (A/B/C/D/F) or adjective labels ("Elite", "Average", "Poor") instead of raw numbers. More thematic; hides the math.
3. **Hidden ratings** — only show accumulated stats; ratings are internal engine values. Most realistic but reduces strategic depth.

**Recommendation**: Start with visible numeric ratings on the player modal. This makes testing easier and lets the player understand the system. Can always add a "scouting mode" toggle later that converts to letter grades.

---

## Rating Granularity

The 7 ratings above (4 batting + 3 pitching) cover the core sim inputs. Additional ratings to consider adding later:

| Rating | What it governs |
|---|---|
| `r_clutch` | Performance modifier in high-leverage situations (optional — adds drama) |
| `r_field` | Defensive ability — could affect whether grounders become hits |
| `r_arm` | Outfield/infield throw strength — runner advancement decisions |
| `r_lefty` | Handedness-based matchup modifier — left/right splits |

These are not needed for the initial sim but are natural extensions once the core engine is stable.

---

## Data in SQLite

Ratings are columns on the `players` table (see `sqlite-storage.md`). Every player has all 7 ratings; `r_stuff`, `r_control`, `r_stamina` are only meaningful for pitchers but are stored for all players (a position player's pitching ratings are irrelevant and never used in sim).

---

## Open Questions

- **Who assigns initial ratings?** Auto-bootstrapped from stats (see above) or hand-tuned by the app author? Probably bootstrap first, then hand-tune the outliers.
- **Dual-role players** (e.g., Sam Kowalski, Marcus Tull — both bat and pitch): all 7 ratings apply. The sim uses batting ratings when they're hitting, pitching ratings when they're on the mound. No conflict.
- **Rating visibility per team**: Should a manager know the exact ratings of opposing players? Or just their stats? Could be a setting — "Full scouting" vs "Stats only" mode.

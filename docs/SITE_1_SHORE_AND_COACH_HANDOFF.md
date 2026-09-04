# Site 1 Shore Planner and Fishing Coach Handoff

_Last updated: 2026-09-04_

## Purpose

Add a shore-fishing planning mode for **Site 1** and define the handoff from Go Fish Planner (pre-trip) to the separate Fishing Coach workflow (during trip).

This feature is part of one integrated fishing system:

```text
Go Fish Planner (PRE)
        ↓
Session Brief / chosen opportunity
        ↓
Fishing Coach (DURING)
        ↓
Trip evidence and observations
        ↓
Fishing Skill Progression (POST)
```

Go Fish Planner must remain a planning and opportunity-selection layer. It must not duplicate live coaching or progression logic.

## Privacy / Naming Rule

The shore location covered by this feature must be called **Site 1** in Go Fish Planner outputs, documentation added for this feature, session briefs, and cross-repo handoff data.

Do not expose or infer a more identifying location name in this feature's user-facing output.

## Site 1 Operating Assumption

Site 1 is normally fishable from either sand or a fixed jetty position. Normal tide, surf, or wind variation therefore does **not** create a routine fishing NO-GO state for this planner mode.

Site 1 should use **GO + quality/opportunity rating**, not the exposed-ocean kayak GO/CAUTION/NO-GO engine.

Exception: genuinely hazardous conditions such as severe weather, unsafe jetty exposure, lightning, closures, or other abnormal hazards must still be surfaced prominently. Safety warnings may override normal opportunity language, but poor fishing conditions alone do not create a Site 1 NO-GO.

## Two Independent Scores

Site 1 should keep two concepts separate:

1. **Conditions Score** — how controllable and comfortable the fishing conditions are.
2. **Fishing Opportunity Score** — how favorable the tide, water movement, structure availability, light, season, and other fishing factors are.

Do not let calm weather automatically produce a high fishing-opportunity recommendation.

## Site 1 Inputs

Use available forecast and derived data for:

- sustained wind;
- gusts;
- wind direction/relation to the fishing area;
- outside surf/swell influence;
- tide height;
- tide direction/stage (rising, high/slack, falling, low/slack);
- tidal movement/exchange rate;
- daylight / dawn / dusk;
- season;
- water temperature when available;
- recent rain/runoff or water-quality information when material;
- forecast confidence/completeness.

Surf and swell should be treated as **influence variables**, not copied directly from exposed-ocean launch thresholds, because Site 1 is partially protected.

## Site 1 Derived Fishing Read

The planner should derive, at minimum:

- likely shelf/trough/deeper-channel availability from tide height and stage;
- relative current strength (weak / moderate / strong);
- whether water movement is increasing, decreasing, or near a turn;
- likely bottom, middle-column, or upper-column opportunity;
- whether baitfish/search-lure activity is favored;
- whether the sand position or jetty fallback is the more practical fishing stance;
- confidence in the read.

Known Site 1 structure patterns are hypotheses to inform the planner, not guarantees. Current observations during Fishing Coach override the pre-trip hypothesis.

## Site 1 Recommendation Output

For each evaluated window, return a compact result similar to:

```text
SITE 1 — GO · High Opportunity
Conditions: 82/100
Fishing Opportunity: 88/100
Best window: 5:15–7:15 PM
Tide: Rising · moderate movement
Wind: Good for line control
Surf influence: Low
Structure read: Shelf filling / trough active
Likely productive zone: Middle + lower water column
Primary approach class: Natural bait / bottom-to-mid search
Secondary approach class: Casting lure search
Confidence: High
```

The planner may recommend an **approach class** (for example bottom natural bait, suspended bait, lure search, baitfish search), but should not perform detailed live tackle selection, exact lure cadence, cast-by-cast adjustments, or progression coaching. Those belong to Fishing Coach.

## Hourly / Day Ranking Behavior

For Site 1:

- rank the best fishing windows by Fishing Opportunity Score, then use Conditions Score and confidence as secondary differentiators;
- identify the best hourly block when sufficient hourly data exists;
- preserve tide turns and changes in current strength rather than averaging them away;
- poor opportunity remains `GO — Low Opportunity` rather than `NO-GO`, unless an abnormal safety hazard exists.

Site 1 should be comparable against other trip choices as an **opportunity**, but its score must not be mixed into exposed-ocean kayak safety rankings.

## Go Fish → Fishing Coach Session Brief Contract

Whenever Go Fish Planner is used to select or recommend a fishing session, it should be able to produce a **Session Brief** for downstream use.

Minimum fields:

```js
{
  planner: "GoFish",
  generatedAt,
  fishingDate,
  siteId: "site-1",
  mode: "shore",
  chosenWindow: {
    start,
    end
  },
  conditionsScore,
  fishingOpportunityScore,
  opportunityBand,
  tide: {
    stage,
    heightRange,
    movementStrength,
    nextTurn
  },
  wind: {
    sustainedRange,
    gustRange,
    direction,
    lineControlEffect
  },
  surfInfluence,
  lightWindow,
  structureHypothesis,
  likelyWaterColumn,
  recommendedApproachClasses,
  expectedSpeciesClasses,
  hazards,
  confidence,
  sourceTimestamps
}
```

The brief should preserve the reasoning-relevant outputs that caused Go Fish Planner to recommend the window. Fishing Coach should not require the user to remember or restate them.

## Persistence / Retrieval Rule

When practical, save or expose the latest chosen Session Brief in a machine-readable or plainly retrievable form associated with the planner run.

Fishing Coach must use this order:

1. Retrieve the latest relevant Go Fish Session Brief for the same date/site/window when available.
2. If no stored brief exists but Go Fish logic/data is accessible, reproduce the same planning calculation for the requested current session.
3. If neither is available, independently obtain current forecast/tide information and reconstruct a clearly labeled provisional pre-trip read.

The user should not be asked to manually repeat what Go Fish Planner already determined when the information can be retrieved or recalculated.

## Responsibility Boundary

### Go Fish Planner owns

- comparing future days;
- ranking windows;
- Site 1 Conditions Score;
- Site 1 Fishing Opportunity Score;
- forecast-based tide/current/structure hypothesis;
- broad approach class;
- Session Brief generation.

### Fishing Coach owns

- validating the planner hypothesis against same-day/current observations;
- trip-specific packing based on actual owned gear;
- bait/rig/lure selection from available inventory;
- exact target depth and placement;
- cast/soak/drift/retrieve instructions;
- live adjustments from bites, catches, bait loss, photos, current, wind, or visible activity;
- end-of-session learning summary;
- routing evidence into Skill Progression and other fishing-repo records.

### Fishing Skill Progression owns

- interpreting accumulated evidence;
- advancing or holding skill stages;
- choosing next learning priorities;
- feeding those priorities back into future Fishing Coach sessions.

## Integration Principle

Go Fish Planner, Fishing Coach, and Fishing Skill Progression are not standalone competing tools. They form a continuous pre → during → post learning loop.

The planner should provide enough structured context that Fishing Coach can begin with: **“This is why we chose this window; now verify it and fish it.”**

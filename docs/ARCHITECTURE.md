# Go Fish Planner – Architecture

**Status:** Canonical architecture reference  
**Purpose:** Define how Go Fish Planner is organized so future features preserve existing behavior, avoid duplicated calculations, and remain compatible with Go Fish Live.

## 1. Architectural Principles

- Preserve existing functionality unless a requested change explicitly replaces or removes it.
- Increment the version automatically for every planned release.
- Never modify scoring logic unless explicitly requested.
- Treat Go Fish Live as the primary deployment target.
- Never place new calculations directly inside the UI.
- Keep one source of truth for each score, threshold, recommendation, and safety decision.
- User-facing modules consume structured outputs from logic modules.
- Mission Control must be an integration layer, not a second decision engine.
- Protected-water logic must remain separate from exposed-ocean launch logic.
- A hard NO-GO must override rankings, Coach Mode, species advice, and convenience features.

## 2. High-Level System Flow

```text
Forecast Data Sources
        ↓
Data Normalization Layer
        ↓
Derived Conditions Layer
        ↓
Safety and Fishing Logic Modules
        ↓
Structured Recommendation Objects
        ↓
User Interface Modules
        ↓
Go Fish Live
```

The UI must not recalculate safety scores, thresholds, species probabilities, battery estimates, drift estimates, or return limits.

## 3. Core Modules

### 3.1 Forecast Data Layer

Retrieves raw weather, marine, and tide information using the selected date, time period, launch-area coordinates, provider endpoints, and tide-station identifiers.

Expected data includes sustained wind, gusts, wind direction, wave height, wave period, swell direction, estimated surf, air and water temperature, tide height and trend, sunrise, sunset, and forecast timestamps.

```js
{
  siteId,
  timestamp,
  windSpeed,
  windGust,
  windDirection,
  waveHeight,
  wavePeriod,
  swellDirection,
  surfHeight,
  tideHeight,
  tideTrend,
  airTemperature,
  waterTemperature,
  sunrise,
  sunset,
  sourceStatus
}
```

Rules:

- Convert provider-specific fields into internal names.
- Represent missing data explicitly.
- UI components must not call forecast APIs independently.
- Forecast retrieval failures must be visible to the user.

### 3.2 Data Normalization Layer

Responsibilities:

- Convert timestamps to the planner’s local time zone.
- Standardize wind, wave, and surf units.
- Align hourly weather, marine, and tide observations.
- Mark stale, missing, or estimated values.
- Preserve source attribution internally.

### 3.3 Derived Conditions Layer

Reusable derived conditions include:

- Offshore, onshore, or cross-shore wind
- Morning versus afternoon wind trend
- Tide stage and tidal exchange
- Wind-dominant versus current-dominant drift
- Estimated launch and return difficulty
- Daylight remaining
- Forecast completeness
- Model agreement

```js
{
  windRelationToShore,
  windTrend,
  tideStage,
  tidalExchange,
  daylightRemaining,
  launchExposure,
  returnExposure,
  forecastCompleteness,
  modelAgreement
}
```

These values should be calculated once and reused.

## 4. Safety Modules

### 4.1 Ocean Launch Safety Engine

Determines whether an exposed-ocean launch is GO, CAUTION, or NO-GO.

```js
{
  siteId,
  safetyStatus,
  safetyScore,
  hardNoGo,
  reasons,
  penalties,
  bestUsableWindow
}
```

Rules:

- Never include protected-water sites in ocean rankings.
- A hard NO-GO cannot be overridden by a high numeric score.
- Existing scoring logic may not change without explicit approval.
- Safety status is calculated before fishing quality.

### 4.2 Protected-Water Safety Engine

Evaluates sheltered fallback locations independently.

```js
{
  siteId,
  protectedStatus,
  protectedScore,
  warnings,
  bestWindow
}
```

Protected-water scores do not compete with ocean scores. Protected does not mean risk-free.

### 4.3 Hard NO-GO Engine

Applies absolute safety overrides such as excessive wind, gusts, surf, waves, or critical forecast-data failure.

```js
{
  hardNoGo: true,
  ruleId,
  reason,
  userMessage
}
```

Every downstream module must respect this result.

### 4.4 Return Safety / Ocean Exit Module

Determines whether returning later may be substantially harder than launching.

```js
{
  returnScore,
  latestRecommendedReturn,
  worseningRisk,
  primaryReason,
  hourlyReturnStatus
}
```

Feeds Coach Mode, Trip Timeline, Hazard Intelligence, Battery Planner, and Mission Control.

### 4.5 Confidence Module

Expresses recommendation reliability based on forecast completeness, horizon, changing conditions, and missing inputs.

```js
{
  confidenceScore,
  confidenceBand,
  reasons,
  missingInputs
}
```

Confidence is not safety. A high-confidence NO-GO remains a NO-GO, while a low-confidence GO should become more conservative.

### 4.6 Hazard Intelligence Module

Aggregates operational hazards including ebb current, tidal exchange, offshore wind, afternoon wind increase, fog, low daylight, vessel traffic, structure hazards, advisories, closures, and data gaps.

```js
{
  hazards: [
    {
      type,
      severity,
      areaId,
      startTime,
      endTime,
      message,
      source
    }
  ]
}
```

## 5. Fishing Intelligence Modules

### 5.1 Species Recommendation Engine

Ranks realistic target species using habitat, water temperature, tide, time, season, structure, and safety context.

Species logic never overrides safety.

### 5.2 Structure and Habitat Module

Returns preferred structure, secondary structure, depth range, search pattern, and move rule.

### 5.3 Gear Recommendation Module

Recommends rigs, bait or lures, leader, weight, hook, snap, position-holding gear, and required safety gear. It should prefer equipment the user already owns and consume existing module outputs.

### 5.4 Position Holding Module

Recommends pedaling, Bixpy positioning, controlled drifting, one stern drift sock, anchoring on open sand, avoiding anchoring, returning to protected water, or ending the trip.

```js
{
  method,
  anchorRecommendation,
  driftSockRecommendation,
  reason,
  prohibitedActions,
  quickReleaseRequired
}
```

Rules:

- Anchoring is never a method for remaining offshore in worsening conditions.
- Do not recommend anchoring in channels, surf influence, heavy traffic, kelp, reef, or snag-prone structure.
- One drift sock is preferred over two for normal kayak control.
- Two drift socks must never substitute for returning.
- Area profiles must be stored separately from UI wording.

### 5.5 Drift Prediction Module

Estimates drift direction, speed, track, time across the target zone, and confidence from wind, current, kayak orientation, drift-sock status, and the area profile.

This module should not be built until its assumptions and limitations are documented.

## 6. Trip Execution Modules

### 6.1 Automatic Trip Timeline

Sequences existing recommendations into a chronological plan. It must not invent new scores.

### 6.2 Bixpy Battery Planner

Estimates consumption and preserves a conservative return reserve. It must not present range as guaranteed.

### 6.3 Live Coaching Module

Consumes existing timeline, safety, hazard, and forecast outputs to advise whether to continue, move, enter protected water, begin returning, or end the trip.

### 6.4 Virtual Fishing Day

Provides an hour-by-hour rehearsal using existing modules. It is a presentation and education layer only.

## 7. Explanation and Transparency Modules

### 7.1 Score Explanation / “Why Not?”

Reads adjustments from the scoring engine rather than recalculating scores.

### 7.2 Learning / Trip History Module

May personalize fishing recommendations based on past trips, but must never weaken fixed safety thresholds without explicit approval.

## 8. User Interface Modules

### 8.1 Seven-Day and 16-Day Outlook

Summarizes existing daily calculations and does not imply precision beyond the source forecast.

### 8.2 Launch Decision

Shows exposed-ocean safety rankings. Protected-water areas cannot appear in this ranking.

### 8.3 Protected-Water Backup

Shows the best sheltered alternative using protected-water safety, species, and position-holding outputs.

### 8.4 Species Recommendations

Shows target species, structure, timing, and technique independently from launch safety. It cannot override launch safety.

### 8.5 Coach Mode

Builds a step-by-step plan from existing outputs.

Rules:

- If all exposed sites are below 60 or hard NO-GO, refuse an exposed-ocean plan.
- Recommend the best protected-water fallback.
- Do not calculate independent safety, species, anchor, drift, or battery results.

### 8.6 Mission Control

Mission Control should be built only after the upstream modules are validated. It displays and prioritizes structured outputs; it does not introduce calculations, thresholds, or safety decisions.

## 9. Area Profile Architecture

```js
{
  id,
  name,
  category,
  coordinates,
  shorelineOrientation,
  exposure,
  launchType,
  protectedWater,
  trafficHazards,
  surfHazards,
  kelpRisk,
  reefRisk,
  anchorZones,
  prohibitedAnchorZones,
  habitatTypes,
  tideStation,
  notes
}
```

Area profiles supply facts and constraints. Logic modules interpret those fields so location-specific rules are not buried in UI text.

## 10. Shared Output Contract

```js
{
  status,
  score,
  recommendation,
  reason,
  warnings,
  confidence,
  sourceTimestamp
}
```

Not every module requires every field, but naming should remain consistent.

## 11. Error Handling

The planner must distinguish among safe, caution, unsafe, missing data, stale data, API failure, and unsupported forecast horizon. Missing marine data must never silently become a positive safety assumption.

## 12. Testing Requirements

Before every release verify that:

- Existing tabs still work.
- Live data still loads.
- Protected-water locations remain excluded from ocean rankings.
- Hard NO-GO behavior still works.
- Coach Mode refuses unsafe exposed-ocean plans.
- Protected-water fallback still appears.
- Version numbers are updated everywhere.
- Go Fish Live compatibility is preserved.
- No scoring logic changed unless requested.
- New UI components consume existing module outputs.
- Mobile layout works on iPhone.
- Forecast failures are visible.
- Anchor guidance remains area-specific and safety-first.

## 13. Version and Release Workflow

1. Start from the latest authoritative source.
2. Preserve the previous version.
3. Make only requested changes.
4. Increment the version.
5. Update the changelog.
6. Document new design decisions.
7. Run the testing checklist.
8. Produce the standalone HTML.
9. Deploy or prepare for Go Fish Live.
10. Confirm the displayed version matches the release.

## 14. Recommended Future File Structure

```text
gofishplanner/
├── index.html
├── docs/
│   ├── PROJECT_SETUP.md
│   ├── DESIGN_DECISIONS.md
│   ├── ARCHITECTURE.md
│   ├── ROADMAP.md
│   ├── CHANGELOG.md
│   └── TESTING.md
├── src/
│   ├── data/
│   │   ├── forecast.js
│   │   ├── normalization.js
│   │   └── areaProfiles.js
│   ├── logic/
│   │   ├── oceanSafety.js
│   │   ├── protectedSafety.js
│   │   ├── hardNoGo.js
│   │   ├── returnSafety.js
│   │   ├── confidence.js
│   │   ├── hazards.js
│   │   ├── species.js
│   │   ├── positionHolding.js
│   │   ├── drift.js
│   │   └── battery.js
│   └── ui/
│       ├── launchDecision.js
│       ├── protectedWater.js
│       ├── speciesPage.js
│       ├── coachMode.js
│       ├── outlook.js
│       └── missionControl.js
└── archive/
    └── versioned releases
```

## 15. Current Development Order

### Phase 1 – Safety Foundation

1. Position Holding Strategy
2. Return Safety / Ocean Exit Score
3. Confidence Meter
4. Hazard Intelligence

### Phase 2 – Trip Execution

1. Automatic Trip Timeline
2. Gear Recommendations
3. Bixpy Battery Planner
4. Drift Prediction

### Phase 3 – Fishing Intelligence

1. Score Explanation
2. Species Probability
3. Seasonal Intelligence
4. Trip Learning

### Phase 4 – Integration

1. Mission Control
2. Live Coaching
3. Virtual Fishing Day

Mission Control remains last because it depends on structured outputs from all prior modules.

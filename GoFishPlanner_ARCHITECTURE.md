# Go Fish Planner – Architecture

**Status:** Canonical architecture reference  
**Purpose:** Define how Go Fish Planner is organized so future features preserve existing behavior, avoid duplicated calculations, and remain compatible with Go Fish Live.

---

# 1. Architectural Principles

All development must follow these rules:

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

---

# 2. High-Level System Flow

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

---

# 3. Core Modules

## 3.1 Forecast Data Layer

### Purpose

Retrieve raw forecast and marine data used throughout the planner.

### Inputs

- Selected date
- Selected time period
- Launch-area coordinates
- Forecast-provider endpoints
- Tide-station identifiers

### Expected data

- Sustained wind
- Wind gusts
- Wind direction
- Wave height
- Wave period
- Swell direction
- Estimated surf
- Air temperature
- Water temperature when available
- Tide height
- Tide direction
- Tide timing
- Sunrise and sunset
- Forecast timestamps

### Outputs

A normalized forecast object for each area and hour.

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

### Rules

- Provider-specific field names must be converted into internal field names.
- Missing data must be represented explicitly.
- UI components must not call forecast APIs independently.
- Forecast retrieval failures must be visible to the user.

---

## 3.2 Data Normalization Layer

### Purpose

Convert all source data into consistent units, time zones, labels, and formats.

### Responsibilities

- Convert timestamps to the planner’s local time zone.
- Standardize wind speed units.
- Standardize wave and surf units.
- Align hourly weather, marine, and tide observations.
- Mark stale, missing, or estimated values.
- Preserve source attribution internally.

### Outputs

A clean hourly condition record that downstream modules can trust.

---

## 3.3 Derived Conditions Layer

### Purpose

Calculate reusable conditions from normalized data.

### Examples

- Offshore, onshore, or cross-shore wind
- Morning versus afternoon wind trend
- Tide rising, falling, near-high, or near-low
- Large tidal exchange
- Wind-dominant versus current-dominant drift
- Estimated launch difficulty
- Estimated return difficulty
- Daylight remaining
- Forecast completeness
- Model agreement or disagreement

### Outputs

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

These values may be used by several modules but should only be calculated once.

---

# 4. Safety Modules

## 4.1 Ocean Launch Safety Engine

### Purpose

Determine whether an exposed-ocean launch is GO, CAUTION, or NO-GO.

### Inputs

- Normalized forecast data
- Derived conditions
- Area exposure profile
- Hobie i12S safety assumptions
- Existing safety thresholds

### Outputs

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

### Rules

- Never include protected-water sites in ocean rankings.
- A hard NO-GO cannot be overridden by a relatively high numeric score.
- Existing scoring logic may not be changed without explicit approval.
- Safety status must be calculated before fishing quality.

---

## 4.2 Protected-Water Safety Engine

### Purpose

Evaluate sheltered fallback locations independently.

### Inputs

- Normalized forecast data
- Protected-water area profile
- Channel exposure
- Boat-traffic considerations
- Tide and current behavior

### Outputs

```js
{
  siteId,
  protectedStatus,
  protectedScore,
  warnings,
  bestWindow
}
```

### Rules

- Protected-water scores cannot compete with ocean scores.
- These locations become the primary fallback when all exposed sites are unusable.
- Protected does not mean risk-free; traffic, current, visibility, and launch access still matter.

---

## 4.3 Hard NO-GO Engine

### Purpose

Apply absolute safety overrides.

### Examples

- Wind beyond the approved operating limit
- Gusts beyond the approved operating limit
- Surf beyond the approved launch limit
- Wave conditions beyond the approved limit
- Forecast failure combined with exposed conditions
- Other explicitly defined safety overrides

### Outputs

```js
{
  hardNoGo: true,
  ruleId,
  reason,
  userMessage
}
```

### Rule

Every downstream module must respect this result.

---

## 4.4 Return Safety / Ocean Exit Module

### Purpose

Determine whether returning later may be substantially harder than launching.

### Inputs

- Hourly forecast trend
- Wind direction relative to shore
- Wind increase
- Gust increase
- Wave and surf trend
- Daylight
- Distance from launch
- Bixpy reserve assumptions

### Outputs

```js
{
  returnScore,
  latestRecommendedReturn,
  worseningRisk,
  primaryReason,
  hourlyReturnStatus
}
```

### Use

Feeds Coach Mode, Trip Timeline, Hazard Intelligence, Battery Planner, and Mission Control.

---

## 4.5 Confidence Module

### Purpose

Express how trustworthy a recommendation is.

### Inputs

- Forecast completeness
- Agreement across available inputs
- Forecast horizon
- Rapidly changing conditions
- Missing marine or tide data
- Estimated versus observed values

### Outputs

```js
{
  confidenceScore,
  confidenceBand,
  reasons,
  missingInputs
}
```

### Important distinction

Confidence is not safety. A high-confidence NO-GO is still a NO-GO, and a low-confidence GO should become more conservative.

---

## 4.6 Hazard Intelligence Module

### Purpose

Aggregate operational hazards not fully represented by the main score.

### Potential hazards

- Strong ebb current
- Large tidal exchange
- Offshore wind
- Rapid afternoon wind increase
- Fog or poor visibility
- Reduced daylight
- Heavy vessel traffic
- Pier, jetty, reef, kelp, or surf-zone hazards
- Water-quality advisories
- Known closures or access restrictions
- Forecast-data gaps

### Outputs

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

---

# 5. Fishing Intelligence Modules

## 5.1 Species Recommendation Engine

### Purpose

Rank realistic target species for the selected area and conditions.

### Inputs

- Area habitat
- Water temperature
- Tide stage
- Time of day
- Season
- Structure type
- Forecast safety
- Existing species rules

### Outputs

```js
{
  speciesRankings: [
    {
      species,
      probabilityBand,
      confidence,
      reasons,
      preferredStructure,
      preferredWindow
    }
  ]
}
```

### Rules

- Species logic never overrides safety.
- Protected-water species recommendations remain separate from exposed-ocean plans.
- Current rules remain unchanged unless explicitly requested.

---

## 5.2 Structure and Habitat Module

### Purpose

Describe the type of water or bottom to fish.

### Examples

- Sand flat
- Sand-to-reef transition
- Kelp edge
- Eelgrass edge
- Riprap
- Dock pilings
- Harbor channel edge
- Bait-school zone

### Outputs

```js
{
  preferredStructure,
  secondaryStructure,
  depthRange,
  searchPattern,
  moveRule
}
```

---

## 5.3 Gear Recommendation Module

### Purpose

Recommend the appropriate setup after safety, species, and structure are known.

### Inputs

- Target species
- Area
- Structure
- Depth
- Wind
- Current
- Position-holding method
- User-owned equipment

### Outputs

```js
{
  primaryRig,
  backupRig,
  lureOrBait,
  leader,
  weight,
  hook,
  snap,
  positionHoldingGear,
  requiredSafetyGear
}
```

### Rules

- Recommendations should prefer equipment the user already owns.
- Gear recommendations must not create new safety calculations.
- Position-holding gear must come from the Position Holding Module.

---

## 5.4 Position Holding Module

### Purpose

Recommend how to control the Hobie at each referenced area.

### Possible outputs

- Pedal positioning
- Bixpy positioning
- Controlled drift
- One stern drift sock
- Anchor on open sand
- Do not anchor
- Return to protected water
- End the trip

### Inputs

- Area-specific profile
- Wind
- Gusts
- Wave and surf energy
- Tide and current
- Target species
- Safety status
- Bottom type
- Vessel traffic
- Kelp, reef, jetty, dock, or surf hazards

### Outputs

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

### Rules

- Anchoring is never a method for remaining offshore in worsening conditions.
- Do not recommend anchoring in navigation channels, surf influence, heavy traffic, kelp, reef, or snag-prone structure.
- One drift sock is preferred over two for normal kayak fishing control.
- Two drift socks must never be framed as a substitute for returning.
- Area profiles must be stored separately from UI wording.

---

## 5.5 Drift Prediction Module

### Purpose

Estimate direction and speed of the kayak’s drift.

### Inputs

- Wind speed and direction
- Tide/current estimate
- Kayak orientation
- Drift sock status
- Area profile

### Outputs

```js
{
  driftDirection,
  driftSpeed,
  estimatedTrack,
  timeAcrossTargetZone,
  confidence
}
```

### Note

This module should not be built until its data assumptions and limitations are documented.

---

# 6. Trip Execution Modules

## 6.1 Automatic Trip Timeline

### Purpose

Turn module outputs into a chronological plan.

### Inputs

- Best launch window
- Best fishing window
- Return deadline
- Tide timing
- Wind trend
- Species strategy
- Bait plan
- Position-holding plan
- Battery reserve

### Outputs

```js
{
  timeline: [
    {
      time,
      action,
      reason,
      status
    }
  ]
}
```

### Rule

The timeline only sequences existing recommendations. It must not invent new scores.

---

## 6.2 Bixpy Battery Planner

### Purpose

Estimate usable power while protecting the return reserve.

### Inputs

- Battery model and capacity
- Estimated speed setting
- Planned motor use
- Wind and return conditions
- Distance
- Pedal contribution
- Required reserve

### Outputs

```js
{
  estimatedConsumption,
  recommendedReserve,
  maximumOutboundUse,
  projectedRemainingBattery,
  batteryStatus
}
```

### Rules

- Preserve a conservative return reserve.
- Do not present battery range as guaranteed.
- Return Safety should influence the required reserve.

---

## 6.3 Live Coaching Module

### Purpose

Provide time-sensitive instructions during a trip.

### Inputs

- Existing timeline
- Updated forecast
- Return deadline
- Hazard changes
- User-reported trip state

### Outputs

- Continue
- Change technique
- Move location
- Enter protected water
- Begin return
- End trip

### Rule

Live Coaching consumes existing module outputs and updated conditions. It does not create parallel safety logic.

---

## 6.4 Virtual Fishing Day

### Purpose

Simulate the planned day before departure.

### Inputs

- Trip Timeline
- Safety modules
- Species strategy
- Gear strategy
- Position Holding
- Return Safety

### Outputs

An hour-by-hour rehearsal.

### Rule

This is a presentation and education layer only.

---

# 7. Explanation and Transparency Modules

## 7.1 Score Explanation / “Why Not?”

### Purpose

Explain why a site received its result.

### Outputs

```js
{
  startingScore,
  adjustments: [
    {
      category,
      amount,
      reason
    }
  ],
  finalScore,
  hardOverrides
}
```

### Rule

The explanation must read values produced by the scoring engine. It must never recalculate the score.

---

## 7.2 Learning / Trip History Module

### Purpose

Store trip outcomes and compare predictions with actual results.

### Possible user inputs

- Launch used
- Actual wind
- Actual surf
- Catch
- Species
- Depth
- Bait or lure
- Water clarity
- Accuracy of forecast
- Return conditions
- Notes

### Future outputs

- Personal success patterns
- Forecast accuracy trends
- Preferred areas and techniques
- Suggested adjustments

### Rule

Learning data may personalize fishing recommendations but must not weaken fixed safety thresholds without explicit approval.

---

# 8. User Interface Modules

## 8.1 Seven-Day and 16-Day Outlook

### Purpose

Help choose the best future day before selecting a detailed trip.

### Rules

- Use only forecast horizons supported by available data.
- Do not imply precision beyond the source forecast.
- The outlook summarizes existing daily calculations.

---

## 8.2 Launch Decision

### Purpose

Show exposed-ocean safety rankings.

### Inputs

- Ocean Launch Safety Engine
- Hard NO-GO Engine
- Confidence Module
- Return Safety Module

### Rule

Protected-water areas cannot appear in this ranking.

---

## 8.3 Protected-Water Backup

### Purpose

Show the best sheltered alternative when exposed launches are poor.

### Inputs

- Protected-Water Safety Engine
- Species Recommendation Engine
- Position Holding Module

---

## 8.4 Species Recommendations

### Purpose

Show target species, structure, timing, and technique independently from launch safety.

### Rule

This page cannot override launch safety.

---

## 8.5 Coach Mode

### Purpose

Build a step-by-step trip plan from existing module outputs.

### Inputs

- Best usable launch
- Hard NO-GO result
- Protected-water fallback
- Species recommendation
- Structure recommendation
- Position Holding Strategy
- Gear recommendation
- Return deadline
- Timeline
- Hazards
- Confidence

### Rules

- If all exposed sites are below 60 or hard NO-GO, Coach Mode refuses to build an exposed-ocean plan.
- Coach Mode recommends the best protected-water fallback.
- Coach Mode does not calculate its own safety score.
- Coach Mode does not create its own species score.
- Coach Mode does not independently calculate anchor, drift, or battery recommendations.

---

## 8.6 Mission Control

### Purpose

Provide one consolidated trip screen.

### Required upstream modules

Mission Control should be built only after these modules exist and are validated:

- Launch Safety
- Protected-Water Backup
- Return Safety
- Confidence
- Hazard Intelligence
- Species Recommendation
- Gear Recommendation
- Position Holding
- Trip Timeline
- Battery Planner
- Drift Prediction
- Score Explanation

### Rule

Mission Control only displays and prioritizes structured outputs from these modules.

It must not introduce new calculations, thresholds, or safety decisions.

---

# 9. Area Profile Architecture

Each referenced location should have a structured profile.

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

Area profiles should supply facts and constraints. Logic modules should interpret those fields.

This prevents location-specific rules from being buried inside UI text.

---

# 10. Shared Output Contract

Every logic module should return a predictable object with:

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

---

# 11. Error Handling

The planner must distinguish between:

- Safe result
- Caution result
- Unsafe result
- Missing data
- Stale data
- API failure
- Unsupported forecast horizon

The planner must never silently replace missing marine data with a positive safety assumption.

---

# 12. Testing Requirements

Before every release, verify:

- Existing tabs still work.
- Live data still loads.
- Protected-water locations remain excluded from ocean rankings.
- Hard NO-GO behavior still works.
- Coach Mode refuses unsafe exposed-ocean plans.
- Protected-water fallback still appears.
- Version number is updated everywhere.
- Go Fish Live compatibility is preserved.
- No scoring logic changed unless requested.
- New UI components consume existing module outputs.
- Mobile layout works on iPhone.
- Forecast failures are visible.
- Anchor guidance remains area-specific and safety-first.

---

# 13. Version and Release Workflow

For each release:

1. Start from the latest authoritative source.
2. Preserve the previous version.
3. Make only requested changes.
4. Increment the version.
5. Update the changelog.
6. Document any new design decision.
7. Run the testing checklist.
8. Produce the standalone HTML.
9. Deploy or prepare for Go Fish Live.
10. Confirm that the displayed version matches the release version.

---

# 14. Recommended Future File Structure

The project may remain a single deployable HTML file, but development logic should be conceptually separated.

```text
gofish/
├── index.html
├── ARCHITECTURE.md
├── DESIGN_DECISIONS.md
├── CHANGELOG.md
├── ROADMAP.md
├── TESTING.md
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

A build step may later combine these modules into one standalone HTML file for Go Fish Live.

---

# 15. Current Development Order

## Phase 1 – Safety Foundation

1. Position Holding Strategy
2. Return Safety / Ocean Exit Score
3. Confidence Meter
4. Hazard Intelligence

## Phase 2 – Trip Execution

1. Automatic Trip Timeline
2. Gear Recommendations
3. Bixpy Battery Planner
4. Drift Prediction

## Phase 3 – Fishing Intelligence

1. Score Explanation
2. Species Probability
3. Seasonal Intelligence
4. Trip Learning

## Phase 4 – Integration

1. Mission Control
2. Live Coaching
3. Virtual Fishing Day

Mission Control remains last because it depends on the structured outputs of all prior modules.

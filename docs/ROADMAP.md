# Go Fish Planner – Roadmap

**Status:** Living roadmap  
**Purpose:** Define what will be built, in what order, and why.

## Vision

Build the safest, most practical Southern California kayak-fishing planner for the Hobie i12S. Every new feature must improve safety, clarity, or trip success while preserving existing functionality.

## Current Status

| Item | Value |
|---|---|
| Current Development Stage | Safety foundation |
| Deployment Target | Go Fish Live |
| Versioning | Increment every planned release |
| Development Model | One module at a time; validate before integration |

## Development Principles

- Preserve existing functionality unless explicitly replacing it.
- Never change scoring logic unless explicitly requested.
- Treat Go Fish Live as the primary deployment target.
- Build one module at a time and validate it independently.
- Mission Control is the final integration layer.
- UI components consume outputs from logic modules and never create duplicate calculations.

## Phase 1 – Safety Foundation

| Status | Module |
|---|---|
| Complete | Position Holding Strategy |
| In progress | Return Safety / Ocean Exit Score |
| Planned | Confidence Meter |
| Planned | Hazard Intelligence |

## Phase 2 – Trip Execution

| Status | Module |
|---|---|
| Planned | Automatic Trip Timeline |
| Planned | Gear Recommendations |
| Planned | Bixpy Battery Planner |
| Planned | Drift Prediction |

## Phase 3 – Fishing Intelligence

| Status | Module |
|---|---|
| Planned | Score Explanation ("Why Not?") |
| Planned | Species Probability |
| Planned | Seasonal Intelligence |
| Planned | Trip Learning |

## Phase 4 – Integration

| Status | Module |
|---|---|
| Planned | Mission Control |
| Planned | Live Coaching |
| Planned | Virtual Fishing Day |

## Backlog / Future Ideas

- Fish-probability heat map
- Wind/current animation
- Expanded launch library
- Additional species
- Additional forecast providers
- Trip sharing/export
- Community reports

## Release Checklist

- [ ] Existing functionality verified
- [ ] Version incremented
- [ ] `CHANGELOG.md` updated
- [ ] Design Decisions reviewed
- [ ] Architecture reviewed
- [ ] Mobile layout tested
- [ ] Go Fish Live deployment verified

## Parking Lot

Record promising ideas here without interrupting the approved roadmap. Items move into a development phase only after explicit approval.

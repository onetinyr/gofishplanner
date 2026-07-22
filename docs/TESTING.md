# Go Fish Planner – Testing

**Status:** Living document  
**Purpose:** Regression checklist to verify every release before deployment to Go Fish Live.

## Release Preparation

- [ ] Start from the latest authoritative source.
- [ ] Version number incremented everywhere.
- [ ] Only requested features implemented.
- [ ] Existing functionality intentionally preserved.

## Forecast and Data

- [ ] Live weather loads.
- [ ] Marine forecast loads.
- [ ] Tide data loads.
- [ ] API failures display clear messages.
- [ ] Only supported forecast horizons are used.

## Safety Engine

- [ ] Ocean launch rankings calculate correctly.
- [ ] Protected-water locations are excluded from ocean rankings.
- [ ] Hard NO-GO overrides all ocean recommendations.
- [ ] Return Safety behaves as expected.
- [ ] Confidence and Hazard modules, when present, are correct.

## Coach Mode

- [ ] Refuses exposed-ocean plans when all exposed sites are below threshold or hard NO-GO.
- [ ] Recommends the best protected-water fallback.
- [ ] Position Holding Strategy is area-specific.
- [ ] Anchor guidance remains safety-first.
- [ ] Species, gear, and timeline recommendations consume existing module outputs.

## Planner Pages

- [ ] Launch Decision
- [ ] Protected-Water Backup
- [ ] Species Recommendations
- [ ] Coach Mode
- [ ] Seven-day and 16-day Outlook
- [ ] Mission Control, when implemented

## Scoring Integrity

- [ ] No scoring logic changed unless explicitly requested.
- [ ] UI performs no independent safety calculations.
- [ ] Module outputs remain the single source of truth.

## Mobile and Deployment

- [ ] Layout verified on iPhone.
- [ ] Version label matches release.
- [ ] Standalone HTML works.
- [ ] Go Fish Live deployment verified.

## Regression Sign-off

- [ ] `CHANGELOG.md` updated.
- [ ] `DESIGN_DECISIONS.md` updated if needed.
- [ ] `ARCHITECTURE.md` updated if needed.
- [ ] `ROADMAP.md` updated if priorities changed.
- [ ] Release approved.

# Go Fish Planner -- Design Decisions (Living Document)

**Status:** Canonical design reference\
**Purpose:** Preserve the reasoning behind the planner so future updates
never accidentally remove or change intentional behavior.

# Vision

Go Fish Planner is a **safety-first Southern California kayak fishing
planner** designed primarily for the Hobie i12S with optional Bixpy
propulsion.

Primary goals, in order:

1.  Safety
2.  Successful fishing
3.  Simplicity
4.  Education

The planner should always prefer a conservative recommendation over an
aggressive one.

------------------------------------------------------------------------

# Design Philosophy

Every future enhancement should follow these rules:

-   **Always preserve existing functionality** unless the requested
    change explicitly replaces or removes it.
-   **Increment the version automatically** for every planned release.
-   **Never modify scoring logic unless explicitly requested.**
-   **Treat Go Fish Live as the primary deployment target.** New
    features should be compatible with the hosted version and avoid
    breaking deployment.
-   **Never introduce new calculations inside the UI.** Every
    calculation should live in a dedicated logic module so Coach Mode,
    Mission Control, and other interface components consume existing
    outputs rather than creating their own scoring or safety logic. This
    maintains a single source of truth for every decision.

------------------------------------------------------------------------

# Core Design Principles

## Safety First

-   The planner is intentionally conservative.
-   A missed fishing day is preferable to an unsafe launch.
-   Coach Mode must never encourage overriding a hard NO-GO.

## Ocean vs Protected Water

Ocean launches and protected-water launches are separate decision
spaces.

Protected-water locations are **never** included in the ocean launch
rankings.

If every exposed launch is below the safety threshold or has a hard
NO-GO:

-   Coach Mode refuses to build an exposed-ocean plan.
-   Coach Mode recommends the best protected-water alternative.

## Coach Mode

Coach Mode should explain:

-   Where to launch
-   When to launch
-   What species to target
-   What bait or lure to use
-   How to position the kayak
-   When to return

Coach Mode is intended to function like an experienced kayak fishing
guide.

## Position Holding Strategy

Recommendations are location-specific and may recommend pedaling, Bixpy
positioning, drift socks, anchoring, or returning to shore. Anchor
recommendations must never encourage remaining offshore in deteriorating
conditions.

## Versioning

Every release receives a new version number while preserving existing
functionality unless intentionally changed.

## Roadmap Priorities

Phase 1 - Return Safety / Ocean Exit Score - Confidence Meter - Hazard
Intelligence - Position Holding Strategy

Phase 2 - Trip Timeline - Gear Recommendations - Bixpy Battery Planner -
Drift Prediction

Phase 3 - Score Explanation - Species Intelligence - Seasonal
Intelligence - Trip Learning

Phase 4 - Mission Control - Live Coaching - Virtual Fishing Day

Mission Control is the final integration layer and should consume
existing modules rather than introducing new calculations.

------------------------------------------------------------------------

## Changelog Notes

Future versions should append: - Version - Date - Summary - Design
rationale

# Go Fish Planner – Project Setup

**Status:** Living document  
**Purpose:** Quick-start reference for every development session.

## Project Identity

| Setting | Value |
|---|---|
| Project Name | Go Fish Planner |
| Deployment Name | Go Fish Live |
| Current Authoritative Version | v19 after verification |
| Primary Deployment Target | GitHub Pages (Go Fish Live) |
| Primary Development Workspace | ChatGPT Go Fish Project |

## Development Rules

- Always start from the latest authoritative source file.
- Preserve existing functionality unless explicitly replacing it.
- Increment the version number for every planned release.
- Never modify scoring logic unless explicitly requested.
- Treat Go Fish Live as the deployment target.
- UI modules consume outputs from logic modules; do not duplicate calculations.

## Primary References

- `DESIGN_DECISIONS.md`
- `ARCHITECTURE.md`
- `ROADMAP.md`
- `CHANGELOG.md`
- `TESTING.md`

## Typical Release Workflow

1. Review `ROADMAP.md` and identify the approved feature.
2. Update `ARCHITECTURE.md` if a new module is introduced.
3. Implement only the requested feature.
4. Run the `TESTING.md` checklist.
5. Update `CHANGELOG.md`.
6. Update `DESIGN_DECISIONS.md` if new philosophy or behavior is introduced.
7. Increment the version.
8. Verify the displayed version matches the release.
9. Deploy to Go Fish Live.

## Session Startup Checklist

- [ ] Confirm the current authoritative version.
- [ ] Confirm Go Fish Live matches the latest approved release.
- [ ] Confirm the correct source file is loaded.
- [ ] Review unfinished roadmap items.
- [ ] Verify that no requested change conflicts with Design Decisions.

## Current Next Tasks

- Treat verified v19 as the authoritative source.
- Keep v19 and all project documents in the GitHub repository.
- Begin the Return Safety / Ocean Exit Score module.

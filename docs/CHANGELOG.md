# Go Fish Planner – Changelog

**Status:** Living document  
**Purpose:** Record every release, what changed, and why.

## Release Entry Template

### Version

**Release Date:**  
**Status:** Planned / Released

#### Summary

- ...

#### Features

- ...

#### Bug Fixes

- ...

#### Behavior Changes

- ...

#### Design Rationale

- ...

#### Compatibility

- Go Fish Live: Verified / Pending
- Existing functionality preserved: Yes / No

#### Testing

- Live forecast retrieval
- Ocean launch rankings
- Protected-water fallback
- Coach Mode
- Mobile layout

## Recorded Releases

### v19

**Summary:** Area-specific Position Holding Strategy.

**Design rationale:** Added area-specific anchor, drift-sock, and position-holding guidance to Coach Mode while preserving existing scoring and safety logic.

### v18

**Summary:** Baseline release.

**Design rationale:** Latest baseline prior to area-specific position-holding enhancements.

### v17 and Earlier

**Summary:** Historical releases.

**Design rationale:** See archived HTML releases. As earlier versions are reviewed, migrate notable changes into this changelog.

## Release Rules

- Every planned release increments the version number.
- Preserve existing functionality unless explicitly requested otherwise.
- Do not modify scoring logic unless explicitly approved.
- Update this changelog before publishing Go Fish Live.
- Document the rationale behind every behavioral change.
- Record new design decisions in `DESIGN_DECISIONS.md`.
- Update `ARCHITECTURE.md` and `ROADMAP.md` when module boundaries or priorities change.

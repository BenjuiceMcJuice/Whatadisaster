# Changelog

All notable changes to the Emergency Exercise Simulator.

## [Unreleased]

### Fixed
- Removed a duplicated block of screen markup (`#fireMap`, `#game`, `#results`, `#modal` were each defined twice), eliminating ~140 lines of invalid duplicate-ID HTML.
- "8 roles" copy corrected to "9 roles" on the splash screen and in `how-it-works.html` (the Voluntary Agency Coordinator brought the total to 9 in v1.2.0).
- Added the missing Voluntary Agency Coordinator card to the `how-it-works.html` role directory.
- In-game header now reads "Exercise Ashdown / Exercise Solstice" to match the naming used everywhere else (previously "Op. …").
- Corrected the `how-it-works.html` timed-question copy (claimed 30–90s; actual timers are ~10–25s) and generalised the between-phases map description to cover both scenarios.
- Removed dead code referencing a non-existent `fireSvg` element in `showFireMap()`.
- Re-enabled pinch-zoom (dropped `maximum-scale`/`user-scalable=no` from the viewport meta) for accessibility.

### Changed
- `APP_VERSION` bumped to `1.2.0` to reflect the shipped Voluntary Agency Coordinator release.

## [1.2.0] — 2026-06-25

### Added
- **New role: Voluntary Agency Coordinator** (🤝, Bronze, LRF Voluntary Sector) across both scenarios, with role-specific learning points and a five-question bank per scenario (Operation Solstice Q3 timed, 25s). Uses fictional voluntary orgs (Avon Emergency Volunteers, Brynmoor Community First).

### Fixed
- Stakeholder content-accuracy corrections: NWFC → **NRFC** (and framed as a Fire-commander decision); hospital Major Incident Plan activation framed as the hospital's own decision, not a TCG request; extreme-heat crew safety made conditional on no other symptomatic crew.

## [1.1.2] — 2026-05-28

### Fixed
- Game header always displayed "Op. Ashdown" regardless of scenario. `getElementById('scenarioTag')` was targeting a non-existent element; replaced with `querySelectorAll('.gtt')` to correctly update both game screen headers on role selection. Operation Solstice now shows "Op. Solstice" in the header.

---

## [1.1.1] — 2026-03-22

### Fixed
- Broken national guidance URLs corrected:
  - `heat-health-alerts` → `heat-health-alerting-system` (GOV.UK page was renamed)
  - `national-risk-register` → `national-risk-register-2025` (publication moved to year-specific URL)
  - `met.gov.uk/...` → `metoffice.gov.uk/services/government/environmental-hazard-resilience/fire-severity-index` (wrong domain entirely; `met.gov.uk` does not exist)

---

## [1.1.0] — 2026-03-22

### Added
- **Scenario-specific pre-exercise briefing**: clicking a scenario button now routes through the briefing before role selection. The briefing dynamically populates based on the selected scenario — Operation Ashdown shows wildfire hazard content; Operation Solstice shows heatwave hazard content (UKHSA Heat-Health Alert levels, urban heat island effect, vulnerable populations, tropical nights, heat illness recognition).
- **Solstice-specific Business Continuity inject**: heatwave BC scenario (staff sent home due to extreme heat, office at 34°C, staff sickness) replaces the wildfire zone BC inject for Operation Solstice.
- **Scenario-specific national guidance links**: Operation Solstice briefing links to UKHSA Heat-Health Alert system and Bristol heat-specific resources; Operation Ashdown links to wildfire and national risk guidance.
- **Role-specific learning points**: end-of-game learning points are now tailored per role (8 roles) and per scenario (Ashdown / Solstice), replacing a single generic list. Each role receives 4 targeted learning points plus 2 scenario-specific shared points.
- **APP_VERSION constant**: version number declared in the script block (`1.1.0`).

### Fixed
- Wildfire-specific learning points (embers, road clearance) no longer appear when playing Operation Solstice.
- "Review Briefing" button in results now shows the correct scenario briefing rather than always showing Ashdown content.

### Changed
- Scenario selection buttons on the splash screen now go to briefing first, then role select — making the pre-exercise brief part of the standard flow rather than an optional detour.

---

## [1.0.0] — 2026-03-22 (baseline)

Initial tracked version. Features at this point:
- Two scenarios: Operation Ashdown (wildfire, Brynmoor Hill) and Operation Solstice (severe heatwave, Bristol)
- Eight playable roles across Gold / Silver / Bronze command levels
- Four-phase question flow with timed pressure decisions and consequence meters (Life Safety, Infrastructure, Community, Coordination)
- Phase-transition fire/heat map animations (SVG)
- Branching questions based on early decisions (e.g. `noTCG` flag)
- Post-game results screen with decision breakdown, learning points, and outcome band (Negligible → Catastrophic)
- Debrief screen with full decision review
- Exercise record submission via mailto to emergency planning team
- `how-it-works.html` explainer page

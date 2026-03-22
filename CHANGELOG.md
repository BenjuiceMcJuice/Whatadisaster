# Changelog

All notable changes to the Emergency Exercise Simulator.

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

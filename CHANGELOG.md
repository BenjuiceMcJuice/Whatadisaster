# Changelog

All notable changes to the Emergency Exercise Simulator.

> **Note:** version numbering is out of sync — `APP_VERSION` in `index.html` is still `1.1.1` and this log skips the voluntary-role release (v1.2.0 in `DEVLOG.md`), the P0 accessibility pass, timer changes, and the LinkedIn launch prep. The `[1.3.0]` header below is provisional pending a decision to reconcile `APP_VERSION`, this log, and the DEVLOG milestones.

## [1.3.0] — 2026-07-19

### Changed
- **Game-screen density overhaul (UI/UX assessment Phase 2 / P1).** The per-question screen dropped from ~2.6 to ~1.0–1.3 viewport-heights at 390×844 so the decision is the clear focus:
  - Replaced the always-on 4-meter Consequence Tracker with a slim status bar carrying a single consequence pill (colour dot + level word, e.g. 🟢 Controlled).
  - Moved the Consequence Tracker, Activated Agencies, and Command Level into a tap-to-open bottom "Situation Status" sheet — reachable from the pill or a quiet "Agencies & Command Level" bar below the decision. No information removed.
  - Made the situation panel collapsible; collapsed, it becomes a sticky one-line recap under the status bar so the scenario stays glanceable while choosing.
  - Capped situation stat chips at 3 with a "+N more" reveal.
- **Calmer motion & colour.** Removed the three simultaneous pulses/glow (only the pressure bar animates on timed questions; nothing pulses on untimed ones); reserved red for wrong-answer/timer states and recoloured "danger" chips to the fire-orange accent. Added `prefers-reduced-motion` support (disables embers, pulses, transitions).

Question content and scoring unchanged. Verified both scenarios play start→results with no JS errors.

---

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

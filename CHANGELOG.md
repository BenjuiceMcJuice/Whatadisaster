# Changelog

All notable changes to the Emergency Exercise Simulator.

`APP_VERSION` in `index.html` tracks the current release (now `1.8.0`). Versions 1.2.0–1.5.0 below were backfilled on 2026-07-19 from the `DEVLOG.md` milestones, which the changelog had fallen behind.

## [1.8.0] — 2026-07-20

### Changed (graphics overhaul)
- **Rebuilt the phase-transition maps as high-resolution tactical graphics (both scenarios).** Replaced the growing-ellipse SVG maps with a `<canvas>` engine rendered at 1024×576 with real-font labels (crisp, no more upscaled-thumbnail blur):
  - *Operation Ashdown (wildfire)* — a **spreading fire**: flame clusters, rising smoke and ember spot-fires grow across the three phase maps, over conifer terrain, a building road-network and a widening burn glow.
  - *Operation Solstice (heatwave)* — a **site-severity board**: each vulnerable site (care home, high-rise blocks, hospital ED, school, estate) carries a `MONITORED` / `AT RISK` / `CRITICAL` status tag + colour-coded outline that escalate each stage, under a rising Heat-Health Alert badge and time·temp readout. Represents cumulative impact, not spatial spread — the tropical-night stage stays critical though the temperature falls.
- **Pixel-art UI icons** replace the prominent emoji: splash emergency beacon, scenario fire/sun buttons, briefing & disclaimer document, dev-build warning, and the fire-map fact label.
- Respects `prefers-reduced-motion` (renders a static map frame; no continuous animation). Still one self-contained HTML file, no new dependencies.

---

## [1.7.1] — 2026-07-19

### Fixed
- **Fire-map heat blobs never animated in Operation Solstice.** The heatwave (Cabot) and wildfire (Ashdown) SVGs both used the same shape ids (`fire1`–`fire5`, `emb1`–`emb3`), so `showFireMap()`'s `getElementById` always hit the hidden Ashdown SVG and animated blobs the player couldn't see. Gave the Cabot SVG unique ids (`cfire1`…/`cemb1`…) and prefixed the lookups by scenario. Also removes the invalid duplicate-id HTML. The map, fact panel, and Continue were unaffected and still work.

### Changed (UI/UX assessment Phase 4 / P3 polish)
- Added `:focus-visible` outlines to the remaining interactive controls (consequence pill, context bar, situation toggle, status-sheet close, "Before you start" accordion, "+N more" chip) for keyboard/switch users.
- The role grid's 9th card (Voluntary Agency Coordinator) now spans the full width when it would otherwise sit orphaned on its own row.

---

## [1.7.0] — 2026-07-19

### Changed
- **Splash simplification (UI/UX assessment Phase 3 / P2).** Reworked the landing screen so the play action leads instead of trailing five text blocks:
  - Moved the two scenario buttons **above the fold** — a scenario CTA is now fully visible on a 390×844 phone without scrolling (first button at ~510px; splash content height ~2,900px → ~1,130px).
  - De-jargoned the lead: a plain-language strapline ("Play a Gold, Silver or Bronze command role in a live emergency — make the calls and watch the consequences unfold") plus a facts line (Free · No login · ~15 minutes · 9 roles · 2 scenarios) replace the acronym-first intro.
  - Demoted the "Exercise Framing" (WCS/RWCS) and "During the Exercise, Consider" (BC / Response Plan / National Guidance) panels into a collapsible "Before you start — facilitator notes" accordion. Content preserved, not removed.

### Fixed
- Splash copy said "8 roles"; corrected to **9** (the Voluntary Agency Coordinator, added in 1.2.0, was uncounted). No other stale role-count copy found in `index.html` / `how-it-works.html`.

---

## [1.6.0] — 2026-07-19

### Changed
- **Game-screen density overhaul (UI/UX assessment Phase 2 / P1).** The per-question screen dropped from ~2.6 to ~1.0–1.3 viewport-heights at 390×844 so the decision is the clear focus:
  - Replaced the always-on 4-meter Consequence Tracker with a slim status bar carrying a single consequence pill (colour dot + level word, e.g. 🟢 Controlled).
  - Moved the Consequence Tracker, Activated Agencies, and Command Level into a tap-to-open bottom "Situation Status" sheet — reachable from the pill or a quiet "Agencies & Command Level" bar below the decision. No information removed.
  - Made the situation panel collapsible; collapsed, it becomes a sticky one-line recap under the status bar so the scenario stays glanceable while choosing.
  - Capped situation stat chips at 3 with a "+N more" reveal.
- **Calmer motion & colour.** Removed the three simultaneous pulses/glow (only the pressure bar animates on timed questions; nothing pulses on untimed ones); reserved red for wrong-answer/timer states and recoloured "danger" chips to the fire-orange accent. Added `prefers-reduced-motion` support (disables embers, pulses, transitions).

Question content and scoring unchanged. Verified both scenarios play start→results with no JS errors.

---

## [1.5.0] — 2026-07-16

### Added
- **Product name & social preview (LinkedIn launch prep).** Adopted the full title "What a Disaster — Multi-Agency Emergency Exercise Simulator" and surfaced the brand on the splash (h1 + subtitle). Added Open Graph / Twitter preview tags + meta description and a designed 1200×630 `og-image.png` to `index.html` and `how-it-works.html` so shared links unfurl as a proper card.

### Changed
- **Corrected the disclaimer's privacy statements** to accurately describe what the app now uses — cookieless Cloudflare analytics, anonymous usage events, and voluntary-only feedback — replacing the out-of-date "no analytics / no data leaves your device" wording.

---

## [1.4.1] — 2026-07-15

### Changed
- **Timed decisions given more time** (repeat stakeholder feedback that countdowns were still too short even after click-to-reveal): added a 2.5s frozen "READ THE OPTIONS" reading grace after reveal, and extended all timed countdowns ~+50% (10→15, 12→18, 15→22, 25→37s). Timeout severity unchanged.

---

## [1.4.0] — 2026-07-12

### Added
- **Feedback widget** via the shared Benjuicey Apps backend (Cloudflare Worker + Firestore), tagged `WDA` — replaces the previous standalone Firestore write.

### Changed
- **Timed questions are now click-to-reveal** — options and the countdown sit behind a "Reveal Options" button so the clock no longer starts before the player has read the question (stakeholder feedback).
- **Answer-length telegraphing rebalanced** across all roles and both scenarios (~97 questions) so the correct option no longer stands out by length/detail.

### Accessibility (UI/UX assessment Phase 1 / P0)
- Re-enabled pinch-zoom and added safe-area insets (`viewport-fit=cover`).
- Removed duplicated DOM blocks so every screen id is unique.
- Modal backdrop tap no longer skips the question — only "Continue →" advances.
- Added an in-game exit (✕ → confirm) so players can leave mid-exercise.
- Role cards are now real `<button>`s with aria-labels and focus-visible outlines.

---

## [1.3.0] — 2026-07-10

### Added
- **Cloudflare Web Analytics** — cookieless, no consent banner.
- **Anonymous usage-event logging** to a Firebase/Firestore `events` collection (role selected, scenario started/completed, question answered, drop-off) — no personal data captured.

---

## [1.2.0] — 2026-06-25

### Added
- **New role: Voluntary Agency Coordinator** (Third Sector Liaison) in both scenarios — 6 questions each, fictional voluntary organisations, a local-authority activation cascade, and a timed decision in Operation Solstice. Decisions are grounded in the Civil Contingencies Act 2004 "have regard" duty, LRF/SCG coordination, and appropriate volunteer tasking (welfare/rest centres vs frontline response). Brings the roster to nine roles.

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

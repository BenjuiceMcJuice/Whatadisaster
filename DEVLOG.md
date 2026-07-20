# DEVLOG — What a Disaster

Milestone tracker. Updated when a feature or significant step is complete, not after every file change.
Granular daily work is in `logs/YYYY-MM-DD.md`.

---

## Milestones

**Release version map** (reconciled with `CHANGELOG.md` + `APP_VERSION` on 2026-07-19): 06-25 = **v1.2.0** (voluntary role) · 07-10 = **v1.3.0** (analytics/telemetry) · 07-12 = **v1.4.0** (feedback widget, click-to-reveal, answer rebalance, P0 a11y) · 07-15 = **v1.4.1** (longer timers) · 07-16 = **v1.5.0** (LinkedIn launch) · 07-19 = **v1.6.0** (game-screen density / P1) · 07-19 = **v1.7.0** (splash simplification / P2) · 07-19 = **v1.7.1** (fire-map Solstice bug fix + P3 polish) · 07-20 = **v1.8.0** (graphics overhaul — hi-res tactical maps + pixel-art icons). `APP_VERSION` in `index.html` now tracks the current release (1.8.0).

| Date       | Milestone                                                        | Status   |
|------------|------------------------------------------------------------------|----------|
| 2026-03-22 | v1.0.0 — Initial tracked version: two scenarios, eight roles, consequence meters, debrief, mailto submission | ✅ Done |
| 2026-03-22 | v1.1.0 — Scenario-specific briefings, role-specific learning points, APP_VERSION constant | ✅ Done |
| 2026-03-22 | v1.1.1 — Broken national guidance URLs fixed (GOV.UK, Met Office, NRR) | ✅ Done |
| 2026-05-28 | Custom domain whatadisaster.uk registered + Cloudflare Pages deploy | ✅ Done |
| 2026-05-28 | SDLC docs, DEVLOG, daily logs structure added                    | ✅ Done |
| 2026-05-28 | Bug fix: game header always showed "Op. Ashdown" in Solstice     | ✅ Done |
| 2026-06-25 | v1.2.0 — New role: Voluntary Agency Coordinator, both scenarios (6 questions, fictional orgs, LA activation cascade, timed Q3 in Solstice) | ✅ Done |
| 2026-07-10 | Cloudflare Web Analytics enabled (cookieless, no consent banner) | ✅ Done |
| 2026-07-10 | Firebase project + Firestore live; usage event logging (role/scenario/completion/drop-off) wired into `index.html` → `events` collection. Security rules written and published in console. | ✅ Done |
| 2026-07-12 | Feedback switched from a standalone Firestore write to the **shared Benjuicey Apps feedback backend**, merged to `main` and confirmed **live on whatadisaster.uk** — see below. Old `feedback` Firestore collection/rules removed from this app. | ✅ Done |
| 2026-07-12 | Timed questions switched to click-to-reveal: options + countdown are now hidden behind a "Reveal Options" button instead of starting immediately on question load (stakeholder feedback — countdowns started before players had read the options) | ✅ Done |
| 2026-07-12 | Answer-length telegraphing fixed for all remaining roles (`strategic`, `tactical`, `fire`, `police`, `council`, `nhs`, `utilities`, `ambulance`), both scenarios — ~97 questions rebalanced so distractors match the correct answer's length/detail, extending the `d4dd910` `voluntary`-role fix | ✅ Done |
| 2026-07-12 | **UI/UX assessment (mobile-focused) completed** — full review of `index.html`/`how-it-works.html` at iPhone/Android viewports. Severity-ranked spec: `info/ui-ux-assessment-2026-07-12.md`. See Backlog → "UI/UX mobile overhaul". | ✅ Done (spec) |
| 2026-07-12 | **UI/UX P0 (correctness + a11y) shipped** — re-enabled pinch-zoom + safe-area insets; deleted the 4 duplicated DOM blocks (every screen id now unique); modal backdrop no longer skips the question; added in-game exit (✕ → confirm); role cards are real `<button>`s with aria-labels + focus outlines. Verified: both scenarios play through to results incl. fire-map, 0 JS errors. Phase 1 of the assessment spec. | ✅ Done |
| 2026-07-15 | Timed decisions given more time (repeat feedback: still too short despite click-to-reveal) — added a 2.5s frozen "👁 READ THE OPTIONS" reading grace after reveal before the countdown drains, and extended all 16 `timerSecs` ~+50% (10→15, 12→18, 15→22, 25→37). Timeout severity unchanged. Verified in-browser + a deterministic tick test of the real `startPressTimer`. | ✅ Done |
| 2026-07-20 | **v1.8.0 — Graphics overhaul (hi-res tactical maps + pixel-art icons) — LIVE** — Replaced the growing-ellipse SVG phase maps with a 1024×576 `<canvas>` engine + real-font labels. Ashdown = spreading wildfire (flame clusters + rising smoke + ember spots grow each stage over conifer terrain, a building road-network and a widening burn glow); Solstice = heatwave site-severity board (per-site `MONITORED`/`AT RISK`/`CRITICAL` tags + colour-coded outlines escalating under a rising Heat-Health Alert badge + time·temp readout; represents cumulative impact, not spatial spread — tropical-night stays critical as temp falls). Prominent emoji swapped for pixel-art icons (splash beacon, scenario fire/sun buttons, briefing/disclaimer document, dev-build warning, fact label). `prefers-reduced-motion` respected; still one self-contained HTML file, no new deps. Verified both scenarios × 3 stages + beacon + reduced-motion, 0 JS errors. Merged to `main`. | ✅ Done |
| 2026-07-19 | **v1.7.1 — Fire-map Solstice animation bug fixed + UI/UX P3 polish — LIVE** — Cabot (heatwave) SVG given unique shape ids (`cfire*`/`cemb*`) so `showFireMap()` animates the visible map, not the hidden Ashdown one (also kills duplicate-id HTML); added `:focus-visible` to the remaining new controls; 9th role card spans full width instead of orphaning. Verified: Solstice blobs now animate (cfire1 opacity 1/rx 30) while Ashdown blobs stay dormant, both scenarios play to results, 0 JS errors. Merged to `main`. | ✅ Done |
| 2026-07-19 | **v1.7.0 — UI/UX P2 (splash simplification) shipped — LIVE** — scenario CTAs moved above the fold (first button ~510px, splash height ~2,900→~1,130px); plain-language strapline + facts line replace the acronym-first lead; WCS/RWCS + BC/Response-Plan/National-Guidance framing demoted into a "Before you start" accordion (content preserved); stale "8 roles" → **9**. Both scenarios still play through; 0 JS errors. Merged to `main`. | ✅ Done |
| 2026-07-19 | **UI/UX P1 (game-screen density) shipped — LIVE on whatadisaster.uk** — the "too busy" game screen rebuilt per Phase 2 of the assessment: slim status bar + single consequence pill; Consequence Tracker/Activated Agencies/Command Level demoted into a tap-to-open status sheet; situation panel collapsible → sticky one-line recap; calmed motion (removed 3 simultaneous pulses/glow, red reserved for urgency); stat chips capped at 3 + "+N more"; `prefers-reduced-motion` added. Game screen ~2.6 → ~1.0–1.3 screenfuls at 390×844 (≤1.6 target met). Both scenarios verified start→results, 0 JS errors. Content/scoring untouched. Merged to `main` (fast-forward `a1427c1`→`44f401c`) and deployed via Cloudflare Pages. | ✅ Done |
| 2026-07-16 | **LinkedIn launch prep shipped** — adopted the full name/title **"What a Disaster — Multi-Agency Emergency Exercise Simulator"** and surfaced the brand on the splash (h1 "What a **Disaster**" + subtitle); added Open Graph/Twitter preview tags + meta description to `index.html`/`how-it-works.html` with a designed 1200×630 `og-image.png` so shared links unfurl as a proper card; **corrected the disclaimer's now-inaccurate privacy statements** to describe the cookieless Cloudflare analytics, anonymous usage events, and voluntary-only feedback the app actually uses. Plan: `info/linkedin-launch-plan.md`. Merged to `main` and confirmed **live** — card verified rendering in the real LinkedIn composer. JESIP posture: independent tool referring to public doctrine, disclaimer disclaims accreditation/endorsement (no consultation required). | ✅ Done |

---

## Feedback: shared Benjuicey Apps backend — ✅ LIVE

**Status (2026-07-12):** merged to `main`, deployed, confirmed live on whatadisaster.uk — the feedback button (bottom-right) loads and submits successfully. Nothing pending on this app's side.

**Standard:** this app follows the cross-app feedback standard — see the central docs in the `Benjuicey-apps` repo, `docs/`: **feedback-standard.md** (the standard: schema, canonical categories) and **feedback-how-it-works.md** (end-to-end flow, the admin dashboard/API, and how to have Claude triage/categorise new submissions — on demand or scheduled). This app's trigram is `WDA`; categories are the canonical `bug`/`content`/`request`/`general`.

This app no longer has its own feedback storage. The feedback button is the embeddable widget from the `Benjuicey-apps` repo's Cloudflare Worker (`https://benjuicey-feedback.benjuicemcjuice.workers.dev/widget.js`, loaded via `<script defer data-app-id="whatadisaster">` in `index.html`). Submissions go to the shared Firestore project, tagged `WDA`, ref format `WDA-0001` etc. Every app Ben builds uses the same widget — BetaLog adopted it the same day; see `Benjuicey-apps/docs/feedback-standard.md`'s adoption table for the rest.

**Email notification (admin-wide, not app-specific):** the Worker can now email Ben on every new submission across all apps, but it's **dormant** until he sets a real `RESEND_API_KEY` secret — see `Benjuicey-apps/docs/feedback-how-it-works.md` → "Turning email on". Nothing to do here.

**What's still local to this app:** anonymous usage-event logging (`role_selected`, `scenario_started`, `question_answered`, `scenario_completed`, `feedback_submitted` — the last one fires off the widget's `benjuiceyfeedback:submitted` event) — stays in the `whatadisaster` Firebase project's `events` collection since it's app-specific analytics, not feedback. `firestore.rules` here now only covers `events` — published in console 2026-07-12.

---

## Backlog

From `info/whatadisaster-changes-briefing.md`:

| Priority | Item |
|----------|------|
| **Next** | **LinkedIn launch readiness** — ~~Cloudflare Web Analytics~~ ✅, ~~Feedback widget + usage events~~ ✅, ~~timer click-to-reveal~~ ✅, ~~stakeholder content-accuracy fixes (TCG/hospital MI plan, NWFC→NRFC, crew safety, ambulance surge)~~ ✅ 2026-06-11. Still open: social preview tags + OG image, better title. Full spec: `info/linkedin-launch-readiness-spec.md` |
| **High** | **UI/UX mobile overhaul** — build from `info/ui-ux-assessment-2026-07-12.md`. **~~P0~~ ✅ shipped 2026-07-12** (pinch-zoom, dup-DOM removal, backdrop-tap fix, in-game exit, role-card buttons, safe-area). **~~P1~~ ✅ shipped 2026-07-19** (slim status bar + consequence pill; Agencies/Command → status sheet; collapsible/sticky situation; calm colour/motion; chip cap; reduced-motion — ~2.6→~1.0–1.3 screenfuls). **~~P2~~ ✅ shipped 2026-07-19** (splash: CTAs above the fold, plain-language lead, framing → accordion, "8 roles"→9; OG/title tags already done in the LinkedIn item above). **~~P3~~ ✅ shipped 2026-07-19** (v1.7.1: `:focus-visible` on remaining controls; orphaned 9th role card spans full width; emoji already paired with text labels — verified; reduced-motion done in P1; E1 timer already extended 2026-07-15). **UI/UX overhaul complete (P0–P3).** Preserves single-file/no-build, fire-orange identity, and all question content. |
| ~~Medium~~ | ~~**Fire-map blob animation bug (Solstice)**~~ — fixed 2026-07-19 (**v1.7.1**): gave the Cabot SVG unique shape ids (`cfire*`/`cemb*`) and prefixed `showFireMap()`'s lookups by scenario, so the visible heatwave map's blobs animate (verified) and the duplicate-id HTML is gone. |
| ~~High~~ | ~~Bug: rename "Operation Ashdown" → "Operation Solstice" in Solstice scenario display text~~ — fixed 2026-05-28 |
| ~~High~~ | ~~New role: Voluntary Agency Coordinator — both scenarios~~ — shipped 2026-06-25 |
| Medium   | Heat-Health Alert (HHA) system — incorporate into Solstice questions and feedback |
| Medium   | Met Office Hazard Manager — reference in Solstice scenario |
| Medium   | UKHSA hot weather guidance — decisions around vulnerable populations and public messaging |
| ~~Medium~~ | ~~Answer-length balancing~~ — `d4dd910` (2026-06-26) fixed telegraphing for `voluntary`; the remaining 8 roles × both scenarios fixed 2026-07-12 (~97 questions). Considered moving question data to Firestore while at it; decided against — static content, instant load, no build/CLI pipeline currently set up for it. |
| Pending  | Extreme heat impacts content — awaiting slides from Heather |

# DEVLOG — What a Disaster

Milestone tracker. Updated when a feature or significant step is complete, not after every file change.
Granular daily work is in `logs/YYYY-MM-DD.md`.

---

## Milestones

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
| 2026-07-12 | **UI/UX assessment (mobile-focused) completed** — full review of `index.html`/`how-it-works.html` at iPhone/Android viewports. Severity-ranked spec written for a later build (not yet implemented): `info/ui-ux-assessment-2026-07-12.md`. See Backlog → "UI/UX mobile overhaul". | ✅ Done (spec) |

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
| **Next** | **LinkedIn launch readiness** — ~~Cloudflare Web Analytics~~ ✅, ~~Feedback widget + usage events~~ ✅, ~~timer click-to-reveal~~ ✅. Still open: social preview tags + OG image, better title, and the remaining stakeholder content fixes below. Full spec: `info/linkedin-launch-readiness-spec.md` |
| **High** | **UI/UX mobile overhaul** — build from `info/ui-ux-assessment-2026-07-12.md` (spec written 2026-07-12, not yet implemented). Phased: **P0** correctness/a11y (re-enable pinch-zoom `user-scalable=no`, delete duplicated `#game`/`#results`/`#modal`/`#fireMap` DOM blocks, stop modal backdrop-tap advancing the question, add in-game exit, role cards → real buttons, safe-area insets); **P1** game-screen density (slim status bar + consequence pill, Agencies/Command → drawer, persistent situation, calm colour/motion — target ≤1.6 screenfuls); **P2** splash (CTAs above the fold, de-jargon lead, "8 roles"→9) + OG/title tags (overlaps LinkedIn item above); **P3** polish. Preserves single-file/no-build, fire-orange identity, and all question content. |
| ~~High~~ | ~~Bug: rename "Operation Ashdown" → "Operation Solstice" in Solstice scenario display text~~ — fixed 2026-05-28 |
| ~~High~~ | ~~New role: Voluntary Agency Coordinator — both scenarios~~ — shipped 2026-06-25 |
| Medium   | Heat-Health Alert (HHA) system — incorporate into Solstice questions and feedback |
| Medium   | Met Office Hazard Manager — reference in Solstice scenario |
| Medium   | UKHSA hot weather guidance — decisions around vulnerable populations and public messaging |
| Medium   | Answer-length balancing — `d4dd910` (2026-06-26) fixed telegraphing (correct answer visibly longer/more detailed) for the `voluntary` role only, both scenarios. Still needs the same treatment on the other 8 roles (`strategic`, `tactical`, `fire`, `police`, `council`, `nhs`, `utilities`, `ambulance`) × both scenarios (`QB`/`QB2`) — ~102 questions. Considered moving question data to Firestore while at it; decided against — static content, instant load, no build/CLI pipeline currently set up for it. Fix in place in `index.html`. |
| Pending  | Extreme heat impacts content — awaiting slides from Heather |

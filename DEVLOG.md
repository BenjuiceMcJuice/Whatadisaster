# DEVLOG — What a Disaster

Milestone tracker. Updated when a feature or significant step is complete, not after every file change.
Granular daily work is in `logs/YYYY-MM-DD.md`.

---

## ⚠️ Needs attention next session

**Firestore rules not yet published.** `firestore.rules` (now `events`-only, see below) is written in the repo but hasn't been pasted into the Firebase console (Firestore → Rules → Publish) yet — Ben to do.

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
| 2026-07-10 | Firebase project + Firestore live; usage event logging (role/scenario/completion/drop-off) wired into `index.html` → `events` collection. Security rules written but **not yet published** in console. | 🟡 Code done, rules pending publish + live verification |
| 2026-07-12 | Feedback switched from a standalone Firestore write to the **shared Benjuicey Apps feedback backend**, merged to `main` and confirmed **live on whatadisaster.uk** — see below. Old `feedback` Firestore collection/rules removed from this app. | ✅ Done |

---

## Feedback: shared Benjuicey Apps backend — ✅ LIVE

**Status (2026-07-12):** merged to `main`, deployed, confirmed live on whatadisaster.uk — the feedback button (bottom-right) loads and submits successfully. Nothing pending on this app's side except the `firestore.rules` publish noted at the top of this file (and that only affects `events` analytics, not feedback).

**Standard:** this app follows the cross-app feedback standard — see the central docs in the `Benjuicey-apps` repo, `docs/`: **feedback-standard.md** (the standard: schema, canonical categories) and **feedback-how-it-works.md** (end-to-end flow, the admin dashboard/API, and how to have Claude triage/categorise new submissions — on demand or scheduled). This app's trigram is `WDA`; categories are the canonical `bug`/`content`/`request`/`general`.

This app no longer has its own feedback storage. The feedback button is the embeddable widget from the `Benjuicey-apps` repo's Cloudflare Worker (`https://benjuicey-feedback.benjuicemcjuice.workers.dev/widget.js`, loaded via `<script defer data-app-id="whatadisaster">` in `index.html`). Submissions go to the shared Firestore project, tagged `WDA`, ref format `WDA-0001` etc. Every app Ben builds uses the same widget — BetaLog adopted it the same day; see `Benjuicey-apps/docs/feedback-standard.md`'s adoption table for the rest.

**Email notification (admin-wide, not app-specific):** the Worker can now email Ben on every new submission across all apps, but it's **dormant** until he sets a real `RESEND_API_KEY` secret — see `Benjuicey-apps/docs/feedback-how-it-works.md` → "Turning email on". Nothing to do here.

**What's still local to this app:** anonymous usage-event logging (`role_selected`, `scenario_started`, `question_answered`, `scenario_completed`, `feedback_submitted` — the last one fires off the widget's `benjuiceyfeedback:submitted` event) — stays in the `whatadisaster` Firebase project's `events` collection since it's app-specific analytics, not feedback. `firestore.rules` here now only covers `events` — still needs publishing, see top of file.

---

## Backlog

From `info/whatadisaster-changes-briefing.md`:

| Priority | Item |
|----------|------|
| **Next** | **LinkedIn launch readiness** — ~~Cloudflare Web Analytics~~ ✅, ~~Feedback widget + usage events~~ ✅ (events rules pending publish). Still open: social preview tags + OG image, better title, and the stakeholder content/timer fixes below. Full spec: `info/linkedin-launch-readiness-spec.md` |
| ~~High~~ | ~~Bug: rename "Operation Ashdown" → "Operation Solstice" in Solstice scenario display text~~ — fixed 2026-05-28 |
| ~~High~~ | ~~New role: Voluntary Agency Coordinator — both scenarios~~ — shipped 2026-06-25 |
| Medium   | Heat-Health Alert (HHA) system — incorporate into Solstice questions and feedback |
| Medium   | Met Office Hazard Manager — reference in Solstice scenario |
| Medium   | UKHSA hot weather guidance — decisions around vulnerable populations and public messaging |
| Pending  | Extreme heat impacts content — awaiting slides from Heather |

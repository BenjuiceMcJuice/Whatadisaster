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
| 2026-07-11 | Feedback switched from a standalone Firestore write to the **shared Benjuicey Apps feedback backend** — see below. Old `feedback` Firestore collection/rules removed from this app. | ✅ Done |

---

## Feedback: shared Benjuicey Apps backend

This app no longer has its own feedback storage. The feedback button (bottom-right) is the embeddable widget from the `benjuicey-apps` repo's Cloudflare Worker (`https://benjuicey-feedback.benjuicemcjuice.workers.dev/widget.js`, loaded via `<script data-app-id="whatadisaster">` in `index.html`). Submissions go straight to the shared Firestore project, tagged with this app's trigram `WDA`, ref format `WDA-0001` etc. Every app Ben builds is meant to use the same widget — see `benjuicey-apps/docs/backlog.md` for the full platform plan (admin dashboard, AI analysis, etc).

**What's still local to this app:** anonymous usage-event logging (`role_selected`, `scenario_started`, `question_answered`, `scenario_completed`, `feedback_submitted` — the last one now fires off a `benjuiceyfeedback:submitted` event dispatched by the widget) — this stays in the `whatadisaster` Firebase project's `events` collection since it's app-specific analytics, not feedback. `firestore.rules` here now only covers `events`.

**Next step:** publish the trimmed `firestore.rules` in the Firebase console (Ben to do), and add `whatadisaster.uk` / `whatadisaster.pages.dev` to the Worker's CORS allowlist + redeploy the Worker (done in `benjuicey-apps` alongside this change — confirm it's live before this branch merges).

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

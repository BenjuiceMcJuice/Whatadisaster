# DEVLOG — What a Disaster

Milestone tracker. Updated when a feature or significant step is complete, not after every file change.
Granular daily work is in `logs/YYYY-MM-DD.md`.

---

## ⚠️ Blocked — needs attention next session

**Cross-app feedback integration.** The `feedback`/`events` Firestore collections built 2026-07-10 are standalone to this app. Ben says the feedback loop must wire into a parent/shared system across his apps (`benjuicey-apps` repo, alongside BetaLog/Dungeon of Montor/BenMed in the Firebase project list). This session's repo access doesn't include `benjuicey-apps`, so the shared system's shape is unknown. **Ben will add repo access and remind Claude to revisit this** — full context in `info/linkedin-launch-readiness-spec.md` under "Cross-app feedback integration." Don't treat the current standalone Firestore setup as final.

**Firestore rules not yet published.** `firestore.rules` is written in the repo but hasn't been pasted into the Firebase console (Firestore → Rules → Publish) yet.

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
| 2026-07-10 | Firebase project + Firestore live; community feedback form (persistent button, all screens) and usage event logging (role/scenario/completion/drop-off) wired into `index.html`. Firestore-only, no GA4 — see `firestore.rules`. Security rules written but **not yet published** in console. | 🟡 Code done, rules pending publish + live verification |

---

## Planned: Feedback & Release Tracking (Firebase)

Backend is now live (see milestone above: community feedback + usage events). Remaining from the original plan:

1. ~~**Community feedback form**~~ — done 2026-07-10. In-app form (category, free text, auto-attached role/scenario) writes to `feedback` collection with `source: 'community'`. Rate limiting still needed (Firestore rules alone can't do this — App Check or a Cloud Function is the likely next step if abuse becomes a problem).
2. **Partner / internal feedback tracker** — not started. Each item needs a human-readable reference (e.g. `WAD-FB-001`); auto-increment vs Firestore auto-ID still an open call. These items get added directly via console, not through the client write path (rules only allow `source: 'community'` from the client).
3. **Admin read UI** — not started. Auth-gated page to review the `feedback` collection. Google Auth is the simplest option.
4. **Structured release log** — not started. Firestore-backed changelog; could power a "What's new" panel in-app.

**Immediate next step:** `firestore.rules` is written but not yet published — paste it into Firebase Console → Firestore Database → Rules → Publish. Until then the collections have whatever default rules the console set at creation.

---

## Backlog

From `info/whatadisaster-changes-briefing.md`:

| Priority | Item |
|----------|------|
| **Next** | **LinkedIn launch readiness** — ~~Cloudflare Web Analytics~~ ✅, ~~Firebase feedback + usage events~~ ✅ (rules pending publish). Still open: social preview tags + OG image, better title, and the stakeholder content/timer fixes below. Full spec: `info/linkedin-launch-readiness-spec.md` |
| ~~High~~ | ~~Bug: rename "Operation Ashdown" → "Operation Solstice" in Solstice scenario display text~~ — fixed 2026-05-28 |
| ~~High~~ | ~~New role: Voluntary Agency Coordinator — both scenarios~~ — shipped 2026-06-25 |
| Medium   | Heat-Health Alert (HHA) system — incorporate into Solstice questions and feedback |
| Medium   | Met Office Hazard Manager — reference in Solstice scenario |
| Medium   | UKHSA hot weather guidance — decisions around vulnerable populations and public messaging |
| Pending  | Extreme heat impacts content — awaiting slides from Heather |

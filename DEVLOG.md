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

---

## Planned: Feedback & Release Tracking (Firebase — setup pending)

Goal: replace ad-hoc email feedback with a structured system. Firebase/Firestore is the intended backend — to be configured when next on laptop.

**Three things to build:**

1. **Partner / internal feedback tracker** — each item gets a human-readable reference (e.g. `WAD-FB-001`). Fields: source, date, category (content / UX / feature / bug), description, status, linked commit. Firestore collection: `feedback`.
2. **Community feedback form** — lightweight in-app form (role played, scenario, free text) writing to the same collection with `source: 'community'`. Rate limiting needed; no auth required.
3. **Structured release log** — Firestore-backed changelog (version, date, type, summary, feedback refs resolved). Could power a "What's new" panel in-app.

**Design decisions to make at setup time:**
- Reference ID format: auto-increment (`WAD-FB-001`) vs Firestore auto-ID — auto-increment friendlier for human reference
- Auth for admin review UI: Google Auth simplest
- Whether community form needs CAPTCHA / rate limiting
- Whether release log is surfaced in-app or internal only

**Build order when Firebase is ready:**
1. Create Firebase project, enable Firestore
2. Add Firebase SDK to `index.html` via CDN (no build step needed)
3. Community feedback write path first
4. Admin read UI separately (simple auth-gated HTML page)

---

## Backlog

From `info/whatadisaster-changes-briefing.md`:

| Priority | Item |
|----------|------|
| ~~High~~ | ~~Bug: rename "Operation Ashdown" → "Operation Solstice" in Solstice scenario display text~~ — fixed 2026-05-28 |
| High     | New role: Voluntary Agency Coordinator — both scenarios |
| Medium   | Heat-Health Alert (HHA) system — incorporate into Solstice questions and feedback |
| Medium   | Met Office Hazard Manager — reference in Solstice scenario |
| Medium   | UKHSA hot weather guidance — decisions around vulnerable populations and public messaging |
| Pending  | Extreme heat impacts content — awaiting slides from Heather |

# LinkedIn Launch Readiness — Spec

_Created: 2026-07-10 · Owner: Ben · Branch for work: `claude/app-roadmap-devlog-y3fhhd`_

**Trigger:** The app is being shared organically and is due to go on LinkedIn. A traffic spike is coming. This spec covers what to put in place **before and around that moment** so we (a) look good when shared, (b) can measure the spike, and (c) capture feedback from a lot of fresh eyes while we have their attention.

---

## TL;DR — the summary to remember

Three gaps today, in priority order:

1. **No social preview** — shared on LinkedIn it's a bare link titled "Emergency Exercise Simulator", no image. **Fix first.**
2. **No analytics** — we're blind on traffic we already have, and about to get more. Turn on **Cloudflare Web Analytics** (cookieless, no consent banner, ~1 toggle).
3. **No prominent feedback channel** — feedback only lives inside question content; the mailto submission also breaks on locked-down council devices. Add a **visible Feedback button** (Google Form interim).

**Two-stage plan:**

- **Stage 1 — Quick wins (ship before the post, ~half a day):** Open Graph/Twitter tags + preview image, better `<title>`, Cloudflare Web Analytics snippet, prominent Feedback button pointing to a Google Form. No backend, keeps the single-file/inspectable property.
- **Stage 2 — Firebase (the follow-up the LinkedIn moment justifies):** Build the already-DEVLOG-scoped Firebase project so it delivers **two things at once** — the structured feedback DB (replacing broken mailto) **and** custom usage events ("what's used vs not": role played, scenario, completion, drop-off).

**Key distinction to hold onto:** "how many people use it" (traffic → Cloudflare, trivial, cookieless) is a *different* question from "what inside it gets used" (events → Firebase or a Cloudflare Function). Do the first now; the second is what Firebase unlocks.

---

## Current state (verified 2026-07-10, `index.html`)

- ❌ No Open Graph or Twitter Card meta tags at all.
- ❌ `<title>` is the generic "Emergency Exercise Simulator".
- ❌ No analytics of any kind (no Cloudflare Insights, no gtag, no beacon).
- ❌ No prominent/visible feedback control in the UI. Submission path is the mailto (`submitToEPT()`), which — per `VISION.md` — fails on locked-down council devices with no configured mail client (exactly our core audience).
- ✅ Static on Cloudflare Pages → will absorb any LinkedIn traffic spike without issue. No scaling work needed.
- ⚠️ Viewport is locked (`maximum-scale=1.0, user-scalable=no`) — verify nothing is cramped on mobile, since LinkedIn traffic skews mobile.

---

## Stage 1 — Quick wins (do these before the LinkedIn post)

### 1.1 Social preview tags + image  ⭐ highest leverage
Turns the share from a bare link into a proper card (title, one-line description, screenshot).

- Add to `<head>` of `index.html` (and `how-it-works.html`):
  - `og:title`, `og:description`, `og:type` (`website`), `og:url` (`https://whatadisaster.uk`), `og:image`
  - `twitter:card` = `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image`
- **Preview image:** 1200×630px, hosted at the domain root (e.g. `/og-image.png`). A clean screenshot of the splash/role-select with the title and a one-line strapline overlaid. Must be an absolute URL for LinkedIn to fetch it.
- Suggested copy:
  - Title: `What a Disaster — Multi-Agency Emergency Exercise Simulator`
  - Description: `Play a Gold/Silver/Bronze command role in a live emergency. Make the calls, see the consequences. A free JESIP-aligned training exercise you can run in 15 minutes.`
- **Validate** with LinkedIn Post Inspector (linkedin.com/post-inspector) and the OG debuggers before posting — LinkedIn aggressively caches, so get it right *before* the first share.

### 1.2 Better `<title>` + meta description
- `<title>What a Disaster — Multi-Agency Emergency Exercise Simulator</title>`
- Add `<meta name="description">` matching the OG description (helps search + some link unfurls).

### 1.3 Cloudflare Web Analytics
- Enable in the Cloudflare dashboard for whatadisaster.uk; paste the provided beacon snippet before `</body>`.
- Cookieless, GDPR-friendly, **no consent banner required** — aligns with VISION's "frictionless first" principle and suits the public-sector audience.
- Gives: visits, page views, referrers (you'll see the LinkedIn spike by source), countries, devices. **Not** custom events — that's Stage 2.
- Turn this on *first* so the spike is captured from the moment the post goes live.

### 1.4 Prominent Feedback button (Google Form interim)
- Persistent, low-profile "Feedback" affordance:
  - A fixed corner button on all screens, **and**
  - A clear call-to-action on the `#results` / `#debrief` screens — the point where a player has just formed an opinion and is most likely to respond.
- Interim target: a **Google Form** (works on locked-down devices where mailto fails; zero backend). Fields: role played, scenario, free-text, optional org/name.
- Keep the existing mailto/`submitToEPT()` for now; the Feedback button is additive.
- **Migration note:** when Stage 2 lands, repoint the button at the in-app Firebase form so submissions land in the `feedback` collection.

### 1.5 "More scenarios coming" signal (optional, cheap)
- A small line near the scenario picker: e.g. "More scenarios in development — request one via Feedback." Signals *platform*, not two fixed exercises — worth doing while lots of new people are looking. (VISION Tier 1.)

### 1.6 Mobile pass
- Load on a real phone; check splash, role grid, question layout, consequence bars, results. Fix any overflow/cramping. LinkedIn traffic is mobile-heavy.

**Stage 1 definition of done:** LinkedIn Post Inspector shows a correct card with image; Cloudflare Analytics is recording; Feedback button visible and working on desktop + mobile; DEVLOG + daily log updated.

---

## Stage 2 — Firebase (the follow-up the moment justifies)

This is the already-scoped work in `DEVLOG.md` ("Planned: Feedback & Release Tracking"). The LinkedIn traffic is the *reason* to build it now, because it does double duty.

### 2.1 What it delivers (two things, one project)
1. **Structured feedback DB** — replaces the broken mailto. In-app form writes to the `feedback` Firestore collection (`source: 'community'`), with the fields already specified in DEVLOG. Repoint the Stage 1 Feedback button here.
2. **Custom usage events — the "what's used vs not" metrics you actually want:**
   - `scenario_started` (which scenario), `role_selected` (which role), `question_answered` (index + quality), `scenario_completed`, `feedback_submitted`.
   - This answers: which roles are popular, which scenario gets played, **where people drop off**, completion rate — none of which a pure page-view tracker can give you.

### 2.2 Analytics approach decision (make at build time)
| Option | Custom events | Cookies / consent | Notes |
|---|---|---|---|
| **Firebase / GA4** | ✅ rich | ⚠️ Google cookies; may need consent notice | Same project as the feedback DB; least code |
| **Cloudflare Pages Function + KV/D1** | ✅ (custom) | ✅ cookieless, in-house | Preserves inspectable/privacy property; more custom work |
| Firestore-only event writes | ✅ | ✅ no GA cookies | Log events as documents; you own the data, simple, no GA layer |

**Recommendation:** Firestore-only event writes (option 3) — keeps everything cookieless and in the one Firebase project you're building for feedback anyway, avoids the GA4 consent-banner question, and keeps the "frictionless first" property. Revisit GA4 only if you want out-of-the-box dashboards.

### 2.3 Rate limiting / abuse
- Community feedback form: rate-limit writes (per DEVLOG open question). No auth. Consider a simple honeypot + client throttle; Firestore security rules to cap write shape/size.

### 2.4 Build order (from DEVLOG, unchanged)
1. Create Firebase project, enable Firestore.
2. Add Firebase SDK via CDN in `index.html` (no build step).
3. Community feedback write path first (repoint Stage 1 button).
4. Event logging second.
5. Admin read UI separately (auth-gated HTML page) — later.

**Stage 2 definition of done:** feedback writes land in Firestore; usage events recorded; Stage 1 Google Form retired or kept as fallback; DEVLOG milestone added.

---

## Open decisions for Ben
1. **OG image** — happy for me to generate a screenshot-based 1200×630 card, or do you want to design one? (I can produce a first draft.)
2. **Feedback form** — OK to stand up a Google Form for the interim, or wait and go straight to Firebase? (Interim recommended — the LinkedIn spike is a one-time feedback opportunity.)
3. **Analytics privacy stance** — confirm the preference for cookieless/no-consent-banner (Cloudflare + Firestore-only events) over GA4. Recommended given the audience.
4. **Timing** — is the LinkedIn post imminent (do Stage 1 today) or is there runway to fold in some of Stage 2 first?

## Explicitly NOT doing (yet)
- Accounts / login (VISION: friction with no payoff here).
- GA4 with a consent banner, unless a specific need for its dashboards emerges.
- Splitting analytics across two live systems long-term — fine short-term (Cloudflare for traffic + Firestore for events), but don't add a third.

---

## Cross-references
- `VISION.md` → Tier 1 (mailto replacement, scenario-picker signal), "frictionless first" principle.
- `DEVLOG.md` → "Planned: Feedback & Release Tracking (Firebase — setup pending)" — Stage 2 is this.
- `info/stakeholder-feedback-june-2026.md` → separate track (timer UX + content-accuracy fixes); not blocked by this spec, can run in parallel.

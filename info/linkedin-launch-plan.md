# LinkedIn Launch — Execution Plan

_Created: 2026-07-16 · Owner: Ben · Branch: `claude/linkedin-publishing-approval-xfx0hv`_

**Status:** Plan drafted, awaiting Ben's go-ahead to apply changes to `index.html` / `disclaimer.html`. All copy below is ready-to-paste.

**Purpose:** Get the app ready for Heather to publish on LinkedIn — correct the now-inaccurate privacy statements, surface the proper name, make the shared link render as a real preview card, and give Heather a post she can use.

---

## TL;DR — what happens and why

Heather is about to share `whatadisaster.uk` on LinkedIn (a formal, professional audience). Before that:

1. **Fix the privacy statements** — they currently say "no analytics / no data leaves your device", which is no longer true (we added Cloudflare Analytics, anonymous usage events, and the feedback widget). **This is the real blocker** — publishing a false privacy claim to a data-protection-aware public-sector audience is the one thing that could actually bite.
2. **Surface the name** — adopt **"What a Disaster — Multi-Agency Emergency Exercise Simulator"** as the title, plus the strapline. The name isn't even on the splash today.
3. **Social preview** — add Open Graph / Twitter tags + a preview image so the link unfurls as a proper card, not a bare blue link. Get this right *before* the first share (LinkedIn caches hard).
4. **Give Heather a post** — pre-written, correctly framed on JESIP (aligned, not accredited).

---

## Locked decisions (from the discussion)

| Decision | Choice |
|---|---|
| Keep the name "What a Disaster"? | **Yes** — memorable, already shared organically. Let the strapline carry the seriousness. |
| Full title | **What a Disaster — Multi-Agency Emergency Exercise Simulator** |
| Strapline | _Play a Gold/Silver/Bronze command role in a live emergency. Make the calls, see the consequences. A free, JESIP-aligned training exercise you can run in 15 minutes._ |
| Domain | Unchanged — `whatadisaster.uk` |
| JESIP posture | **No consultation required** — independent tool referring to public doctrine. Never claim "official/accredited/endorsed". Disclaimer already draws the line correctly; keep post copy inside it. |
| JESIP courtesy heads-up | Optional, upside-only — not a blocker. |
| PWA / offline | **Deferred.** Not needed for a link share. Service-worker caching is a liability while iterating. Optional manifest+icons (no SW) can come later. |

---

## Work items (in order)

### 1. ⚠️ Correct the privacy statements — BLOCKER · `disclaimer.html` · Claude
The app now has Cloudflare Web Analytics (cookieless), anonymous Firestore usage events, and the feedback widget (POSTs name/email/message *only* on voluntary submit). The disclaimer must stop claiming otherwise.

**Replace the "Data & Privacy" section (`disclaimer.html`, currently ~line 99–100):**

> **Data & Privacy**
>
> This tool does not require an account or log-in, and it uses no tracking cookies or cross-site advertising trackers.
>
> To understand how the tool is used and improve it, we collect a small amount of **anonymous** data:
> - **Aggregate traffic analytics** via Cloudflare Web Analytics — cookieless page-view and referrer counts. No personal data, no cookies, no consent banner required.
> - **Anonymous usage events** — which role and scenario are chosen, question progress, and completion — so we can see which content is used and where people drop off. These contain no personal or identifying information.
>
> The **only** time you send us personal information is if you **choose** to submit feedback: the feedback form asks for a name, email and message, sent to our feedback service so we can read and respond. Feedback is entirely optional.
>
> Your exercise results and decision log are stored in your browser's own local storage so you can review your debrief — these stay on your device and are not transmitted.

**Replace the FAQ answer "What data does it collect about me?" (currently ~line 155–156):**

> No account, no log-in, and no tracking cookies. We collect **anonymous, aggregate** analytics (cookieless page views via Cloudflare) and **anonymous** usage events (which role/scenario you play, your progress and completion) to see what's used and improve the tool — none of it identifies you. Your decision log and results are saved in your browser's local storage and never leave your device. The only personal data we ever receive is what you **voluntarily** type into the feedback form (name, email, message), and only if you choose to send it.

_Note: the "Does it work offline?" FAQ (mentions the Google Fonts dependency) stays accurate and needs no change._

### 2. Name + title update · `index.html`, `how-it-works.html` · Claude (splash markup → Ben to eyeball)

**`index.html` `<title>` (line 6):**
```html
<title>What a Disaster — Multi-Agency Emergency Exercise Simulator</title>
```

**`index.html` splash `<h1>` (line 236) — surfaces the brand for the first time.** Current: `<h1 class="st">Emergency<br><span>Exercise Simulator</span></h1>`. Proposed:
```html
<h1 class="st">What a <span>Disaster</span></h1>
<div class="ssub">Multi-Agency Emergency Exercise Simulator</div>
```
- Puts "Disaster" in the fire-orange accent (`<span>`) — on-brand.
- `.ssub` is a small new style (condensed, dimmed, letter-spaced) — needs a one-line CSS add; exact size/spacing finalised at implementation. **Ben to eyeball the look before ship.**
- The existing badge (`Multi-Agency · Bristol · JESIP · Off-the-Shelf Exercise`) and intro card can stay.

**`how-it-works.html` `<title>` (line 6):**
```html
<title>How It Works — What a Disaster</title>
```

### 3. Social preview / OG tags · `index.html` + `how-it-works.html` `<head>` · Claude
Add to `index.html` `<head>` (additive markup, no logic):
```html
<meta name="description" content="Play a Gold/Silver/Bronze command role in a live multi-agency emergency. Make the calls, see the consequences. A free, JESIP-aligned training exercise you can run in 15 minutes.">
<meta property="og:type" content="website">
<meta property="og:url" content="https://whatadisaster.uk">
<meta property="og:site_name" content="What a Disaster">
<meta property="og:title" content="What a Disaster — Multi-Agency Emergency Exercise Simulator">
<meta property="og:description" content="Play a Gold/Silver/Bronze command role in a live emergency. Make the calls, see the consequences. A free, JESIP-aligned training exercise you can run in 15 minutes.">
<meta property="og:image" content="https://whatadisaster.uk/og-image.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="What a Disaster — Multi-Agency Emergency Exercise Simulator">
<meta name="twitter:description" content="Play a command role in a live multi-agency emergency. Make the calls, see the consequences. Free, JESIP-aligned, ~15 minutes.">
<meta name="twitter:image" content="https://whatadisaster.uk/og-image.png">
```
For `how-it-works.html`, the same block with `og:url` = `https://whatadisaster.uk/how-it-works.html` and `og:title`/`twitter:title` = `How It Works — What a Disaster`. Same image.

### 4. OG preview image · **Ben** (Claude can draft)
- 1200×630 PNG, committed to repo root as `og-image.png` (Cloudflare Pages serves it at `https://whatadisaster.uk/og-image.png`).
- Content: clean screenshot of the splash/role-select with **"What a Disaster"** + the strapline overlaid.
- Must be an absolute URL (already is above) for LinkedIn to fetch it.
- **Ben to approve the visual.** Claude can produce a first draft.

### 5. Deploy, prime & validate the card · Ben + Claude
1. Merge the branch to `main` → Cloudflare Pages deploys.
2. Run `https://whatadisaster.uk` through **LinkedIn Post Inspector** (linkedin.com/post-inspector) to force-refresh and confirm the card (title + description + image) — **before** Heather posts. LinkedIn caches the first fetch aggressively.
3. Spot-check the unfurl in Slack/Teams/WhatsApp too (same OG tags).

### 6. Optional, deferred — app-install polish (no service worker)
Manifest + `apple-touch-icon` + `theme-color` so "Add to Home Screen" gives a proper icon/name. **Not** a launch blocker, **no** offline/service-worker (avoids stale-cache pain while iterating). Revisit only if offline on locked-down devices becomes a real requirement — note the disclaimer's "works offline" claim is currently aspirational (no service worker exists).

---

## JESIP & agency framing — guardrails for all public copy

**Safe (use freely):** "JESIP-aligned", "consistent with JESIP principles", "based on the Joint Decision Model", "independent training tool".

**Do NOT use:** "official", "accredited", "endorsed by JESIP", "JESIP training", the JESIP logo, or anything implying a partnership. Same for named agencies (Bristol City Council, Avon Fire, etc.) — reference for realism, never imply endorsement.

Why: the app is an independent piece of work *referring to publicly-published doctrine* — that needs no permission. The disclaimer already states it's not accredited/endorsed. The only real exposure is over-claiming in the post, or a content inaccuracy being challenged publicly. Content-accuracy fixes flagged by a real EP professional (Clare) are already in; keep the post inside the disclaimer's claims and you're clear.

---

## Heather's LinkedIn post (draft — Ben/Heather to tweak voice)

### Primary version
> **What would you do if you were in command?**
>
> Over the past few months I've been helping design and validate a free training tool that drops you into a Gold, Silver or Bronze command role during a live multi-agency emergency — a wildfire, or a severe heatwave.
>
> You make the real decisions a commander faces, under time pressure, and watch the consequences play out across life safety, infrastructure, communications and inter-agency coordination. After every call there's a short debrief explaining the JESIP-aligned reasoning behind the best option.
>
> It won't replace accredited training or a live exercise — it's a ~15-minute, off-the-shelf way to test your understanding, see how the other agencies' roles fit together, and spark discussion before a tabletop.
>
> No login, no install — just a link that works on a locked-down browser:
> 👉 https://whatadisaster.uk
>
> It's an independent, non-commercial tool and we're actively improving it. If you're an emergency planner or multi-agency responder, I'd genuinely value your feedback (there's a button built in) — and if there's a scenario your area needs, tell us.
>
> #EmergencyPlanning #Resilience #JESIP #MultiAgency #CivilContingencies

### Short version
> New free training tool for multi-agency responders: take a **Gold/Silver/Bronze command role** in a live wildfire or heatwave, make the real decisions under pressure, and see the consequences — with a JESIP-aligned debrief after every call.
>
> ~15 minutes, no login, works on a locked-down browser. Independent and non-commercial — I helped design and validate the scenarios and would value your feedback.
>
> 👉 https://whatadisaster.uk
>
> #EmergencyPlanning #Resilience #JESIP #MultiAgency

**Notes for Heather:**
- Trim hashtags to taste (3–5 is the LinkedIn sweet spot).
- Posting a native image/screenshot alongside the link tends to get more reach than a bare link — but paste the URL *after* the card is validated (item 5) so the preview is correct.
- Her "helped design and validate" framing is accurate (she's credited in-app) and lends credibility — keep it.

---

## Definition of done
- [ ] Privacy statements corrected in `disclaimer.html` (Data & Privacy + FAQ).
- [ ] Title + splash name updated; `how-it-works.html` title updated. Splash look approved by Ben.
- [ ] OG/Twitter tags added to both pages.
- [ ] `og-image.png` (1200×630) approved and committed to repo root.
- [ ] Branch merged to `main`, deployed to `whatadisaster.uk`.
- [ ] LinkedIn Post Inspector shows the correct card.
- [ ] Heather has final post copy.
- [ ] DEVLOG milestone + daily log updated.

## Open items for Ben
1. **Approve the splash name treatment** ("What a Disaster" with "Disaster" in fire-orange + strapline subtitle) or ask for a variant.
2. **OG image** — happy for Claude to draft the 1200×630 card, or design your own?
3. **JESIP courtesy heads-up** — want to send one before/after Heather posts, or skip?
4. **Timing** — apply changes now and merge, or hold?

## Cross-references
- `info/linkedin-launch-readiness-spec.md` — the earlier readiness spec (analytics/feedback/social preview). This plan executes its Stage-1 social-preview items + the name + the privacy correction.
- `disclaimer.html` — current (now partly inaccurate) privacy + JESIP framing.
- `DEVLOG.md` — analytics/events/feedback milestones that made the old privacy copy inaccurate.

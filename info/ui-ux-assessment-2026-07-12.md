# UI / UX Assessment & Build Spec — What a Disaster

_Created: 2026-07-12 · Author: Claude Code · Branch: `claude/whatadisaster-ui-ux-assessment-xwrslb`_

**Trigger:** Ben's concern that the UI/UX "may be a bit busy." This document is a full review of `index.html` (and `how-it-works.html`), with a focus on **mobile (iOS / Android)** since most players will be on phones. It is a spec to be **built later** — nothing here has been implemented yet.

**Method:** Static read of `index.html` (~2,600 lines, single file) + live rendering at an iPhone-class viewport (390×844, DPR 2) via headless Chromium. Screenshots of the splash, role-select, and in-game screens informed the density findings. References are `file:line` against the current `index.html`.

---

## TL;DR — the verdict

**Yes, it's busy — but the busyness is concentrated in two screens, and it's fixable without a redesign.**

- The **splash screen** front-loads ~1,500px of acronym-heavy framing (WCS, RWCS, SCG, JESIP, BC) *above* the actual play buttons. On a phone the primary action is below the fold on first load.
- The **game screen** is ~2,160px tall per question on a 390px phone — roughly 2.6 screenfuls. The decision (the whole point) sits in the middle, wrapped in six persistent panels, two of which (Activated Agencies, Command Level) live **below** the answer options and are almost never seen during a decision. Every panel competes for attention with saturated colour, uppercase condensed type, and animated/pulsing elements.
- Underneath the density are a set of **concrete "bad behaviours"**: pinch-zoom disabled (accessibility fail), duplicated DOM blocks with clashing IDs, a modal that advances the game when you tap the backdrop by accident, no way to quit mid-exercise, and role cards that aren't real buttons.

The role-select screen, by contrast, is clean and works well — it's the model the other screens should move toward.

Priority order: **P0 accessibility/correctness → P1 game-screen density → P2 splash simplification → P3 polish.**

---

## Severity legend

| Tag | Meaning |
|-----|---------|
| **P0** | Correctness / accessibility defect. Fix regardless of the "busy" question. |
| **P1** | Core of the "too busy" problem. High player-facing impact on mobile. |
| **P2** | Meaningful density / clarity improvement. |
| **P3** | Polish / nice-to-have. |
| **Effort** | S = <1h · M = a few hours · L = half-day+ |

---

## A. P0 — Correctness & accessibility bad behaviours

These are defects independent of the aesthetic question. Do them first; several are quick.

### A1. Pinch-zoom is disabled — WCAG 1.4.4 fail · Effort S
`index.html:5` and `how-it-works.html:5`:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```
`maximum-scale=1.0, user-scalable=no` blocks pinch-to-zoom on both pages. This is an accessibility violation (WCAG 2.1 SC 1.4.4 Resize Text) and actively hostile on mobile — some of the on-screen text is 0.58–0.72rem (see A-block below), exactly the content a low-vision user would need to zoom. It also can't prevent iOS Safari zoom reliably anyway.
- **Fix:** `<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">`. Drop `maximum-scale` and `user-scalable`. `viewport-fit=cover` also lets us respect the notch/home-indicator safe areas (see A6).
- **Acceptance:** pinch-zoom works on iOS Safari and Android Chrome; layout unchanged at default zoom.

### A2. Duplicated DOM with clashing IDs — invalid HTML, dead weight · Effort M
The body contains **two full copies** of several screens, a merge/copy-paste artifact:

| Block | Live (used) | Dead duplicate |
|---|---|---|
| `#game` | `index.html:350` | `index.html:488` |
| `#results` | `index.html:397` | `index.html:535` |
| `#modal` | `index.html:423` | `index.html:561` |
| `#fireMap` | `index.html:266` | `index.html:435` |

Every interior ID (`scoreEl`, `cLife`, `pdots`, `fire1`–`fire5`, `emb1`–`emb3`, `mBody`, …) therefore exists **twice**. `document.getElementById()` returns the first match, so the app happens to work by always hitting the first (live) copy — but this is invalid HTML, confuses assistive tech and any future querySelector work, and the dead copies inflate the file materially (the file is 393 KB). The two dead `#results` blocks even differ (the live one at `:397` has a "Review Decisions" button; the dead one at `:535` doesn't), which is a latent trap for anyone who edits the wrong copy.
- **Fix:** delete the dead duplicates (`#game` `:488`, `#results` `:535`, `#modal` `:561`, `#fireMap` `:435`) after confirming — via a diff — that the first copy of each is the one the JS targets (it is: `showScreen`/`showFireMap`/`loadQ`/`pick` all resolve to the first). Re-test every screen transition end-to-end after removal.
- **Acceptance:** exactly one element per ID (`grep -c 'id="game"'` → 1, etc.); full playthrough of both scenarios still renders game, fire-map, modal, results, debrief correctly.

### A3. Modal advances the game on an accidental backdrop tap — destructive · Effort S
`index.html:423` (and dupe `:561`): `<div id="modal" ... onclick="closeModal()">`, and `closeModal()` (`:2473`) does `G.qIdx++` then loads the next question. So tapping anywhere on the dimmed backdrop **both dismisses the feedback the player is meant to read and irreversibly advances** to the next question — no undo, no way back. On a phone the backdrop is a large easy-to-mis-tap target above the bottom-sheet card.
- **Fix:** decouple dismissal from progression. The backdrop `onclick` should do nothing (or a gentle "tap Continue to proceed" nudge); only the explicit **Continue →** button advances. Keep `event.stopPropagation()` on the card.
- **Acceptance:** tapping the backdrop does not change `G.qIdx`; only the Continue button advances.

### A4. No way to quit / restart mid-exercise — a trap · Effort S
Once in `#game` there is no back, quit, or restart control anywhere (`.gh` header `:351` has role/score/timer only). The player is committed to answering all 6–8 questions to reach results, or must hard-reload the page (losing everything). On mobile, where people dip in and out, this is a real trap — and it inflates drop-off, which we're now measuring.
- **Fix:** add a small, unobtrusive **Exit** affordance to the game header (e.g. a ✕ or "End" at the far right) that opens a lightweight confirm ("End exercise and return to menu?") → `restart()` / `showScreen('roleSelect')`. Must be visually quiet so it doesn't add to the busyness.
- **Acceptance:** from any question, the player can return to role-select in ≤2 taps with a confirm step.

### A5. Role cards aren't real buttons — keyboard/AT inaccessible · Effort S
`buildRoles()` (`:2288`) emits role cards as `<div class="rc" onclick="selectRole(...)">`. They're not focusable, not keyboard-activatable, and expose no button role to screen readers. The same applies to the agency/command chips (decorative, fine) but the role cards are primary controls.
- **Fix:** render role cards as `<button>` (or add `role="button"`, `tabindex="0"`, and Enter/Space handling). Add an `aria-label` combining role name + level. Give a visible `:focus-visible` outline (currently only `:active` is styled, `:39`).
- **Acceptance:** role cards are Tab-reachable and Enter/Space-activatable; each announces name + command level.

### A6. Safe-area insets ignored — content under the notch/home bar · Effort S
Sticky game header (`.gh` `position:sticky;top:0` `:66`) and the bottom-sheet modal (`.md` `:158`) don't account for iOS safe areas. With `viewport-fit=cover` (A1) the header can sit under the status bar/notch and the modal's Continue button can sit under the home indicator.
- **Fix:** add `env(safe-area-inset-*)` padding — e.g. `.gh{padding-top:calc(.75rem + env(safe-area-inset-top))}` and bottom padding on the modal and any bottom-fixed CTA.
- **Acceptance:** on an iPhone with a notch/Dynamic Island, header text and modal button are fully visible and tappable.

---

## B. P1 — The "too busy" game screen

This is the heart of Ben's concern. The game screen stacks **six persistent panels** around every question:

1. Header — role · score · elapsed timer (`:351`)
2. Progress dots (`:356`)
3. **Consequence Tracker** — 4 meters, Life/Infra/Community/Coordination (`:357`)
4. **Situation panel** — phase badge, title, paragraph, up to 4+ stat chips (`:366`)
5. Escalation banner (conditional, pulsing) (`:371`) + pressure timer (conditional) (`:372`)
6. The **decision** — question stem + option buttons (`:376`)
7. **Activated Agencies** — 8 chips (`:377`)
8. **Command Level** — Gold/Silver/Bronze (`:386`)

**Measured:** at 390×844 the screen is **~2,160px tall** — the player sees, above the fold, only the header + tracker + situation + question stem + the first ~1.5 options. To read all options they scroll, which pushes the situation text out of view — so they're holding the scenario in working memory while comparing three multi-line options. Panels 7 and 8 sit **below** the options, so in practice they're passive noise the player scrolls past *after* deciding, if at all.

Compounding the density: nearly every panel uses saturated colour fills, uppercase Barlow Condensed, and motion. Three things pulse/animate simultaneously in some states — the "DECIDE NOW" label (`up` keyframe `:92`), the escalation banner glow (`ep` `:118`), and the urgent question label (`:101`). Colour is doing double duty everywhere (red = danger chip, red = wrong answer, red = urgent timer), so nothing reads as the priority.

### B1. Establish a visual hierarchy — decision first · Effort M–L · **highest leverage**
The single most effective change: make the **decision** the visual and spatial focus, and demote everything else to supporting context.

Recommended structure (top → bottom):
1. **Slim sticky status bar** (replaces the current header + full consequence tracker): role · score · elapsed on the left; a single compact **consequence indicator** on the right (one dot/among-4 mini-bar cluster, or a single "Situation: Controlled/Strained/Critical" pill derived from `updateConsBars()`'s existing level logic). Tappable to expand the full 4-meter breakdown in a sheet for anyone who wants detail.
2. **Situation** — keep, but make it collapsible after first read (see B3).
3. **The decision** — question + options, given the most vertical room and the calmest container so it's unmistakably the focus.
4. **Agencies + Command Level** — move into a collapsed "Context" drawer / accordion below the fold, or into the same expandable sheet as the consequence detail. They're reference state, not decision inputs at the moment of choosing.

Net effect: the option buttons should be reachable with **at most one short scroll**, and the situation text should remain glanceable while choosing (see B3).

### B2. Calm the colour and motion · Effort S–M
- Reduce simultaneous animation to **at most one** attention-grabbing motion at a time. The pressure timer bar is the legitimate one during timed questions; suppress the escalation-banner glow and the pulsing label when a timer is active.
- Reserve **red** for genuine urgency (timer, wrong-answer reveal). Recolour the "danger" situation chips (`.chip.r` `:85`) to a less alarming treatment (e.g. amber/neutral with an icon) so red isn't ambient.
- The four consequence bars are currently 5px tall with 0.58rem labels (`.ctb` `:76`, `.ctl` `:75`) — near-unreadable and yet visually noisy. Either enlarge them into the expandable detail sheet (B1) or replace the always-on version with the single pill.
- **Acceptance:** on a timed question, exactly one element animates; on an untimed question, none pulse.

### B3. Make the situation persist without dominating · Effort M
Because options push the situation off-screen, players lose the scenario while deciding. Options:
- Make the situation paragraph **collapsible** — full text on load, then a one-line summary the player can tap to re-expand, so it stays near the top while options fill the screen; **or**
- Keep a compact **one-line scenario recap** pinned just above the options.
- **Acceptance:** while the option buttons are on screen, the player can see (or one-tap reveal) the scenario without scrolling back up.

### B4. Tighten the stat chips · Effort S
The situation exposes up to 4 base chips plus branch-pressure chips (`getBranchPressure()` `:2359`), each a coloured pill. In WCS states this can be 6+ pills wrapping to three rows. Cap the visible count (e.g. 3 most salient) with a "+N more" affordance, or group them. Reduces the top-of-screen noise before the decision.

### B5. Reduce per-question scroll length · Effort S (falls out of B1)
Target: the full game screen for a typical question should be **≤1.6 screenfuls** (down from ~2.6) at 390×844, achieved by moving Agencies + Command Level into the drawer (B1) and collapsing the situation (B3).

---

## C. P2 — Splash screen density

The splash (`:216`–`:251`) is ~2,900px tall on mobile. Order today: badge → big title → subtitle → "No prior knowledge needed" card → **Exercise Framing** (Strategic WCS / Tactical RWCS) → **During the Exercise, Consider** (three dense paragraphs: Business Continuity, Response Plan, National Guidance) → *then* the two scenario buttons → briefing/how-it-works/disclaimer links. So a first-time mobile visitor scrolls past five text blocks and ~6 acronyms before reaching a play button.

### C1. Move the primary action above the fold · Effort M
- Put the **two scenario buttons** (and a single one-line strapline) near the top, immediately after the title — the play action should be visible without scrolling on a 390×844 phone.
- Demote **Exercise Framing** and **During the Exercise, Consider** into a collapsible "Before you start / For facilitators" accordion, or move them onto the existing briefing screen (they're facilitator/context content, not needed to press play). This is exactly the kind of material `how-it-works.html` and the briefing already exist to hold.
- **Acceptance:** on first load at 390×844, at least one scenario CTA is fully visible without scrolling.

### C2. De-jargon the first impression · Effort S
The first paragraphs lead with WCS, RWCS, SCG, JESIP, BC. Keep the framing available (accordion/briefing) but the *first* thing a lay visitor sees should be plain: what it is, that it's free, that it takes ~15 minutes. Reuse the launch-spec strapline: _"Play a Gold/Silver/Bronze command role in a live emergency. Make the calls, see the consequences."_ (`info/linkedin-launch-readiness-spec.md`).

### C3. Fix stale role-count copy · Effort S (content)
Splash says "across **8 roles**" (`:222`) and the badge/subtitle reference 8, but there are now **9 roles** (Voluntary Agency Coordinator was added 2026-06-25 — `strategic, tactical, fire, police, council, nhs, utilities, ambulance, voluntary`). Update to 9 (or "9 roles" / "nine perspectives"). Grep for other "8 roles" occurrences before shipping.

---

## D. P2 — Head / shareability / SEO (overlaps the launch spec)

Already scoped in `info/linkedin-launch-readiness-spec.md` §1.1; restating here because it's a UX-of-the-share issue and cheap:

### D1. `<title>` + Open Graph / Twitter tags · Effort S
`index.html:6` is the generic `Emergency Exercise Simulator`; there are **no** OG/Twitter tags. Shared on LinkedIn/WhatsApp it's a bare, image-less link — poor first impression exactly where mobile traffic originates. Add per the launch spec: better `<title>` (`What a Disaster — Multi-Agency Emergency Exercise Simulator`), `og:title/description/type/url/image`, `twitter:card=summary_large_image` + a 1200×630 image at an absolute URL. Add to `how-it-works.html` too. Validate with the LinkedIn Post Inspector before the first share (LinkedIn caches hard).

---

## E. P3 — Polish

- **E1. Timer duration (content, but UX-adjacent).** Click-to-reveal shipped 2026-07-12, but the countdowns themselves are still 10–12s (`timerSecs` values; `startPressTimer` `:2417`) for options that are 2–3 lines each. Stakeholders (Clare + Jon) flagged them as too short. Now that reveal is behind a button, consider bumping the shortest timers (10s → ~15s) so reading time isn't the bottleneck. Cheap, and directly addresses live feedback. _(Tracked separately in the stakeholder doc; noted here for completeness.)_
- **E2. `:active`-only feedback.** Buttons and cards style `:active` but not `:hover`/`:focus-visible`. Fine for pure touch, but add `:focus-visible` for keyboard/switch users (pairs with A5).
- **E3. Orphaned last role card.** 9 roles in a 2-col grid leaves Voluntary alone on the last row (harmless; a 1-col full-width treatment for the final card would tidy it).
- **E4. Emoji as sole signifier.** Command levels and agencies rely on emoji + colour; ensure text labels remain (they mostly do) so meaning survives emoji-rendering differences across Android/iOS.
- **E5. Reduced-motion.** Add `@media (prefers-reduced-motion: reduce)` to disable the ember drift, pulsing labels, and banner glow for users who opt out.

---

## Proposed build order (phased)

Each phase is independently shippable and testable.

**Phase 1 — Correctness & a11y (P0, ~half-day)**
- A1 viewport / pinch-zoom + safe-area (A6)
- A2 delete duplicated DOM blocks
- A3 modal backdrop no longer advances
- A4 in-game exit control
- A5 role cards as buttons + focus states

**Phase 2 — Game-screen density (P1, ~1 day)**
- B1 hierarchy: slim status bar + consequence pill; Agencies/Command → drawer
- B3 persistent/collapsible situation
- B2 calm colour & motion; B4 chip cap; B5 verify scroll target

**Phase 3 — Splash + share (P2, ~half-day)**
- C1 CTAs above the fold; framing → accordion/briefing
- C2 de-jargon lead; C3 role-count copy
- D1 title + OG/Twitter tags + image

**Phase 4 — Polish (P3, opportunistic)**
- E1–E5

---

## Acceptance criteria (whole effort)

- Pinch-zoom works on iOS Safari + Android Chrome; no `user-scalable=no`.
- Exactly one element per DOM id (no duplicate blocks); both scenarios fully playable start→results→debrief.
- Tapping the modal backdrop never advances the question.
- A player can exit to the menu mid-exercise in ≤2 taps.
- Role cards are keyboard-focusable and Enter/Space-activatable.
- Game screen for a typical question is ≤1.6 viewport-heights at 390×844, with the option buttons reachable in ≤1 short scroll and the scenario glanceable while choosing.
- At most one animated element at a time; none on an untimed question; all motion respects `prefers-reduced-motion`.
- Splash shows a scenario CTA without scrolling at 390×844; lead copy is jargon-free; role count reads 9.
- Shared link renders a titled card with image (LinkedIn Post Inspector passes).

---

## Non-goals / things to preserve

- **Keep the single-file, no-build, no-dependency architecture** (per `CLAUDE.md` / `VISION.md`). Everything here is achievable in-file with vanilla CSS/JS.
- **Keep the dark, fire-orange identity and the Barlow type system** — the goal is to *calm and prioritise* the existing look, not restyle it. Role-select is already the target calmness.
- **Don't touch question content/scoring** in this workstream — content-accuracy fixes and the utilities split are tracked separately in `info/stakeholder-feedback-june-2026.md`.
- **Preserve the consequence/agency/command information** — it's demoted into drawers, not deleted. Facilitators value it; it just shouldn't compete with the live decision.

---

## Verification plan (for whoever builds this)

1. Static server (`python3 -m http.server`) + headless Chromium (pre-installed at `/opt/pw-browsers`) at **390×844** (iPhone) and **360×800** (common Android) — screenshot splash, role-select, game (full-page), timed reveal, and modal; compare against the "before" shots in this branch's scratchpad.
2. Manual: full playthrough of **both** scenarios in a real mobile browser (iOS Safari + Android Chrome) — verify pinch-zoom, safe-area, exit flow, backdrop-tap safety, and that the decision is the focal point.
3. Re-run `grep -c 'id="game"'` etc. → 1 each.
4. LinkedIn Post Inspector on `https://whatadisaster.uk` for the share card.
5. `prefers-reduced-motion` on → confirm animations stop.

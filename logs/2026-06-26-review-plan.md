# App Review & Action Plan — 2026-06-26

Full review of `index.html`, `how-it-works.html`, `disclaimer.html` plus a code/consistency
pass. Findings are prioritised. Each item lists **evidence** (line refs at time of review)
and a **recommended fix**. Nothing in this doc has been changed yet — this is the "what may
need doing" plan.

Legend: 🔴 High · 🟠 Medium · 🟢 Low

---

## Progress (updated 2026-06-26)

**Implemented this session** (smoke-tested headless — full flow + all 3 phase maps work, no JS errors):
- ✅ 1.1 Deleted the duplicated screen block (~140 lines; all IDs now unique)
- ✅ 1.3 Removed dead `fireSvg` code
- ✅ 1.4 Re-enabled pinch-zoom (viewport meta)
- ✅ 2.1 "8 roles" → "9 roles" (splash + how-it-works)
- ✅ 2.2 Added the Voluntary Agency Coordinator card to how-it-works
- ✅ 2.3 In-game header "Op." → "Exercise"
- ✅ 2.4 Corrected the how-it-works timer copy (30–90s → ~10–25s); also fixed "three options" → "three or four"
- ✅ 2.6 Generalised the between-phases map description to both scenarios
- ✅ 3.1 `APP_VERSION` → 1.2.0 + added CHANGELOG 1.2.0 and Unreleased entries
- ✅ 3.2 Updated CLAUDE.md (9 roles, removed stale `submitToEPT`)

**Still open:** 1.2 (flag refactor), 2.5 (cabot rename), 4.1 (timed-question click-to-reveal — headline UX),
4.2 (ambulance surge framing), 4.3 (telegraphing), 4.4 (water/utilities coverage), 4.5 (pending HHA content).

---

## 1. Code bugs & correctness

### 🔴 1.1 Duplicated screen blocks (invalid duplicate IDs)
The markup for four screens is present **twice**: `#fireMap`, `#game`, `#results`, `#modal`
each appear at lines ~263–431 (current/correct set) **and again at ~432–569** (an older,
single-map variant). Duplicate `id`s are invalid HTML; `getElementById` only ever returns the
first, so the second set is ~140 lines of dead markup that ships to every user.
- The **first** set is the keeper: it has the two-scenario SVGs (`svgAshdown` / `svgCabot`)
  and the newer results buttons (`Review Decisions` + `Review Briefing` → `showBriefing()`).
- The **second** set is stale: single `fireMapSvg`, and a results screen whose "Review
  Briefing" calls `showScreen('briefing')` directly (no rebuild) and has no debrief button.
- Also duplicates dozens of inner IDs (`fire1‑5`, `emb1‑3`, `scoreEl`, `qContainer`, `mTitle`…).

**Fix:** delete the second block (≈ lines 432–569). Test the full flow afterwards
(splash → briefing → role → game → maps between phases → results → debrief → modal).

### 🟠 1.2 Fragile branching via title string-matching
`pick()` sets branching flags by substring-matching the question title:
- `noTCG`  ← title contains `TCG` **and** `phase==='PHASE 1'`
- `hospitalLate` ← title contains `Hospital Notification`
- `registerWithheld` ← title contains `Vulnerable Persons`  ← **matches 10 different
  questions** across multiple roles/phases
- `towerDelayed` ← title contains `Meridian House Early`

This couples game logic to display copy: any title edit silently breaks branching, and
`registerWithheld` fires far more broadly than intended. `getBranchPressure()` then only
surfaces some flags on a specific phase (e.g. `hospitalLate` only on PHASE 3,
`registerWithheld` only on PHASE 2), so several flags can be set but never shown.

**Fix:** add an explicit field to the option/question (e.g. `setFlag:'noTCG'` on the weak
option, or `flagOnPoor:true`) and have `pick()` read that instead of parsing titles. Then
audit that each pressure chip in `getBranchPressure()` can actually appear.

### 🟢 1.3 Dead code: `getElementById('fireSvg')`
`showFireMap()` (line ~2189) looks up `fireSvg`, which doesn't exist (the SVGs are
`svgAshdown` / `svgCabot`). The `--fire-fill` / `--ember-fill` block it guards never runs —
heatwave colours actually come from the `.cabot` CSS + SVG swap. Harmless but misleading.

**Fix:** remove the dead `fmsvg` block.

### 🟢 1.4 Accessibility: pinch-zoom disabled
All three pages set `maximum-scale=1.0, user-scalable=no` in the viewport meta. This blocks
zoom for low-vision users — a real concern for a tool aimed at a broad professional audience.

**Fix:** drop `maximum-scale` / `user-scalable=no` (or set `maximum-scale=5`).

---

## 2. Consistency issues

### 🔴 2.1 "8 roles" copy is now wrong — there are 9
A 9th role (`voluntary` — Voluntary Agency Coordinator) shipped in v1.2.0, but user-facing
copy still says 8:
- `index.html:220` splash subtitle — "across **8 roles**"
- `how-it-works.html:261` heading — "The **8 Roles**"
- `CLAUDE.md` — "ROLES — **8 roles**"

**Fix:** update all three to 9.

### 🔴 2.2 how-it-works.html is missing the 9th role card
The roles grid (`how-it-works.html` ~280–337) lists only 8 cards — the Voluntary Agency
Coordinator card is absent.

**Fix:** add the voluntary role card (🤝, green, Bronze, LRF Voluntary Sector) to match the app.

### 🟠 2.3 Scenario naming drift — "Op." vs "Exercise"
Everywhere user-facing uses **"Exercise Ashdown / Exercise Solstice"** (splash buttons,
briefing subtitle, how-it-works, disclaimer) **except the in-game header**, which still reads
**"Op. Ashdown / Op. Solstice"** (`selectRole()`, line ~2284, plus the hard-coded defaults at
lines ~350 and ~488). Internal docs (`CLAUDE.md`, `DEVLOG.md`, `docs/`) use a third term,
**"Operation"**.

**Fix:** pick one canonical user-facing term (recommend "Exercise X") and update the game
header to match. Decide whether docs should keep "Operation" or switch — and apply
consistently.

### 🟠 2.4 how-it-works timer claim contradicts reality
`how-it-works.html:249` says timed questions are "some are **30 seconds**, some are **90**."
Actual `timerSecs` values across the banks are **10–25s**. This both misleads players and
amplifies the stakeholder "timers too short" complaint (see §4).

**Fix:** correct the copy to match real values (and revisit once §4.1 is decided).

### 🟢 2.5 Legacy "cabot" naming for the heatwave scenario
The heatwave scenario (`hotwells`) still carries internal "Cabot" naming from an earlier
fuel-tanker concept: `svgCabot` id, `.cabot` CSS class, `#fireMap.cabot` rules, MAPS2 comment.
Functional but confusing tech debt.

**Fix (optional):** rename to a neutral/`hotwells`/`solstice` token, or leave a one-line code
comment explaining the legacy name.

### 🟢 2.6 how-it-works says maps are Ashdown-only
`how-it-works.html:207` describes the phase map as appearing "In Exercise Ashdown" — but
Solstice also shows phase maps (`MAPS2`).

**Fix:** generalise the wording to both scenarios.

---

## 3. Documentation drift (`CLAUDE.md` / changelog / version)

### 🟠 3.1 APP_VERSION stale
`APP_VERSION='1.1.1'` (line 597) while `CHANGELOG.md` is at 1.1.2 and the VAC role shipped as
"v1.2.0" (per DEVLOG). There is also **no 1.2.0 entry in CHANGELOG.md**.

**Fix:** bump `APP_VERSION` to `1.2.0` and add the matching CHANGELOG entry (new role,
question-bank rework, the content-accuracy fixes from stakeholder feedback).

### 🟠 3.2 CLAUDE.md describes features that no longer exist / are out of date
- Documents `submitToEPT()` (mailto to `emergencyplanning@bristol.gov.uk`) and a "mailto
  submission" milestone — **no such function or mailto exists in `index.html` anymore**.
- Lists the 8-role roster without `voluntary`.

**Fix:** update CLAUDE.md (roles list + count, remove/flag `submitToEPT`, reconcile the key
functions table).

---

## 4. Stakeholder feedback — status & remaining work
(Source: `info/stakeholder-feedback-june-2026.md`)

Already addressed in the question banks (verified this review):
- ✅ NWFC → **NRFC** (and framed as a Fire-commander decision, Gold confirms).
- ✅ Hospital MI Plan is the **hospital's** decision, not a TCG request.
- ✅ Crew safety in extreme heat now conditional on **no other symptomatic crew**.

Still outstanding:

### 🔴 4.1 Timed questions too short (raised by BOTH Clare and Jon)
The single most-repeated piece of feedback. Current behaviour: the countdown starts the moment
a timed question renders (`loadQ()` → `startPressTimer()` immediately). Suggested approaches:
- **Click-to-reveal** — show the question/situation first, then a "Reveal options & start
  timer" button that begins the countdown; **and/or**
- **Extend** the `timerSecs` values (currently 10–25s).

**Fix (recommended):** implement click-to-reveal for timed questions, then re-tune durations,
then fix the how-it-works copy (§2.4).

### 🟠 4.2 Ambulance surge framing
Clare: ambulance surge management is an **internal Ambulance** matter, not a TCG decision.
Worth a targeted pass over ambulance/NHS surge questions to confirm framing (some look already
correct — confirm none present surge as a TCG request).

### 🟠 4.3 Answer telegraphing
Correct options can read as longer / more detailed than distractors. Partly fixed for the VAC
bank already; consider a length/specificity balance pass across the older banks. Clare noted
this may be acceptable "by design" — decision needed.

### 🟠 4.4 Coverage gaps (Jon, Utilities)
- "Utilities" lumps energy + water together. Consider splitting, or at least adding
  water-specific content.
- **Water has no representation in Operation Solstice** despite heat stressing water supply.
  Jon offered examples — follow up.

### 🟢 4.5 Pending content (from briefing doc)
- Heat-Health Alert (HHA) levels deeper integration, Met Office Hazard Manager reference,
  UKHSA hot-weather guidance — partially present; expand when Heather's slides arrive.

---

## Suggested order of work
1. **Quick wins / low-risk:** §1.1 (delete duplicate block), §1.3, §2.1, §2.2, §3.1, §3.2,
   §2.4, §1.4. Mostly deletions/copy — high value, low regression risk.
2. **UX, scoped:** §4.1 click-to-reveal timer (the headline feedback item) + §2.3 header
   naming.
3. **Content passes:** §1.2 flag refactor, §4.2/§4.3 framing & telegraphing, §4.4 water/utilities.
4. **Backlog / awaiting input:** §2.5, §4.5.

> None of the above is implemented yet. Happy to take any subset and ship it on
> `claude/app-review-improvements-wn9kuy`.

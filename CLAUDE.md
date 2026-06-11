# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Emergency exercise simulator for multi-agency fire/heatwave response training. Two HTML files, no build system, no dependencies beyond Google Fonts.

- `index.html` — the main simulator (~2,400 lines)
- `how-it-works.html` — standalone explainer page linked from the splash screen

Edit HTML directly and open in a browser to test. Deploy by pushing to `main` (Cloudflare Pages → whatadisaster.uk). `whatadisaster.pages.dev` continues to serve the same app.

## Dev Log

Two-tier logging system — same pattern as BetaLog:

**`DEVLOG.md`** — milestone tracker. One entry per completed feature or significant step. Read this at the start of a new session.

**`logs/YYYY-MM-DD.md`** — daily work log. Granular: what was built, files changed, key decisions.

Rules:
- Update today's log file **as you go** — after each meaningful change, not at the end of the session
- Create the log file at the start of the day's work if it doesn't exist yet
- Only update `DEVLOG.md` when a milestone is complete
- At the start of any new session, read `DEVLOG.md` first, then the most recent log file

## Documentation Index

| File | Purpose |
|------|---------|
| `DEVLOG.md` | **CURRENT** — Milestone tracker, read first in any session |
| `VISION.md` | **CURRENT** — Product strategy, roadmap, principles |
| `CHANGELOG.md` | **CURRENT** — Version history |
| `docs/guides/whatadisaster_sdlc.md` | **CURRENT** — Dev → test → deploy workflow |
| `info/whatadisaster-changes-briefing.md` | **CURRENT** — Pending changes briefing for Claude Code |

## Scenarios

Two scenarios share the same engine but use separate question banks and SVG maps:

| Key | Name | Setting | Question bank | Map data |
|---|---|---|---|---|
| `ashdown` | Operation Ashdown | Wildfire, Brynmoor Hill | `QB` | `MAPS` |
| `hotwells` | Operation Solstice | Severe heatwave, Bristol | `QB2` | `MAPS2` |

`G.scenario` (`'ashdown'` or `'hotwells'`) switches which bank/maps are loaded via `getQs()` / `showFireMap()`.

## Screen flow

`showScreen(id)` shows/hides fullscreen `<section>` divs. The flow is:

```
#splash → #roleSelect → #briefing → #fireMap → #game → #results → #debrief
```

`#fireMap` is shown between phases via `checkAndShowMap()` / `showFireMap(idx, callback)`. After the player clicks **Continue**, `continueFromMap()` fires the callback and returns to `#game`.

## Global state (`G`)

All runtime state lives in one object, reset by `restart()`:

```js
G = {
  roleKey,           // key into ROLES
  score, qIdx,       // current score and question index
  answers,           // array of {title, pts, max, quality, timeout, ft}
  timerInt,          // exercise elapsed-time interval
  exTimer,           // elapsed seconds
  pressInt,          // pressure-timer interval (timed questions)
  cons: {life, infra, comm, coord},  // consequence meters, 0–100
  flags,             // {noTCG: bool, ...} — branching flags set by picks
  mapShown,          // {0: bool, 1: bool, 2: bool} — prevents double-showing maps
  shuffledOpts,      // randomised option order for current question
  scenario           // 'ashdown' | 'hotwells'
}
```

## Question bank structure

Each entry in `QB` / `QB2` is keyed by role (e.g. `QB.strategic`, `QB.tactical`, `QB.fire`). `getQs()` returns the correct array for the current role and scenario.

Question object fields:
- `phase`, `title`, `sit`, `stats`, `cmd`, `ag` — display metadata
- `q` — the question text
- `opts` — array of option objects: `{t, r, pts, best, cons, fi, ft}`
  - `pts` — 0/3/5/8/20 — fed into `G.score`
  - `cons` — `{life, infra, comm, coord}` deltas applied to `G.cons`
  - `best: true` — marks the correct option
  - `fi` — feedback icon class (`'pos'`/`'neg'`/`'neu'`)
  - `ft` — one-line flash text shown in modal
- `timed: true` / `timerSecs` — enables the pressure timer; timeout doubles all `cons` deltas
- `branch_sit`, `branch_flag` — alternate situation text shown when the named flag is set in `G.flags`
- `esc` — escalation panel shown above the options
- `fb` — post-answer feedback `{title, body, jesip}`

## Key functions

| Function | Purpose |
|---|---|
| `buildRoles()` | Renders the role selection grid from `ROLES` |
| `selectRole(key)` | Sets `G.roleKey`, resets state, navigates to briefing |
| `loadQ()` | Renders the current question from `getQs()[G.qIdx]` |
| `applyBranching(q)` | Swaps in `branch_sit` if the relevant flag is set |
| `pick(idx, timedOut)` | Processes a player choice: scores, applies `cons`, sets flags, shows modal |
| `startPressTimer(secs, q)` | Starts the countdown bar; calls `pick(-1, true)` on expiry |
| `updateConsBars()` | Redraws the four consequence bars and sets the level label |
| `checkAndShowMap(qIdx, cb)` | Shows a fire-map interstitial at phase boundaries |
| `showFireMap(idx, callback)` | Populates the `#fireMap` screen with `MAPS`/`MAPS2` data and animates SVG |
| `endGame()` | Calculates outcome via `getOutcome()`, renders `#results` |
| `showDebrief()` | Renders the full decision review in `#debrief` |
| `submitToEPT()` | Builds a mailto link to `emergencyplanning@bristol.gov.uk` with the exercise record |
| `restart()` | Partial reset (score/qIdx/answers) — keeps scenario, goes to role select |

## Scoring and outcome

- Max 20 pts per question; `SLABS` maps point thresholds to labels.
- `OUTCOMES` array defines five impact bands (Negligible → Catastrophic), evaluated by `getOutcome()` using `G.score`, average of `G.cons`, and `G.cons.life` specifically.
- Results and a summary are also saved to `localStorage` keyed by role.

## Data constants

- `ROLES` — 8 roles: `strategic`, `tactical`, `fire`, `police`, `council`, `nhs`, `utilities`, `ambulance`
- `SLABS` — score-to-label map
- `OUTCOMES` — five outcome bands
- `MAPS` / `MAPS2` — three phase-transition map objects per scenario, each with SVG element targets for fire blob/ember animation and a `fact` string

## Code style

- Minified CSS in `<style>`. Dark theme, fire-orange accent (`#FF4500`), Barlow / Barlow Condensed fonts.
- Vanilla JS, ES5 style (`var`, no modules, no classes). Data in object literals.
- CSS variables: `--fire`, `--gold`, `--silver`, `--bronze`, `--dark`, `--dark2`, `--dark3`, `--card`, `--text`, `--dim`, `--green`, `--red`, `--blue`, `--amber`.
- `--rc` is a per-role-card CSS variable set inline to the role's colour.
- `how-it-works.html` duplicates the design tokens and Barlow font; it has its own self-contained `<style>` and minimal JS (tab switching only).

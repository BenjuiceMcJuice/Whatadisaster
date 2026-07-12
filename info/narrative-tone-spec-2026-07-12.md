# Spec: dynamic narrative tone from consequence meters

Date: 2026-07-12
Status: **spec only — not yet built**
Depends on: `info/branching-review-2026-07-12.md` (the bug review this spec responds to)
Preserves: single-file/no-build architecture, ES5 vanilla JS, existing question content and scoring — no new authored question content required.

## 1. Goal

Make a bad decision (or a good one) visibly colour how the *rest* of the exercise reads — not just the score at the end — without hand-authoring bespoke rewrites for individual questions (the approach that broke down in the current `branch_sit` system; see the review doc). Two playthroughs of the same role should read differently if they went differently, in any combination of questions, not just the 4 specific moments the old system tried to cover.

**Non-goals:** no new options, no changed scoring, no per-question narrative authoring debt, no change to `QB`/`QB2` structure size. Both scenarios (`ashdown`, `hotwells`) get this for free since it's driven by state that already exists identically in both (`G.cons`).

## 2. Why meters, not flags

`G.cons.{life,infra,comm,coord}` already updates continuously and identically across every role and both scenarios (`pick()`, `index.html:2327`). It's the one piece of state that's always present, always current, and never role/scenario-specific — unlike the old flag system, which only worked if a specific named question existed in that exact role's list. Driving tone off the meters means the mechanism works everywhere automatically; only the fragment *text* needs writing, once, not per question.

## 3. Mechanism A — fix the existing flag system (cleanup, not a new feature)

Before adding anything new, retire the parts of the current system that can never fire, and keep the parts that already work:

- **Delete** the 4 dead `branch_sit`/`branch_flag` field pairs (`index.html:702,742,767,1041`) and the now-pointless `applyBranching()` wrapper (`index.html:2234-2237`) — revert `loadQ()` to reading `q.sit` directly. These were verified unreachable in every role/scenario (see review doc §2) and are pure dead weight.
- **Keep** the working parts: the `noTCG`/`hospitalLate`/`registerWithheld` escalation chips (`getBranchPressure()`, `index.html:2239-2246`) and the `noTCG` debrief callout (`index.html:2407`) — these already fire correctly and are cheap to leave in place.
- **Fix** `towerDelayed`'s chip condition, which is the one flag with zero effect anywhere: change its match from `q.title.indexOf('Meridian')>=0` (a title that never recurs in `tactical`'s own list) to `q.phase!=='PHASE 1'`, matching the pattern already used for `noTCG`.

Effort: small, mechanical, no new content. Do this first so the new mechanism (below) isn't fighting dead code.

## 4. Mechanism B — meter-threshold tone fragments (the main feature)

### 4.1 Trigger: reuse the existing severity bands

`updateConsBars()` (`index.html:2367-2380`) already computes `avg = (life+infra+comm+coord)/4` and bands it: **Controlled** (<20) / **Manageable** (<40) / **Significant** (<60) / **Serious** (<75) / **Critical** (≥75). Reuse these exact thresholds for narrative tone rather than inventing a second scale — the player already sees this label on the consequence bar, so the prose will agree with what they can see.

- **Controlled band → no fragment.** Every exercise starts here (meters init at 10); silence at low severity means nothing appears out of nowhere on Q1, with no special-casing needed for question index.
- **Manageable → no fragment** (keep it subtle; don't over-narrate a merely-average run).
- **Significant / Serious / Critical → one fragment appended**, picked as described below.

### 4.2 Which meter gets named

Within Significant+, identify the single worst meter (`Math.max` of the 4). If it's clearly ahead of the others (e.g. ≥10 points above the next-highest), name it specifically ("coordination between agencies is visibly straining"); otherwise use a generic multi-meter fragment ("the response is under serious strain on multiple fronts"). This avoids naming a meter that isn't actually the standout problem.

### 4.3 Content bank

New data object, `NARRATIVE_TONE`, near `OUTCOMES`/`SLABS`:

```js
const NARRATIVE_TONE = {
  bands: {
    significant: ['...', '...'],   // 2 generic variants
    serious:     ['...', '...'],
    critical:    ['...', '...']
  },
  meter: {
    life:   {significant:'...', serious:'...', critical:'...'},
    infra:  {significant:'...', serious:'...', critical:'...'},
    comm:   {significant:'...', serious:'...', critical:'...'},
    coord:  {significant:'...', serious:'...', critical:'...'}
  }
};
```

- 2 variants per generic band (picked at random per question load) so replays of the same role at similar severity don't read identically every time — `Math.random()` is fine here, this is live browser JS, not a workflow script.
- 1 clause per meter per band (12 total) — short, second-person-scene-setting, no new facts (must not contradict any specific question's own `sit` text): e.g. `coord`/critical → "Command structures are fraying — Bronze commanders are making calls without a shared picture." No role-specific or question-specific wording, so one bank covers all 9 roles × 2 scenarios.
- **Total new content: ~6 generic + 12 meter clauses = 18 short sentences, written once.** This is the entire authoring cost — no per-question work.

### 4.4 Integration point

New function `getNarrativeTone()`:
```js
function getNarrativeTone(){
  var avg=(G.cons.life+G.cons.infra+G.cons.comm+G.cons.coord)/4;
  var band = avg<40?null : avg<60?'significant' : avg<75?'serious' : 'critical';
  if(!band) return '';
  var keys=['life','infra','comm','coord'], worst=keys[0];
  for(var i=1;i<keys.length;i++) if(G.cons[keys[i]]>G.cons[worst]) worst=keys[i];
  var isStandout = G.cons[worst] - secondHighest(keys, worst) >= 10;
  var pick = isStandout ? NARRATIVE_TONE.meter[worst][band]
                        : randomOf(NARRATIVE_TONE.bands[band]);
  return ' ' + pick;
}
```
Called from `loadQ()` (`index.html:2253`), appended to the existing `sitText` after `q.sit` (not replacing it):
```js
document.getElementById('sitTitle')... // unchanged
document.getElementById('sitText').textContent = q.sit + getNarrativeTone();
```
This is strictly additive — every question keeps its authored `sit` text verbatim; the tone clause is a suffix computed fresh every time, so it can never desync from what actually happened in that specific playthrough.

## 5. Mechanism C — generalize the debrief callout

The existing `noTCG` debrief line (`index.html:2407`, `ll.push(...)`) is the right idea but hard-coded to one flag. Generalize it: at `endGame()`, after the existing flag-based lines, compute the same worst-meter logic as §4.2 on the **final** meter values and push one plain-language line naming the dimension the exercise struggled with most (skip if final avg is Controlled/Manageable — a clean run doesn't need a "here's what went wrong" line). Reuses the exact `ll.push()` pattern already in place; no new UI, no new screen.

## 6. What this does *not* change

- Scoring, options, correct answers — untouched.
- `branch_sit` mid-exercise chip pressure for the 3 flags that already work (§3) — kept, unrelated mechanism, still useful for the handful of specifically-named teaching moments.
- Question sequence/count — untouched.

## 7. Test plan (before calling it done)

1. Play the same role twice: once picking mostly-correct answers, once mostly-wrong. Confirm the situation text on matching question indices differs between the two runs (tone suffix present in the bad run, absent/lighter in the good run).
2. Confirm Q1 of every role never shows a tone suffix (meters start at Controlled).
3. Confirm the named-meter clause only appears when one meter is a clear outlier, and the generic clause appears when multiple meters are similarly bad.
4. Spot-check both `ashdown` and `hotwells`, at least 2 roles each.
5. Confirm no console errors and that `sitText` never renders literally `undefined`/`null` (guard `getNarrativeTone()` returning `''` cleanly).
6. Confirm the `towerDelayed` chip fix (§3) actually appears on a later `tactical` question after failing "Meridian House Early Warning".
7. Confirm the debrief line (§5) appears on a bad run and is absent on a clean run.

## 8. Post-implementation requirement

**Once built, write a standalone "how it works" reference doc** (separate from this build spec) documenting the shipped feature for future quick reference — the actual band thresholds used, the full fragment bank, the exact hook points in `index.html`, and how to add/edit tone fragments later without re-deriving the design from this planning doc. This mirrors how `info/ui-ux-assessment-2026-07-12.md` is the *plan* and the DEVLOG + code comments are the *record of what shipped* — this feature should get the same treatment, likely as `info/narrative-tone-how-it-works.md`, linked from the Documentation Index table in `CLAUDE.md` once it exists.

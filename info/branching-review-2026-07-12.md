# Branching Review — does a bad choice actually change anything?

Date: 2026-07-12
Scope: `index.html` — verified against the actual game logic (`pick()`, `applyBranching()`, `getBranchPressure()`, `getOutcome()`), not just the data model.

## Short answer

**Score and the four consequence meters (life/infra/comm/coord) always respond to your choice — that part is solid.** The separate "branching" layer — where a bad early call is supposed to rewrite later situation text — is **broken**. All four of its rewrite triggers are unreachable, so **no player, in any role, in any scenario, will ever see the alternate situation text it was built to show.** Two smaller, working side-effects (warning chips + one debrief line) partly cover for this, but only for 2 of the 4 branch points.

---

## 1. What definitely works, every time

Every option (`opts[]` entry) carries its own `pts` (score) and `cons` (meter deltas), and `pick()` applies both unconditionally the moment you answer:

```js
if(chosen.cons){ ...G.cons[k] += chosen.cons[k]*mult... }
G.score += pts;
```

- A bad choice always scores lower and always pushes `life`/`infra`/`comm`/`coord` in the worse direction (doubled if you let the timer run out).
- These meters directly decide your ending: `getOutcome()` bands the result on `G.score`, the average of the four meters, and `G.cons.life` specifically.

So the headline mechanic — "worse decisions produce a worse outcome" — is real and verified.

## 2. The separate branching layer (the thing that's supposed to make one bad call haunt you later)

On top of the above, four specific questions are wired to set a flag if you score below 12 on them:

| Flag | Set by (question title, phase) | Role/scenario it lives in |
|---|---|---|
| `noTCG` | "Establishing the TCG" — Phase 1 | `tactical` (both scenarios) |
| `towerDelayed` | "Meridian House Early Warning" — Phase 1 | `tactical` (Ashdown) |
| `hospitalLate` | "Hospital Notification" — Phase 2 | `tactical` (Ashdown) |
| `registerWithheld` | any question titled "Vulnerable Persons…" | varies (see §4) |

Once a flag is set, two things are *supposed* to happen:
1. A later question swaps its normal situation text for a grimmer `branch_sit` version.
2. A red warning chip appears on later questions via `getBranchPressure()`.

### Finding: the situation-text rewrite (`branch_sit`) never fires — for any of the 4 flags

`applyBranching()` only rewrites text if the flag was already set *before* that question loads. Checking where each flag's rewrite actually lives:

- **`towerDelayed` and `hospitalLate`**: the `branch_sit` rewrite is written on the **same question object** that sets the flag. The flag can only become true *after* you answer that question — by which point it's already been displayed and can't be re-shown. Self-referential and permanently dead.
- **`registerWithheld`**: same problem — the rewrite lives on the same council question that sets the flag.
- **`noTCG`**: this one *is* wired to a different, later question — "Escalating Incident Report" (Phase 3) — which is the right idea. But that question only exists in the **`strategic`** role's list, while the only question that can ever set `noTCG` ("Establishing the TCG") only exists in the **`tactical`** role's list. Since a player only ever plays one role per session, these two never meet in the same playthrough. Also permanently dead, just for a different reason (orphaned across roles instead of self-referential).

**Net result: `branch_sit` is 100% dead code.** All four instances of it in the file are unreachable under any play sequence.

## 3. What still partially compensates

`getBranchPressure()` is a second, independent mechanism (a chip label, not rewritten prose) and it isn't self-referential in the same way — it checks the flag on whichever question is loading *now*, so it can legitimately fire on a later question in the same list:

| Flag | Chip condition | Actually reachable? |
|---|---|---|
| `noTCG` | any later question, phase ≠ 1 | ✅ Works — tactical players who fail the TCG question see "No early TCG…" on every later question, and it also adds a callout line in the final debrief (`endGame()`). |
| `hospitalLate` | any Phase 3 question | ✅ Works — tactical/Ashdown has Phase 3 questions after "Hospital Notification". |
| `registerWithheld` | any Phase 2 question | ✅ Works for `council`/Ashdown (the intended case: Phase 1 question sets it, two Phase 2 questions follow). Also fires in other roles that happen to have a later Phase 2 question, but see §4 below on precision. |
| `towerDelayed` | later question titled "Meridian" | ❌ Dead — the only other "Meridian"-titled questions belong to the `fire` and `council` roles, not `tactical`. Nothing in `tactical`'s own list ever re-mentions "Meridian" by title, so this chip can never appear. **`towerDelayed` has zero visible effect anywhere, ever.** |

## 4. A precision issue worth knowing about (not a hard bug)

`registerWithheld`'s trigger is a loose substring match — `title.indexOf('Vulnerable Persons')>=0` — with no role or phase restriction. Several different roles/scenarios each have their own "Vulnerable Persons …" question (council, strategic, police, nhs, voluntary — across both scenarios), asking about different things (data-sharing consent, active outreach, door-knocking, welfare checks). Any of them scoring low sets the same generic flag, which then shows the same fixed chip text ("Vulnerable persons data not shared") regardless of which specific decision actually caused it. Not broken, just imprecise — the compounding warning may not describe the actual mistake the player made.

## 5. Scope: this only exists in one scenario

All branching (`branch_sit`/`branch_flag`) is defined in `QB` (Ashdown/wildfire) only. `QB2` (Solstice/heatwave) has **no branch points at all** — score and consequence meters still work normally there, but no choice ever changes later situation text or triggers a warning chip in that scenario.

---

## Bottom line

- **Does a bad choice matter?** Yes — immediately, via score and the four consequence meters, in every question, every role, every scenario. This drives the final outcome band correctly.
- **Does a bad choice change what happens later in the story (branching)?** Barely, and not as designed. The intended mechanism (rewritten situation text) never runs at all. Of the four planned "this compounds" moments, two produce a warning chip + debrief note as a fallback (`noTCG`, `hospitalLate`), one produces a chip only in its home scenario (`registerWithheld`), and one produces **nothing at all** (`towerDelayed`). This layer only exists in the Ashdown scenario to begin with.

## If you want it fixed

The smallest correct fix per flag:
- **`towerDelayed`/`hospitalLate`/`registerWithheld`**: move the `branch_sit` text off the setter question and onto a genuinely later question in the same role's list (the chip logic already shows which later questions are reachable).
- **`noTCG`**: either add a "TCG" Phase 1 question to `strategic`'s own list, or move the `branch_sit` reader onto a later `tactical` question instead of a `strategic` one.
- **`towerDelayed` chip**: either add a later "Meridian"-titled question to `tactical`, or relax the chip condition (e.g. match on phase instead of title, the way the other three do).

Happy to implement any of these if you want the branching to actually show up in play.

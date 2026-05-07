# Whatadisaster — Vision & Roadmap

_Last updated: 2026-05-02_

## What this is

A browser-based emergency exercise simulator for multi-agency response training. Built as a single self-contained HTML file, no build step, no backend. Two scenarios: **Operation Ashdown** (wildfire, Brynmoor Hill) and **Operation Solstice** (severe heatwave, Bristol). Eight playable Gold/Silver/Bronze command roles across fire, police, council, NHS, utilities, and ambulance services.

A single officer can play through a scenario in 15–20 minutes, see their decision log, and email the record to their planning team.

## Who it's for

- **Primary**: emergency planning officers and resilience teams in UK local authorities — currently anchored to Bristol City Council via the `submitToEPT()` mailto.
- **Secondary**: multi-agency partners (fire, police, NHS, utilities, voluntary sector) needing low-friction tabletop-style training between formal exercises.
- **Tertiary**: students and trainees on emergency management courses; resilience forums running awareness sessions.

The audience is people who are paid to think about disasters and who *already* run formal Live/Tabletop/Discussion exercises. This tool is not a replacement for those — it's the thing you can give someone to do on a Tuesday afternoon.

## Why it exists

Formal multi-agency exercises are expensive and infrequent. JESIP (Joint Emergency Services Interoperability Principles) requires regular practice, but most officers are exposed to scenarios only once or twice a year. The simulator fills the gap with:

- **Repeatable practice** at decision-making under time pressure.
- **Role empathy** — playing the Police Lead when you're a Council officer reveals the constraints the other side is working with.
- **Consequence visibility** — the four meters (Life Safety, Infrastructure, Community, Coordination) make the cost of bad decisions tangible in a way a tabletop discussion often doesn't.
- **Frictionless distribution** — a URL. No login, no install, works offline.

## Current state (v1.1.1)

- 2,418 lines, one HTML file. Vanilla JS, ES5, zero dependencies.
- Two scenarios sharing one engine via `QB`/`QB2` question banks and `MAPS`/`MAPS2` SVG data.
- Decision log saved to `localStorage` per role. Submission via `mailto:emergencyplanning@bristol.gov.uk`.
- Hosted on GitHub Pages. Companion `how-it-works.html` explainer.

The architecture is a strength: anyone can fork it, host it on a council intranet, or view-source to understand exactly what their officers are being trained on. That trust property is rare and worth protecting.

## Principles

1. **Content before code.** The value is in the question banks, the consequence model, and the scenario realism — not the framework. Most future work should be authoring, not engineering.
2. **Frictionless first.** No login, no install, works on a council laptop with locked-down browsers. Anything that adds a sign-in screen has to clear a high bar.
3. **Inspectable.** Public-sector users should be able to see what the tool does. Single-file HTML is genuinely useful here.
4. **JESIP-aligned.** Every decision and feedback panel maps back to the Joint Decision Model and the Five Principles. Don't drift.
5. **Realistic, not gamified.** Scoring exists to drive reflection, not competition. Avoid leaderboards, badges, and other patterns that trivialise the subject matter.

## Next steps

Ordered by ratio of value to effort. Stop at any point that no longer matches the project's actual demand.

### Tier 1 — low effort, immediate value

- **Replace mailto with a proper submission path.** Mailto fails on locked-down council devices that don't have a configured mail client. Options: a Google Form behind a button, or a Cloudflare Worker that forwards to email. Keeps the zero-backend feel, fixes the most common point of friction.
- **Add a scenario picker that's clearly labelled as expandable.** Today's UI implies "two scenarios, take it or leave it." A "More scenarios coming — request one" link signals this is a platform, not a pair of fixed exercises.
- **Bristol-agnostic mode.** The hardcoded Bristol email and Severn-Trent / Bristol-specific copy limits pickup outside the city. A `config` object at the top of the file (council name, planning team email, locale-specific copy) would let other authorities fork-and-rename in 5 minutes.
- **Print/PDF the debrief.** Officers want a physical record for CPD logs. A `window.print()` button with print-friendly CSS is an afternoon's work.

### Tier 2 — medium effort, opens the door to wider use

- **Externalise question banks to JSON.** Today they're inline JS objects. Moving them to `scenarios/ashdown.json`, `scenarios/solstice.json` makes scenario authoring possible without touching code. This is the unlock for the next item.
- **Scenario authoring guide.** A short markdown doc + JSON schema so a planning officer in Manchester (floods) or Cornwall (coastal) can write their own scenario. The simulator becomes a platform; you become the maintainer of the engine.
- **Facilitator mode.** A query-string flag (`?facilitator=1`) that shows correct answers, JESIP rationale, and pacing notes alongside each question. Lets one officer run a small group through the simulator as a guided session.
- **Multi-role debrief view.** A page where 8 officers who each played a different role can paste their decision logs side-by-side and see where the agencies diverged. Pure copy-paste; no backend needed.

### Tier 3 — bigger commitment, only if demand justifies

These all imply Firebase (or equivalent) and probably a proper account system. **Don't build any of this until a council asks for it and is willing to pay or pilot.**

- **Live multi-agency exercises.** Facilitator creates a session, participants join with a code, decisions sync in real time. Useful for distributed exercises across a county. Big build.
- **Cohort analytics.** A council can see "across our 40 officers who played Solstice, where did we collectively fail?" Aggregates the debrief data without identifying individuals. Real value to resilience leads.
- **Custom scenario commissioning.** Bristol commissions a flooding scenario for £X; reuse engine and SVG-map pattern; deliver in two weeks. This is the most plausible monetisation path.
- **Account system + saved progress.** Only if there's a reason an officer would log in (e.g. CPD record, history, returning to a paused exercise). Otherwise it's friction with no payoff.

### Things to deliberately not do

- **React rewrite.** The app is 2,400 lines of mostly content. A rewrite buys nothing and breaks the inspectability property. Revisit only if a Tier 3 feature genuinely demands componentisation.
- **Native mobile app.** PWA is sufficient. Council procurement teams prefer a URL.
- **Gamification (XP, levels, leaderboards).** Wrong tone for the subject matter. The four consequence meters already provide all the feedback signal needed.
- **AI-generated scenarios at runtime.** Tempting but risky — emergency training content needs to be authored and reviewed, not improvised. AI can *assist* a human author offline; it shouldn't drive the experience live.

## Distribution & monetisation (early thinking)

The realistic path looks like:

1. **Free + Bristol-anchored** (today). Builds credibility and case studies.
2. **Free + multi-council** (Tier 1–2 above). Same product, configurable per council. Word-of-mouth via resilience forums.
3. **Paid scenario authoring** (Tier 3). "We'll build you a flooding scenario for your patch" — a service offer, not a SaaS subscription. Low overhead, sustainable.
4. **Optional SaaS layer** for live exercises and cohort analytics, only if Tier 3 proves demand. Annual licence per authority makes more sense than per-seat.

The thing to protect is the free, single-file, public version. That's what gets the tool into a planning officer's hands in the first place.

## Open questions

- Is the mailto-to-Bristol mechanism actually being used by anyone outside the immediate team? (Quick way to validate: ask.)
- Would partner agencies (Avon Fire, Avon & Somerset Police, NHS South West) endorse it as a JESIP-aligned training resource? An endorsement letter is worth more than a feature.
- How does this fit alongside the Cabinet Office's Resilience Direct platform? Complementary or competitive?
- Who owns the IP if the tool is used in formal exercises? Worth a brief sanity check before any council commissions custom scenarios.

# Feedback & Release Tracking System — Scope

**Status:** Planned — Firebase setup required (local laptop)
**Last updated:** 2026-06-11

---

## Problem

Feedback currently arrives by email, gets noted in a doc, and has no formal link to when (or whether) it gets fixed. There's no way for players of the simulator to submit feedback at all. When a release goes out, there's no way to communicate what changed or close off specific requests.

---

## What We're Building

Three connected pieces:

```
[In-app form]  ──┐
                 ├──▶  Firestore (feedback collection)  ──▶  Admin UI  ──▶  Release log
[Manual entry] ──┘
```

### 1. Community Feedback Form (in-app)

A feedback button available to players — most likely on the results or debrief screen, possibly the splash screen too. Submits directly to Firestore.

**Fields collected:**
- Role played
- Scenario (Ashdown / Solstice)
- Category: `content` | `ux` | `bug` | `suggestion`
- Free text (max ~500 chars)
- Auto-captured: timestamp, browser/device type (no PII)

**Reference ID assigned automatically:** `WAD-FB-001` format (sequential, stored as a counter in Firestore).

**No auth required** for submission. Rate limit by session to prevent spam.

---

### 2. Partner / Internal Feedback Entry

Same `feedback` collection, same reference ID format. Entered manually (by us) via the admin UI or a simple script.

Additional fields for partner entries:
- `source_name` — e.g. "Clare, Emergency Planning" or "Jon Muddel, Utilities"
- `source_type`: `partner` | `community`
- `date_received` — may differ from entry date

This means feedback from emails like Clare's and Jon's gets logged with a reference ID, tracked, and formally closed when resolved — same as community items.

---

### 3. Release Log

Each release (bug fix, content update, or feature) creates an entry in a `releases` Firestore collection.

**Release entry fields:**
- `version` — e.g. `1.2.0`
- `date`
- `type`: `bugfix` | `content` | `feature` | `chore`
- `summary` — one-line description
- `items` — array of change items, each with:
  - `description`
  - `type`
  - `feedback_refs` — array of `WAD-FB-xxx` IDs resolved by this change (may be empty)

When a release is published, any feedback items listed in `feedback_refs` are automatically marked `resolved` with a pointer back to the release version.

---

## Data Model

### `feedback` collection

```
{
  ref:          "WAD-FB-001",        // human-readable ID
  source_type:  "community",         // or "partner"
  source_name:  null,                // populated for partner entries
  date:         timestamp,
  scenario:     "hotwells",          // or "ashdown" or null
  role:         "fire",              // or null
  category:     "content",
  description:  "The NWFC reference should be NRFC...",
  status:       "open",              // open | in_progress | resolved
  resolved_in:  null,                // e.g. "1.2.1" when closed
  resolved_date: null
}
```

### `releases` collection

```
{
  version:  "1.2.1",
  date:     timestamp,
  items: [
    {
      description:   "Corrected NWFC → NRFC in wildfire mutual aid question",
      type:          "content",
      feedback_refs: ["WAD-FB-002", "WAD-FB-003"]
    }
  ]
}
```

### `meta` document (counter)

```
{
  feedback_counter: 7    // increment before each new feedback write
}
```

Using a Firestore transaction on `meta/counters` to generate sequential IDs safely.

---

## Admin UI

A separate auth-gated HTML page (`admin.html` — not linked from the main app, excluded from public nav).

**Views needed:**
- Feedback list — filterable by status / category / source type
- Feedback detail — view full text, update status, add notes
- New release form — enter version, pick changes, link feedback refs (auto-closes them)
- Release history

**Auth:** Firebase Google Sign-In. Only authorised Google accounts can access.

---

## In-App Integration Points

| Location | What |
|---|---|
| `#results` screen | "How did we do? Leave feedback" button |
| `#debrief` screen | Same, contextualised to the specific role/scenario |
| `#splash` screen | Subtle "Feedback" link in footer |

The form should feel lightweight — not a survey. One category pick and a text box. Submits in the background, shows a brief confirmation, doesn't interrupt the flow.

---

## What This Is Not

- Not a public issue tracker (Firestore rules will prevent reads by unauthenticated users)
- Not a support system — no email notifications, no response mechanism (for now)
- Not a CMS — question bank content still lives in `index.html`

---

## Build Order

1. **Firebase project setup** — Firestore, Auth, security rules
2. **Sequential ID counter** — `meta/counters` document + transaction helper
3. **Community feedback write path** — form UI in `index.html`, Firestore write, rate limiting
4. **Manual partner entry** — admin UI form to log partner feedback
5. **Release log write** — admin form to create releases, auto-close linked feedback
6. **Release log read (in-app)** — optional "What's new" panel on splash or debrief

7. **Claude session-start hook** — see below

---

## Claude Auto-Review (Session-Start Hook)

At the start of each Claude Code session, a hook triggers an automatic feedback review without needing a manual prompt. Claude checks the Firestore `feedback` collection, surfaces anything new or unreviewed, flags urgent items, and summarises open bugs/features before any other work begins.

**Behaviour:**
- Runs once at session start
- Queries Firestore for feedback with `status: 'open'` or `status: 'in_progress'`
- Groups by category (bug / content / UX / feature)
- Flags anything marked urgent or from a named partner
- Posts a brief summary to the session before waiting for instruction

**Interim behaviour (pre-Firebase):**
Until Firestore is live, the hook reads `info/` feedback docs and flags any items without a resolved status.

**Setup:**
- Add `SessionStart` hook to `.claude/settings.json`
- Hook prompt: *"Check the feedback collection (or info/ docs if Firebase not yet live), summarise open items by category, flag anything urgent or from a named partner, then wait for instruction."*
- `CLAUDE.md` documents the expected behaviour so it persists across sessions

**On-demand review:**
Claude can also be asked to review feedback at any point — useful for batching fixes before a release or reviewing a surge of community submissions.

**Future: auto-triage on submission**
A Firebase Cloud Function can call the Claude API when new feedback is written, posting a triage label and severity back onto the Firestore record automatically — so by the time a session starts, items are already categorised.

---

## Open Questions

- Does the "What's new" / release panel need to be public-facing in the app, or internal only?
- Should partner feedback entries be visible in the app anywhere (e.g. "X improvements made based on partner feedback")?
- Rate limiting approach: Firestore security rules by IP, or a lightweight Cloud Function?
- Is `admin.html` hosted on the same Cloudflare Pages domain (blocked via `_redirects`) or a separate Firebase Hosting page?

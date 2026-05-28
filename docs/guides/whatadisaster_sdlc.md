# What a Disaster — SDLC

> Last updated: 2026-05-28

How changes go from idea to production at whatadisaster.uk.

---

## Branches

| Branch | Purpose | Deploys to |
|--------|---------|-----------|
| `main` | Production. All tested work lands here. | whatadisaster.uk (Cloudflare Pages, auto-deploy) |

This is a single-file app with no build step. There's no feature branch required for small content edits — open the file, test in browser, commit, push.

For larger changes (new role, new scenario, engine rework) it's worth working on a local branch and only merging to `main` when tested.

---

## Development Workflow

### 1. Start a session

```
git checkout main
git pull origin main
```

Read `DEVLOG.md` and the most recent `logs/YYYY-MM-DD.md` to understand where things are.

### 2. Edit and test locally

Open `index.html` directly in a browser. No dev server needed.

For mobile testing: use VS Code Live Server extension or Python's built-in server:
```
python -m http.server 8080
```
Then open `http://localhost:8080` on your phone (same WiFi, use your machine's local IP).

### 3. Test checklist before committing

- [ ] Played through the affected scenario/role end-to-end
- [ ] No console errors in browser devtools
- [ ] Consequence meters, timers, and branching behave correctly
- [ ] Debrief and results screens render cleanly
- [ ] Tested on mobile viewport (or at least Chrome devtools responsive mode)
- [ ] No hardcoded test data or debug `console.log` left in

### 4. Commit and push

```
git add index.html   # (and any other changed files)
git commit -m "feat: description of what changed"
git push origin main
```

Commit conventions:
- `feat:` — new feature or content (new question, new role, new scenario)
- `fix:` — bug fix
- `refactor:` — code restructure, no behaviour change
- `docs:` — documentation only

**Every commit must also update:**
- `logs/YYYY-MM-DD.md` — what was changed, files affected, key decisions
- `DEVLOG.md` — only when a milestone is complete
- `CLAUDE.md` — only if architecture or engine changed

### 5. Verify production

- Visit `https://whatadisaster.uk`
- Hard refresh (Ctrl+Shift+R) to bypass any cache
- Play through affected scenario/role briefly
- `whatadisaster.pages.dev` continues to work in parallel — no action needed

---

## Pre-Merge / Pre-Push Checklist

- [ ] Feature tested locally, desktop and mobile
- [ ] No debug code or placeholder content left in
- [ ] `logs/YYYY-MM-DD.md` updated for today's work
- [ ] `DEVLOG.md` updated if a milestone was completed
- [ ] `CLAUDE.md` updated if engine or architecture changed

---

## Rollback

If a bad push reaches production:

1. **Instant rollback:** Cloudflare Pages dashboard → Deployments → find last good deploy → Rollback
2. **Code fix:** Fix locally, test, push to `main` again

---

## What Lives Where

| Concern | Location |
|---------|---------|
| App source | `index.html` |
| Explainer page | `how-it-works.html` |
| Legal disclaimer | `disclaimer.html` |
| Vision & roadmap | `VISION.md` |
| Milestone tracker | `DEVLOG.md` |
| Daily work logs | `logs/YYYY-MM-DD.md` |
| SDLC guide | `docs/guides/whatadisaster_sdlc.md` (this file) |
| Claude Code guidance | `CLAUDE.md` |
| Change briefings | `info/` |
| Version history | `CHANGELOG.md` |

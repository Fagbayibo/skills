---
name: status
compatibility: Built for Claude Code — reads git and workflow artifacts. Installs on any Agent Skills client.
description: "Use this skill to orient yourself in a project — where things stand and what's safe to pick up — across a paused session or a team. Run /status when resuming work ('where was I?', 'what's left?', 'catch me up'), when joining a repo others are working in ('what's in progress?', 'am I behind?'), or before starting anything to avoid colliding with a teammate. It reads git state (branch, uncommitted work, ahead/behind the remote), the feature roadmap, and the ADRs, then reports what's done, what's in-progress with its resume point, and what to do next — flagging collaboration hazards (you're behind the remote, uncommitted work, a feature someone else is mid-build on). Read-only: it reports, it never writes."
---

## What this skill does

The "you are here" view. Work spans sessions and teammates, so before picking anything up you need to know: what state is the repo in, what's already in flight, and where it's safe to start. `/status` answers that from the durable artifacts and git — no memory of the last session required.

It reports:
1. **Git state** — branch, uncommitted/staged work, commits ahead/behind the remote, recent commits.
2. **Roadmap progress** — features by status (`planned`/`in-progress`/`done`), sub-task completion, any `⚠ ADR pending`.
3. **Decisions** — ADRs by status (`Proposed`/`Accepted`/`Superseded`).
4. **Resume points** — each in-progress feature and its first unchecked sub-task ("pick up at *data integration*").
5. **Collaboration hazards** — the things that bite teams and resumed sessions (below).
6. **Recommended next action** — the single most sensible next command.

## Asks vs acts

**Reads and reports only.** Writes nothing, changes nothing, runs no build. Safe to run anytime, including on a dirty tree or mid-task. Asks nothing — it just tells you where you are.

## Artifact ownership

None. Chat output only.

---

## Portability (any OS, any agent)

`git` is the only required CLI. Other reads use your agent's file tools. Runs inline (no subagent). The artifact paths below default to `docs/`, or `.workflow/` if `docs/` is a published docs site — read from whichever exists.

## Execution

### Step 1 — Git state

```bash
git rev-parse --abbrev-ref HEAD                      # current branch
git status --short                                   # uncommitted + staged + untracked
git rev-parse --verify main >/dev/null 2>&1 && BASE=main || BASE=master
git fetch --quiet 2>/dev/null                        # refresh remote view (skip if offline)
git rev-list --left-right --count origin/$BASE...HEAD 2>/dev/null   # behind <tab> ahead
git log --oneline -8                                 # recent history
```

Note: behind > 0 → **you're not up to date**; uncommitted entries → **work in progress**; ahead > 0 → **unpushed commits**.

### Step 2 — Roadmap

Scan `docs/mvp/` (or `.workflow/mvp/`) for roadmap files — one or more numbered files (`01-mvp.md`, `02-…`). Parse all of them:
- Count features by **Status** across every roadmap file; for each `in-progress` feature, list its checked/total sub-tasks and the **first unchecked** one (the resume point).
- Note any feature flagged `⚠ ADR pending` or `Needs ADR? = yes` with an empty `ADR` cell (a decision owed before building).

If there's no roadmap, say so — suggest `/mvp` (greenfield) or `/audit` (brownfield) to establish one.

### Step 3 — Decisions

List ADRs from `docs/adr/` (or `.workflow/adr/`) with their **Status** line: `Proposed` (awaiting acceptance), `Accepted` (ready to build), `Superseded`. Flag any `Proposed` ADR whose feature is already being built — that's building ahead of an unconfirmed decision.

### Step 4 — Collaboration & session hazards

Surface anything that makes it unsafe to just dive in:
- **Behind the remote** (`behind > 0`) → "Pull first — N commits on `origin/$BASE` you don't have; a teammate may have changed what you're about to touch."
- **Uncommitted work** present → list the areas; "finish or stash before starting something new."
- **In-progress feature overlap** → for each `in-progress` feature, check whether its `Code area` has commits by **other authors** in recent history (`git log --format='%an' -- <area>` shows names other than yours) → "someone else may be mid-build on *<feature>*; coordinate before continuing it."
- **Proposed ADR being built** → "*<feature>* is in-progress but ADR <NNNN> is still `Proposed`, not `Accepted`."
- **Detached HEAD / non-feature branch** → note it.

### Step 5 — Report

```
## Status

**Branch**: <name>  ·  <ahead> ahead / <behind> behind `origin/<base>`
**Working tree**: clean | <N> files changed (<areas>)

**Roadmap** (`<base path>/mvp/01-mvp.md`):
- done: <n>  ·  in-progress: <n>  ·  planned: <n>
- In progress:
  - <feature> — <c>/<t> sub-tasks · resume at **<first unchecked>**

**Decisions**: <n> Accepted · <n> Proposed · <n> Superseded
- ⚠ <NNNN> Proposed but <feature> is already building

**Heads-up**:
- <collaboration/session hazard, or "none">

**Suggested next**: <the single most sensible command — e.g. "pull, then `/develop <feature>` to resume at data integration">
```

Omit any section with nothing to say. If the tree is clean, you're up to date, and nothing is in progress, say so in one line and point at the next `planned` feature. `/status` only reports — it never starts the work for you.

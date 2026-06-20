# /harden

---
name: harden
description: Use this skill to stress-test a change against the failure modes that only show up in production — edge cases, concurrency, scale, and security. Run /harden after the code works and is tested (typically the last step before merge on medium/full tier, or when /test or /review flagged a hardening concern). It acts as a systems-level principal engineer — the person who has debugged outages at 3am — and probes the change for the ways it breaks under load, adversarial input, partial failure, and time. It produces a prioritised hardening checklist in docs/hardening/ with concrete, verifiable items; it does not rewrite your code.
---

## What this skill does

Takes working, tested code and asks the question tests rarely do: **how does this break in production?** It reasons at the systems level — concurrency, resource limits, network partitions, clock skew, adversarial input, data growth — and produces a ranked checklist of hardening items, each concrete enough to act on or verify.

- **Acts** — scopes the change, analyses it, writes the checklist. Asks only if the change set is empty.
- **Read-mostly** — it diagnoses and recommends; it writes only the checklist (not application code). With confirmation it can apply a specific, contained fix, but its default output is the checklist.
- Runs the deep analysis in a **subagent** so the heavy reading stays out of the main context.

Owns the hardening checklist (`docs/hardening/`). Does not write tests (/test), reviews (/review), code, ADRs, or CLAUDE.md.

## Asks vs acts

**Acts.** It scopes from git, analyses, and writes the checklist without upfront questions. It pauses only when there is **nothing to harden** (empty change set). After presenting the checklist, if the engineer asks it to fix a specific item, it applies that one contained change and re-states the residual risk — it does not auto-fix the whole list.

## Artifact ownership

`docs/hardening/<YYYY-MM-DD>-<branch>.md` — created by this skill only. The subagent writes it; the main model relays a summary.

---

## Execution

### 1. Scope the change set (cheap — names only)

Same scoping as /review: the change under hardening is what differs from the base branch plus uncommitted work. Gather **names and the base ref only**; the subagent reads the diff and files.

```bash
git rev-parse --verify main >/dev/null 2>&1 && BASE=main || BASE=master
CUR=$(git rev-parse --abbrev-ref HEAD)
if [ "$CUR" = "$BASE" ]; then
  echo "MODE=uncommitted BASE=$BASE"
  git diff --name-only HEAD 2>/dev/null; git ls-files --others --exclude-standard 2>/dev/null
else
  MB=$(git merge-base "$BASE" HEAD)
  echo "MODE=branch BASE=$BASE MERGE_BASE=$MB"
  git diff --name-only "$MB" 2>/dev/null; git ls-files --others --exclude-standard 2>/dev/null
fi
```

De-duplicate; drop lock/generated files from the count. **If the change set is empty**, stop and say there's nothing to harden — point /harden at a branch or make a change first. Do not spawn.

### 2. Gather lightweight pointers (do NOT read heavy files here)

Paths and cheap signals only.

```bash
find docs/adr -name "*.md" 2>/dev/null | xargs ls -t 2>/dev/null | head -3   # recent ADR paths
[ -f test-preferences.json ] && echo "HAS_TESTS"                              # are there tests to extend?
find docs/reviews -name "*.md" 2>/dev/null | xargs ls -t 2>/dev/null | head -1  # latest review, if any
```

Pass to the subagent: CLAUDE.md contents inline (short), the recent ADR **paths**, the latest review **path** (its findings inform what's already known), the diff scope, and whether tests exist.

### 3. Spawn the hardening subagent

Read `.claude/skills/harden/agent-prompt.md` (lean template; the full threat rubric lives in `harden-guide.md`, which the subagent reads itself). Fill and spawn:

- `model: "sonnet"`  (override to `opus` only if triage marked the change `critical` — deeper reasoning for high blast radius)
- `description: "Harden: <N> changed files"`
- Tools: `Read`, `Bash`, `Grep`, `Glob`, `Write` — **no `Edit`** by default (it produces a checklist, not edits). If the engineer later approves a specific fix, re-spawn with `Edit` for that one item.
- `prompt`: filled template with:
  1. Diff scope: `MODE`, `BASE`, `MERGE_BASE`, changed-file list + the exact `git diff` command
  2. CLAUDE.md contents (inline)
  3. Recent ADR paths + latest review path (read if relevant)
  4. `HAS_TESTS` (so it can say "add a test for this" vs "no test harness")
  5. Output path: `docs/hardening/<date>-<branch>.md`

### 4. Relay the result

The subagent writes the checklist and returns a compact summary. Relay:

```
## /harden complete

**Analysed by**: systems-level review on <model>
**Scope**: <N> files — <branch vs base | uncommitted>
**Checklist**: `docs/hardening/<date>-<branch>.md`

**Risk posture**: <Ship as-is | Harden before merge | Do not ship>

**Must-fix before merge** (<count>):
- <category — one line each, file:line>

**Should-harden** (<count>):
- <category — one line each>

**Watch / accept** (<count>): <how many residual risks the team is choosing to accept>
```

Show all **must-fix** items in chat; collapse should-harden and watch to counts with a pointer to the file. If the engineer wants a specific item fixed, apply that one contained change (re-spawn with `Edit`, or do it in the main thread if trivial), then re-state the residual risk. /harden does not auto-fix the list and does not invoke other skills.

---

## Reference files

- `.claude/skills/harden/agent-prompt.md` — lean spawn template the main model fills
- `.claude/skills/harden/harden-guide.md` — systems threat rubric, severity, and checklist format (read by the **subagent**, not the main model)

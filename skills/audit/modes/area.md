# Audit Mode: area scan (Phase 3)

Trigger: a path or area name was given (e.g. `/audit src/auth`).

Pre-flight additionally:
1. Check the area path exists. If not: stop immediately, tell the engineer "Path `<area>` not found. Check the path and try again.", and do not spawn a subagent.
2. Root `AGENTS.md` (the canonical file): missing, with no legacy CLAUDE.md to migrate → run the whole-repo scan (`modes/whole-repo.md`) fully first (spawn the whole-repo subagent, wait for root AGENTS.md + CLAUDE.md pointer), then continue with the area scan. Only a legacy root `CLAUDE.md` → run the legacy migration first. Exists → proceed directly.
3. Check if `<area>/AGENTS.md` exists; note present or missing.

Spawn a subagent. description: "Audit: area scan — <area>"; tools: `Read`, `Bash`, `Write`, `Edit`. Placeholder values: `PHASE=area`, `AREA=<path>`, root AGENTS.md path, area AGENTS.md path or MISSING. The subagent adds the nested pointer line to root `AGENTS.md` via the Edit tool; it does not re-create root.

After the run, parse the report's `Root gaps flagged` section:
- `ROOT_GAPS: none` → relay the full report, done.
- Gaps exist → ask (picker as above): Question: "I found things in `<area>` not reflected in root AGENTS.md. What should I do?" Option 1: `Add them now`, description: "I'll apply the additions immediately". Option 2: `Show me the diff`, description: "Print exactly what would change; I'll apply it manually". Option 3: `Skip for now`, description: "Leave root AGENTS.md as-is".
- On `Add them now`: locate the `ROOT_GAPS:` block and extract each line starting with `- ` (each holds the exact markdown to insert and the target section, `— target section: ## <section>`). Apply one Edit call per gap into root `AGENTS.md`. Do not paraphrase.
- On `Show me the diff`: print each addition as a fenced markdown block with the target section labelled. Do not write.
- On `Skip for now`: do nothing.

Relay the full report after the choice is applied.

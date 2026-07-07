# Audit Mode: gap-fill (Phase 4 — root AGENTS.md already exists)

Trigger: no area argument, codebase exists, AND root AGENTS.md already exists (including right after a legacy `CLAUDE.md` migration). Audit the whole codebase against what's written and fill the holes, conservatively.

Pre-flight additionally: note root `AGENTS.md`'s path, and list all nested `AGENTS.md` paths (excluding `node_modules` and `.git`) to pass inline.

Spawn a subagent. description: "Audit: gap-fill — codebase vs existing docs"; tools: `Read`, `Bash`, `Write`, `Edit`. Placeholder values: `PHASE=gap-fill`, root AGENTS.md path, nested AGENTS.md paths list.

After the run, handle proposals before applying:
- Nested docs the subagent created for clearly undocumented areas → already written; list them in the relay.
- `ROOT_GAPS` and `PROPOSED_ADDITIONS` to existing files → ask (`Add them now` / `Show me the diff` / `Skip for now`) exactly as in the area scan; apply with `Edit` (verbatim, no paraphrase) on `Add them now`.
- `CONTRADICTIONS` (docs the code disproves) → surface to the engineer, do not auto-fix (these touch possibly curated lines). Relay each as "`<doc>` says *X*, but the code/ADR shows *Y*" and let them decide (correct it, or update the code). Never silently overwrite.

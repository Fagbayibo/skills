# Audit Mode: whole-repo scan (Phase 2 — root + judged nested)

Trigger: pre-flight classified clearly established, or Phase 0 → `Existing codebase`. Run the Tool-skills sweep (`modes/tool-skills.md`) once the scan has identified the stack, before/with writing `AGENTS.md`.

Spawn a subagent. description: "Audit: whole-repo scan — root + nested AGENTS.md"; tools: `Read`, `Bash`, `Write`, `Edit` (`Edit` to add nested pointers into the root it just wrote). Placeholder values: `PHASE=whole-repo`, root AGENTS.md noted as MISSING, `MONOREPO_OR_NO`.

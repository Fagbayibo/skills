# Audit Mode: greenfield setup (Phase 1)

Trigger: pre-flight classified clearly greenfield, or Phase 0 → `New project`.

Step 1, ask coding patterns AND tooling. Main model asks, as decision panels (single-select unless marked; one suggested pick each; the picker adds Other automatically), up to 4 per round, as many rounds as it takes. Be thorough, not minimal; this is the one place conventions and tooling get set. First read the real scaffolded project (manifest, existing config, tools the scaffold installed), then tailor every question: skip one the stack already settles, list an already installed tool first as the suggested pick, phrase options for the actual language and framework. Answers are captured into `AGENTS.md`. `/audit` records the choices, installs nothing; installing (packages, config files, pre-commit hooks, CI) is the `/develop tooling` sub-task that follows, but ask the tooling questions here where the choice is made and recorded.

Architecture & code conventions:
- Architecture style: present all four preset options without reading their files into main context: Clean Architecture (`patterns/clean-architecture.md`), Functional (`patterns/functional.md`), Domain Driven Design (`patterns/domain-driven.md`), and SOLID OOP (`patterns/solid-oop.md`). The chosen preset is passed as a path in Step 2, and the subagent reads only that file.
- Type strictness (typed languages only; skip if untyped): `strict` (no `any`, exhaustive types) · `gradual` (strict for new code) · `loose`.
- Module & folder structure: `folder-by-feature` (colocate by feature) · `by-layer` (controllers/services/repos) · match what the scaffold already set.
- Additional code standards (multi-select): documented public APIs · a consistent error-handling pattern · validate env vars at startup · named exports only (no default exports) · consistent naming conventions · accessibility baseline on UI (WCAG AA) · conventional commit messages.

Tooling (asked here, installed by `/develop tooling`):
- Linting & formatting (adaptive): the standard linter + formatter for this stack (suggested; list an already-installed one first) · a specific alternative · minimal for now.
- Pre-commit enforcement: lint + format + typecheck on every commit (suggested) · format only · none.
- Testing gate (captured as the convention, the runner is set up by `/test`): unit + integration with a framework (suggested) · typecheck + manual `/verify` only · tests-first (TDD).
- Continuous integration: a basic CI check on push (lint, typecheck, test) (suggested) · not yet · already configured.

Adapt the list: drop what doesn't apply (no CI question for a throwaway prototype, no type-strictness for an untyped language); add any stack-specific convention worth pinning.

Step 2, resolve SELECTED_PATTERNS: a named pattern → the absolute path of the matching `patterns/*.md` file (path only, do not inline its contents). "Other" (free text) → the engineer's exact typed text, passed inline (no file).

Step 3, run the Tool-skills sweep (`modes/tool-skills.md`), then spawn a subagent. description: "Audit: greenfield setup — create root AGENTS.md + CLAUDE.md pointer"; tools: `Read`, `Bash`, `Write`. Placeholder values: `PHASE=greenfield`, `SELECTED_PATTERNS=<pattern file path, or the Other free text>`, `ADDITIONAL_STANDARDS=<all the other Step 1 selections: code standards, type strictness, folder structure, AND the tooling choices (lint/format, pre-commit, testing gate, CI)>`, `MONOREPO_OR_NO` (`yes — apps: web, api, …` if detected), plus the sweep's `INSTALLED_SKILLS` / `DECLINED_TOOLS` so the subagent writes the `Agent skills:` line. Tell it to capture the tooling choices clearly (a short `## Tooling` note or explicit Rules lines) so `/develop tooling` installs exactly what was chosen. Per `agent-prompt.md` it writes root `AGENTS.md` + `CLAUDE.md` pointer, seeds `## Build approach` from the roadmap header (else `<TBD — set by /roadmap>`), and adds per-workspace nested docs if `MONOREPO=yes`.

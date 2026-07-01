# Notes — improvements & fixes

All items below have been addressed. Kept as a record of what was fixed and where.

- [x] **Per-app ADR paths** (`docs/adr/api/001.md`) — `/architect` resolves `$ADR_DIR` → `docs/adr/<workspace>/` per monorepo workspace, `docs/adr/_root/` for repo-wide.
- [x] **Multiple mvps/adrs per whole-app refactor** — `/architect` uses one umbrella ADR directory (`NNNN-<umbrella>/` with children + `research/`) instead of many; `/develop` builds a decided rollout inline/fan-out rather than re-architecting.
- [x] **Show the source of a recommendation** — `/architect` appends `(basis: …)` to recommended options; `/mvp` does the same on roadmap recommendations + runs a sourcing subagent that web-verifies links into a `## References` section.
- [x] **Reading all mvp files when unrelated** — `/architect`, `/develop`, `/sync` now scope roadmap reads to the single numbered file containing the target feature; only `/status` (global reporter) reads all.
- [x] **ADR status "Accepted" too early** — ADR lifecycle is now `Proposed` → `In Progress` → `Accepted` (+ `Superseded`), mirroring the feature: `/architect` creates it `Proposed` (confirmation ratifies content, does not flip status), `/develop` advances it as it builds, `/sync` reconciles, `/status` flags ADR↔feature drift. `Accepted` means the feature has shipped.
- [x] **Unused code left after a refactor** — `/develop` now removes superseded code as part of the build (SKILL.md rule + `logical-guide.md` Phase 6): delete dead functions/files/branches/imports, verify nothing references them, typecheck/build/lint clean with the old code gone.

---
name: roadmap
compatibility: Built for Claude Code — uses interactive questions. Installs on any Agent Skills client but is tuned for Claude Code.
allowed-tools: Bash, Read, Grep, Glob, Write, Edit, Task, AskUserQuestion
description: "Use this skill to turn a product idea into a living, coarse, spec-driven roadmap — and to keep it current as you ship. Run /roadmap to plan a new product or the next slice of an existing one, `/roadmap replan` after a feature or phase ships to reconcile and queue what's next, and `/roadmap add <feature>` to enroll one ad-hoc feature. As a senior product engineer it asks across business, product, and go-to-market, then lays out the features, their order, phasing, per-feature process weight, and which carry a decision — each with an intent line and acceptance-criteria seeds. It writes the roadmap to docs/roadmap/. It seeds the WHAT; it does not design features (/architect), pick tools (/architect), or write code (/develop). Build tasks are derived from each feature's ADR, not guessed here."
---

## What this skill does

Turns an idea into an **ordered, coarse, living plan** — and keeps that plan honest as the product ships. It is the entry point when the question is *"what do I build, in what order, how heavy is each, and which ones need a decision first?"* — **not** *"how do I build this one thing?"* (that's `/architect` and `/develop`).

A roadmap here is deliberately **coarse and small**. Each feature row carries only: its order, its phasing, its status, a process **weight**, whether it **needs an ADR**, an **ADR pointer**, a **code area**, plus a one- or two-line **intent** and a few **acceptance-criteria seeds** (definition-of-done seeds — the WHAT). It does **not** hold an exhaustive build-task breakdown. Those tasks are **derived from each feature's ADR** when `/architect` designs it — `/roadmap` seeds the *what*; `/architect` designs the *how* and fills the tasks; `/develop` builds them.

Three modes:

| Mode | When | What it does |
|---|---|---|
| **plan** (default) | New product, or scoping the next slice of an existing one | Full pass: ask → decompose into coarse feature rows → order + phase → write the roadmap |
| **replan** | After a feature or phase ships | Reconcile what shipped, enroll needs that surfaced during the build, reorder, queue the next slice. **This is the normal living rhythm, not a rare event.** |
| **add** | An engineer invents one ad-hoc feature | Enroll **one** coarse row (intent + order + weight + Needs ADR) without re-planning the whole product |

It seeds the plan and hands you a coarse, checkable list. Architecting each feature (which fills its build tasks) and building them is the rest of the workflow.

## Asks vs acts

**Senior product engineer role.** You are scoping a product you'll be judged on shipping — be thorough across *all* dimensions, not just the fun ones. Same **infer / ask / recommend** discipline as `/architect`:
- **INFER** what the idea already tells you (product category, obvious capabilities) — don't ask it.
- **ASK** the un-inferable across business, product, and go-to-market — in as many batched rounds as needed (up to 4 questions per round; see *Decision panels*).
- **RECOMMEND** the phasing strategy, the build order, each feature's weight, and which features need an ADR — those are expert calls; present them, don't make the engineer sequence their own backlog.

**`/roadmap` never picks tools.** No provider, library, ORM, host, or BaaS is chosen or named here — that's `/architect`'s job, per feature, in the ADR. If a feature implies a tool choice, that's exactly what makes it `Needs ADR: yes`. Keep the roadmap tool-agnostic so it doesn't rot.

## Decision panels (every user-facing choice)

Every choice you put to the engineer is an **options panel**, never a neutral menu:
- **2–4 concrete options**, each a real answer to *this* product — not placeholders.
- **Exactly one** option marked **`(recommended)`**, with a one-line why. You are the senior engineer; make the call and let them override — never present equal options with no pick.
- Always include a free-text **Other** so they can override with their own answer.
- **Capability-first rendering:** use your agent's interactive option picker (`AskUserQuestion` on Claude Code); if it has none, ask the *same* options as plain text. Batched question rounds follow the same rule — up to 4 per round.

## Artifact ownership

`docs/roadmap/` — the **feature roadmap**, created and maintained by this skill. Clean separation from `/architect`, which owns `docs/adr/` (the ADR files). Other skills find a feature by scanning `docs/roadmap/` for the row that names it.

**The roadmap is a living document, not a pile of dated snapshots.** `plan`, `replan`, and `add` all **edit the roadmap in place** — reconciling and appending, never spawning a new dated file per pass. A small product is a **single file**; a big one is **split by epic** (below). Writes nothing else — no ADR files, no code, no `AGENTS.md`. **`docs/roadmap/` holds roadmap files only** — never inventories, analyses, or research docs (those are decision-support and live *with the ADR*, under `docs/adr/…/research/`, owned by `/architect`).

**File shape — single-file by default, epic-split on demand:**
- **Small product** → one file, `docs/roadmap/roadmap.md` (overview table + per-feature intent/seed blocks + legend).
- **Large product** → **split by epic**, mirroring the ADR umbrella (index + children) and monorepo per-workspace layout: an **`docs/roadmap/index.md`** (epics + order + status rollup) plus one file per epic (`docs/roadmap/auth.md`, `docs/roadmap/checkout.md`, …). **Promote on demand:** start single-file; when it grows past comfortably scanning in one view (roughly a dozen-plus features spanning clearly distinct areas), split the biggest areas into epic files and leave `index.md` as the map. Don't pre-split a small product.

Keep every file **coarse and small** — that's the whole point of splitting. If one epic file is getting long, the fix is finer features and a tighter intent, not a build-task dump.

**Status lifecycle — `/roadmap` sets the *initial* status; the pipeline advances it:**
- New features start **`planned`**. On **brownfield**, `/roadmap` also enrols pre-existing features as **`existing`** (complete) or **`in-progress`** (partial) — the one place `/roadmap` writes a status other than `planned`.
- From there, **`/develop`** advances *pipeline-built* work (`planned` → `in-progress` → `done`) and **`/sync`** reconciles against the diff.
- **`done` ≠ `existing`**: `done` means *this pipeline* built and verified it; `existing` means it predates the workflow. `/develop` and `/sync` never touch `existing` rows.
- **Pivots / de-scoping**: `replan` may set a de-scoped feature to **`dropped`** — it **never deletes rows**. `dropped` keeps history visible and excludes the feature from active counts and work. `/develop` and `/sync` skip `dropped` rows.

**Process weight — a coarse right-sizing attribute (this absorbs the old `/triage`).** Every feature carries a **Weight**: `lean` · `medium` · `full`. It's not a separate skill or step — it's one column that turns downstream process **on or off** for that feature:
- **`lean`** — trivial, low-risk, well-understood → skip design-review and skip `/harden`. Often `Needs ADR: no`.
- **`medium`** — moderate scope or a real decision → normal path.
- **`full`** — high risk, large scope, or compliance-sensitive → **design-review and `/harden` are required**; almost always `Needs ADR: yes`.

`/roadmap` sets an **initial** weight per feature (a coarse call from the same signals `/architect` and `/develop` use); the README's Tiers are the reference for what each weight buys. Downstream skills read this column to decide how much process a feature gets.

**Artifact base.** The roadmap lives under `docs/` by default. If `docs/` is a *published* docs site (`docusaurus.config.*`, `.vitepress/`, `mkdocs.yml`, Astro Starlight, or Nextra detected), use `.workflow/` instead (`.workflow/roadmap/…`). **Always follow whichever base — `docs/` or `.workflow/` — already exists** (paths here assume `docs/`).

**Concurrency & collaboration.** The roadmap is shared across sessions and teammates. **Re-read it immediately before writing** (it may have changed since you last looked); make **surgical** edits (append new rows in order, reconcile the cells that changed, never rewrite the whole file); and if it isn't in the state you expected, **flag rather than clobber**. Append new features with the next free numbers so two people adding features don't collide on a row.

---

## Reference files

- **`roadmap-template.md`** — the structure `/roadmap` writes to: the coarse overview table (with the new columns), the per-feature intent + acceptance-criteria-seeds block (and the derived-tasks note), the epic-split `index.md` and per-epic shapes, the `replan` and `add` outputs, and the `## /roadmap complete` report block. Read it when writing the roadmap and the report.

## Portability (any OS, any agent)

Written for any Agent Skills client on macOS, Linux, or Windows. Detection snippets are POSIX **reference** — use your agent's own cross-platform file tools to look for source files and read/write Markdown. Planning runs inline. Two **optional** subagents are capability-first — spawn them via your agent's subagent tool only where it exists, and degrade to inline otherwise: an optional **read-only code-scan** subagent for brownfield mapping on large repos (a **fast/cheap** model), and an optional **sourcing** subagent that runs **only if the engineer opts into web-sourced links** (Step 6b). If your tool has no interactive-question picker, ask every decision panel as plain text with the same options.

## Execution

### Step 0 — Mode & idea check

Determine the mode from how `/roadmap` was invoked:
- **`/roadmap replan`** (or a re-run described as "reconcile / what's next") → **replan mode** (see the *Replan* section) — the normal rhythm after shipping.
- **`/roadmap add <feature>`** → **add mode** (see the *Add* section) — enroll one row, no full re-plan.
- **`/roadmap <idea>`** or bare `/roadmap` → **plan mode**, below.

If plan mode and no idea was provided (`/roadmap` with no argument and no existing roadmap to extend): **stop and ask** before anything else:

"What are you building? Describe the product or the slice of it you want to plan — one or two sentences about what it does and who it's for."

Wait for the answer. Use it as the product idea.

### Step 1 — Locate the roadmap; greenfield / brownfield / monorepo

Using your agent's own file-search tools, detect (skip `node_modules/` and `.git/`):
- **Any source files** — at least one `.ts`, `.tsx`, `.js`, `.py`, `.go`, or `.rs`. Presence ⇒ brownfield; none ⇒ greenfield.
- **A root `AGENTS.md`** — note whether it exists.
- **An existing roadmap** — look under `docs/roadmap/` for `roadmap.md` (single-file) **or** `index.md` + epic files (split) — and, in a monorepo, under `docs/roadmap/<workspace>/`. Note the shape you find.

**Greenfield** — decompose the whole MVP from scratch, foundations-first (Step 3).

**Brownfield** — read root `AGENTS.md` (and the existing roadmap, if any) so you plan the *next* slice on top of what's there:
1. **Enroll the already-built features** for context — derive them from `AGENTS.md` (its nested-area docs map to existing areas) plus a light code scan, each with a `Code area` pointer. **On a large repo, offload that scan to a read-only exploration subagent** (a **fast/cheap** model with `Read`/`Grep`/`Glob`) that returns a compact map — don't read the tree inline. **Assess completeness honestly from the code**, and set status accordingly — don't just stamp everything done:
   - **Complete & shipped** → **`existing`** (a *distinct* marker — **not** `done`).
   - **Partially built** → **`in-progress`** (so `/develop` can resume it).
   Never mark a half-built feature `existing`.
2. **Plan the next slice** as `planned` rows. Don't re-plan features already complete (`existing`).
   - If there's no root `AGENTS.md`, note in the report that `/audit` should run first to give real context.

**If a roadmap already exists (a re-run) — read the *union*, don't duplicate or fragment:**
- **Read the whole roadmap** — the single file, or `index.md` + every epic file — and build the **full set of features already on it** at *any* status (`planned`, `in-progress`, `done`, `existing`, `dropped`). This is your dedup baseline.
- **Dedup against all of it.** Don't add a feature that already exists in any status. If the request overlaps an existing `planned` feature, **extend that row** (sharpen its intent / seeds) rather than creating a duplicate. Only genuinely-new features get new rows.
- **Reconcile drift.** If you find **shipped work or ADRs no roadmap row covers** (built off-plan), enroll them — completed as `existing`/`done`, unfinished as `in-progress`. Note these as "drift enrolled".
- **State what you found** in the report: how many features already on the roadmap, how many new, how many drift items, and which file(s) you wrote to. (For a full reconcile after shipping, prefer **replan mode**.)

**Monorepo — plan per workspace, don't mix apps in one roadmap.** Detect a monorepo: a workspaces config (`pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json`, or `workspaces` in root `package.json`) or multiple app/package manifests under `apps/*` / `packages/*`. If found:
- **Each workspace gets its own roadmap directory**: `docs/roadmap/<workspace>/` (single-file `roadmap.md`, or `index.md` + epics if that workspace is large). Repo-wide planning — monorepo tooling, a shared design system in `packages/ui`, cross-cutting infra — goes in **`docs/roadmap/_root/`**.
- **Scope to the workspace.** `/roadmap web <idea>` plans the `web` app; a bare `/roadmap` on a monorepo **asks which workspace(s)** to plan (or "repo-wide") — as a decision panel. Read **that workspace's** nested `AGENTS.md` for *its* stack/conventions — apps often differ (e.g. web vs api vs mobile), so don't assume one.
- **Each feature's `Code area` points into its workspace** (`apps/web/...`). Foundations are per-workspace, **except** genuinely shared ones (monorepo tooling, a shared UI package) which live in `_root` and the apps depend on.
- **A feature spanning workspaces** → plan it in `_root` (tag its intent by workspace), or split into coordinated per-workspace features. Don't bury cross-app work in one app's roadmap.

### Step 2 — Ask (batched rounds, as decision panels)

Generate questions tailored to *this* idea; infer and skip what's stated. Run as many batched rounds as needed (up to 4 per round; every question is a decision panel per the convention above). Cover:

**Round 1 — product & business.**
- **MVP boundary** — the smallest version that delivers the core value (the most important question; everything hangs off it).
- **Primary audience** — only if unclear from the idea.
- **Monetization** — free / subscription / one-time / usage-based / ads / none yet (shapes whether billing features exist).
- **Success metric** — what "working" looks like (signups, activation, revenue) — informs analytics features.
- **Hard constraints** — deadline, budget, team size, compliance scope. **These shape the phasing recommendation and the per-feature weights.**

**Round 2 — capabilities.** Multi-select of the cross-cutting capabilities the product plausibly needs, tailored to its type — e.g. authentication, multi-tenant orgs, payments/billing, email/notifications, file/media upload, search, realtime, admin panel, public API. Confirm which are in scope for *this* slice vs deferred. Each selected capability becomes one or more features. **Name capabilities, never the tool that implements them** — the tool is `/architect`'s call.

**Round 3 — cross-cutting & go-to-market.** Routinely forgotten, belong in the plan from day one:
- **SEO** — public/marketing pages, metadata, sitemap, structured data, social cards, SSR/SSG needs (skip for purely internal/auth-walled apps).
- **Performance** — Core Web Vitals targets, caching, expected load.
- **Analytics & tracking** — product analytics, error monitoring, conversion events.
- **Accessibility** — WCAG target.
- **Internationalization** — multiple languages/locales, RTL.
- **Legal/compliance** — cookie consent, privacy/terms, GDPR/CCPA, age gating.

Each "yes" becomes its own feature or folds into a relevant feature's acceptance-criteria seeds (e.g. "SEO metadata present" is a seed on each public page; cookie consent is its own feature).

### Step 3 — Choose the phasing (decision panel)

Ask **how to phase the feature sequence** — a decision panel with a recommendation you make from the product's goal and the constraints from Round 1:

- **MVP-first** — build a thin end-to-end core loop first (the one thing the product must do), then thicken. Good when the goal is fast validation of a single core value.
- **Vertical feature slices** — each feature built end-to-end (data → API → UI → integration) before the next. `(recommended)` **for a proper production build** — each slice ships something real and complete.
- **User-journey slices** — sequence by the user's path (onboarding → first value → retention), each journey end-to-end. Good when the experience/funnel is the product.
- **UI-first prototype** — build all screens against placeholder data first, wire data later. **Flag as prototype-grade:** fast and clickable for demos/feedback, **not production-complete**. Only recommend it when the explicit goal is a quick clickable prototype.
- **Other** (free text).

**Recommend one** — for a production build the default recommendation is **vertical feature slices**; shift the recommendation if the goal (from Round 1) is validation (→ MVP-first), a funnel (→ user-journey), or a throwaway demo (→ UI-first prototype). State the one-line why. Record the chosen strategy in the roadmap header — it sets the **Phasing** column values for feature rows (which slice / journey each belongs to).

### Step 4 — Foundations-first sequencing (always, regardless of phasing)

**Whatever phasing the engineer picked, the foundations come first — this is not optional and not up for a vote.** No feature slice starts before the ground it stands on exists. Sequence the roadmap so these lead, in this order, each an explicit **foundation feature** (not a sub-task buried in a page):

1. **Coding standards & conventions** — run `/audit` (greenfield) to capture the engineer's standards into root `AGENTS.md`. `Needs ADR: no` (captured by `/audit`, not designed).
2. **Stack & architecture** — `/architect` → ARCHITECTURE ADR. `Needs ADR: yes`. This is where tools/providers get chosen — not here.
3. **Data model** — an **explicit, non-skippable foundation feature** (`Needs ADR: yes`). The core entities, relationships, and persistence shape that every later feature builds on. Never fold this into another feature or skip it — a wrong data model is the most expensive thing to redo.
4. **Design system / UI foundation** — `/architect` → `design.md`, then base components (`Needs ADR: yes`) — if the product has meaningful UI. Cross-cutting: every page depends on it.
5. **Walking-skeleton slice** — a **thin vertical slice wired end-to-end** (DB → API → UI), doing **one trivial real thing** (e.g. a single record you can create and see rendered). It proves the whole stack is connected before feature work piles on. Weight `medium`, and it usually leans on the foundation ADRs above rather than needing its own.

**Then** the feature slices, ordered and phased per Step 3. The **Phasing** column marks each row as `Foundation`, `Skeleton`, the slice/journey it belongs to (e.g. `Slice 2`), or `Deferred`; the **Order** column is the integer build sequence across the whole roadmap.

### Step 5 — Decompose into coarse feature rows (you reason; don't ask)

From the answers, produce the feature list — foundations first (Step 4), then the slices, then explicitly-deferred nice-to-haves. For **each** feature, set:

- **Keep features small** — one page or one cohesive unit each. A home page and per-segment landing pages are *separate* features; a listing, a product page, and a cart are three, not one "storefront". Finer features make the roadmap honest and progress visible. If a "feature" spans unrelated screens, split it.
- **Intent (1–2 lines)** — what it is and why it matters. The one-liner a teammate reads to know what this row is for.
- **Acceptance-criteria seeds** — a few bullet "definition of done" seeds: the **WHAT**, the observable outcomes that mean this feature works (e.g. "user can filter the list by category and the URL reflects the filter"; "empty and error states render"; "SEO metadata present"). These are **seeds, not a spec** — `/architect` refines them into the ADR's full requirements and acceptance criteria. Don't over-specify; capture the load-bearing outcomes.
- **Weight** — `lean` / `medium` / `full` (see *Artifact ownership*). Set the initial call from risk, scope, and compliance sensitivity.
- **Needs ADR?** — use the *invent-test*: **would building it require a decision the engineer hasn't made?** Flag **yes** when it involves a provider/library choice, a data model, a cross-cutting pattern, the design system, a whole page/screen with no spec yet, or non-trivial behavior (search, filtering, recommendations). Flag **no** only for genuinely pure implementation an existing `design.md`/ADR/convention already covers. When unsure, flag **yes** — an unflagged decision is the expensive miss. A `full`-weight feature is almost always `yes`.
- **One decision per ADR — don't bundle, don't false-flag.** When a feature carries **multiple distinct decisions**, each is its own `Needs ADR: yes` item — don't lump unrelated decisions into one "strategy" ADR. If several genuinely share **one** broad decision that then splits, model it as an **umbrella** and let dependents reference it — but never mark a dependent `no` when it actually carries its own decision.

**No build-task breakdown here.** Each feature gets exactly one derived step: if `Needs ADR: yes`, its next action is **`/architect <feature>`** (which produces the ADR and, from it, the build tasks); if `Needs ADR: no`, it goes to `/develop` directly. Record this as the feature's `Decision (ADR)` line plus the note: *"build tasks are derived from the ADR when this feature is architected."* Do **not** enumerate UI / data-model / API / integration / test sub-tasks — that is `/architect`'s output, per feature, and guessing it here ships a wrong plan.

**Analysis/inventory is not a roadmap row.** Cataloguing duplication, listing call sites, auditing current state — that's **decision-support research** that belongs with the ADR (`/architect` produces it, under `docs/adr/…/research/`). Never plan a row or step that writes a `.md` into `docs/roadmap/`.

### Step 6 — Write the roadmap (single-file or epic-split)

Re-list the roadmap location immediately before writing (a teammate may have changed it), then write to the structure in `roadmap-template.md`:

- **Small product → single file** `docs/roadmap/roadmap.md` (monorepo: `docs/roadmap/<workspace>/roadmap.md`): the coarse overview table (with the brownfield-enrollment rows) + a per-feature intent/seed block for each row + the legend.
- **Large product → epic-split**: `docs/roadmap/index.md` (epics + order + status rollup) + one file per epic (`docs/roadmap/<epic>.md`) holding that epic's feature rows and intent/seed blocks. **Promote to this only when the single file has outgrown a comfortable scan** — otherwise stay single-file.
- **Re-run (living update)** — **edit in place**, don't spawn a dated file: append new rows with the next free `#`, sharpen existing rows' intent/seeds, and leave existing statuses untouched. Set a now-out-of-scope row to `dropped` (never delete). On brownfield, append enrolled `existing`/`in-progress` rows above the `planned` ones.

**Basis on recommendations.** Where the roadmap *recommends* something the engineer didn't dictate — the phasing choice, the order rationale, a suggested capability, flagging a feature `Needs ADR`, a weight call — append a short `(basis: …)`: a **project source** (`your AGENTS.md`, an ADR, the existing stack) or a **named practice** (`vertical slices ship real value early`, `foundations before features`, `data model is the costliest thing to redo`). You have no web tools here, so **name the source/practice — never a URL**.

### Step 6b — Ground the recommendations (sourcing subagent — ask first)

Adding **web-verified reference links** runs a subagent that web-searches and fetches pages to confirm them — useful, but it **costs extra tokens**. So **ask the engineer first** (decision panel; capability-first picker or plain text):
- **question**: "Add web-sourced reference links to the roadmap? I'll run a subagent that web-searches and fetches to verify official docs/standards — it costs some extra tokens. Either way the roadmap already names its sources."
- **header**: "Web sources"
- **options**: `No — skip it (no web, no extra tokens) (recommended)` · `Yes — fetch & verify links` · Other

**If they decline** (or there's no answer, or the agent has no web tools): **skip the subagent** — the roadmap's `(basis: …)` named sources stand on their own; add a one-line `## References` note ("Links: none — web sourcing skipped"). You're done.

**If they say yes**, spawn a **sourcing subagent** (capability-first) so links are *fetched-and-confirmed*, not fabricated:
- `model`: a **fast/cheap** model (e.g. `haiku` on Claude Code; a light model on other agents) · `description: "Roadmap: source & reference the recommendations"`
- Tools: `Read`, `Edit`, `WebSearch`, `WebFetch`
- `prompt`: give it the roadmap file path(s) and its recommendations. Its job: for the load-bearing recommendations, confirm each `(basis: …)` is sound, and where a **canonical source is worth linking** (an official doc, a named standard/practice), **web-search + fetch to confirm it exists and says what's claimed**, then add a **`## References`** section — *Project sources* (verifiable), *Practices & standards* (named), *Links* (web-verified only, else "none verified"). **Never invent a URL.** Keep it lean.
- If the client has no web tools/subagents, do this inline with named practices + project sources only (no links) — an acceptable degrade.

### Step 7 — Report and hand off

Print the completion report using the **`## /roadmap complete`** block in `roadmap-template.md`, filled with this run's specifics.

`/roadmap` does not run `/architect` or `/develop` for you — it hands you the ordered, coarse, weighted list; you walk it feature by feature (architect the `Needs ADR: yes` ones, then build).

---

## Replan (the living rhythm — run after a feature or phase ships)

`replan` is the **default cadence**, not a rare event: run it each time a feature or phase lands to keep the roadmap matching reality and to queue the next slice. It **reconciles in place** — never spawns a new file.

1. **Re-read the whole roadmap** (single file, or `index.md` + epics; the workspace's, in a monorepo) and the code/ADRs for what just shipped.
2. **Reconcile what shipped** — mark completed features `done` (verify from the code/ADR — don't stamp), tick nothing you can't confirm. Where `/develop`/`/sync` already advanced rows, leave them.
3. **Enroll needs that surfaced during the build** — read the shipped features' **ADR `## Consequences` and `## Follow-up`** sections: a follow-up ("add rate limiting", "backfill migration", "the search index we deferred") that isn't yet a roadmap row becomes a **new `planned` row** with an intent, weight, and `Needs ADR?`. This is how the roadmap grows from real build feedback rather than up-front guessing.
4. **Reprioritize / reorder** — with the new rows and what's now known, re-sequence the `Order` and adjust `Phasing` for the not-yet-built work. Foundations stay first; de-scoped work becomes `dropped` (never deleted).
5. **Queue the next slice** — make clear which feature(s) are next (the lowest-`Order` `planned` rows), and whether each is `Needs ADR: yes` (→ `/architect` next) or `no` (→ `/develop`).
6. **Report** via the completion block (mode: replan) — what you marked done, what you enrolled from ADR follow-ups, what you reordered/dropped, and the next step.

Keep it coarse and surgical: reconcile cells and append rows; don't rewrite the file.

## Add (enroll one ad-hoc feature — lightweight)

`/roadmap add <feature>` enrolls **one** coarse row without re-planning the product — for a feature the engineer invents mid-stream.

1. **Re-read the roadmap** and **dedup** — if it already exists in any status, extend that row instead of adding a duplicate.
2. **Ask only what's needed** (a short decision panel if the intent/weight is ambiguous — otherwise infer): the feature's **intent**, its **weight**, and where it sits (its **Order** / **Phasing**).
3. **Set `Needs ADR?`** with the invent-test (Step 5). If yes, its next step is `/architect <feature>`.
4. **Append one row** (next free `#`, status `planned`) with its intent line and a couple of acceptance-criteria seeds — **no build-task breakdown** (derived from the ADR later). In an epic-split roadmap, add it to the right epic file and bump that epic's rollup in `index.md`.
5. **Report** briefly (mode: add): the row added, its weight, whether it needs an ADR, and the next command.

---

## Reference

- **`roadmap-template.md`** — coarse overview table + per-feature intent/seed blocks, epic-split `index.md`/per-epic shapes, replan/add outputs, and the completion report block.

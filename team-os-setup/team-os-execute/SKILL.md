---
name: team-os-execute
description: Stage 4 of the team-os-setup chain. Reads the blueprint and builds a Level 1 (pointer) environment in a sibling directory — CLAUDE.md files, YAML registries, sample artifacts, and a _pointers.md index back to source folders. Also handles --upgrade=L2 / --upgrade=L3 flags as light drafts (no destructive ops in v1).
---

# Stage 4 — Execute (build the Level 1 environment)

You are the final stage of the `team-os-setup` chain. By the time you run, the user has approved a blueprint at Gate 3 — your job is to scaffold the actual Team OS Level 1 environment from that blueprint, validate it, and present L2/L3 upgrade previews.

**Read first:**
- `_voice.md` — voice rules (warm, peer-to-peer, junior-employee metaphor)
- `_principles.md` — sizing tiers, format-by-depth, build-on-demand defaults
- `<repo-root>/team-os-build/state.yaml` — full state from Stages 1–3
- `<repo-root>/team-os-build/research/blueprint-YYYY-MM-DD.md` — the resolved blueprint

**You write:**
- All folders + files inside the new L1 sibling directory (`<L1-root>/`)
- Updates to `state.yaml`: `target_l1_path`, `pmos_routing_rules`, `deferred_items[]`, `gates_passed[3]` (after gate approval)
- `<L1-root>/SETUP-COMPLETE.md` with L2/L3 previews

### Required `state.yaml` shape after Stage 4

By the end of Stage 4, `state.yaml` MUST contain these keys:

```yaml
target_l1_path: "<relative-or-absolute-path>"   # confirmed in Step 2
build_mode: "L1" | "L2" | "L3"                  # set in orchestrator Step 3.5
sparse_checkout:                                 # set in orchestrator Check 0.5
  cone_mode: true | false
  auto_add: true | false
pmos_routing_rules:                              # built in Step 3.5
  principle: "Save artifacts to the MOST-SPECIFIC work-folder relevant to prompt content, not the launch folder"
  routes:
    - work_scope: "<concrete description>"
      destination: "<path/from/L1-root>"
      ask_first: false
  subfolder_creation: "Skills create sub-folders (specs/, decisions/, prds/, plans/, meetings/, research/) on first use, in destination folder"
deferred_items:                                  # mirror of L1's _backlog.md (machine-readable)
  - id: "<snake_case_identifier>"
    location: "<path/from/L1-root, or 'TBD' if unknown>"
    reason: "<why deferred — 1 sentence>"
    trigger: "<what event triggers scaffolding — 1 sentence>"
artifacts:
  l1_total_files: <int>
  l1_total_folders: <int>
  l1_claude_md_count: <int>
gates_passed[3]:
  stage: 4
  name: execute
  timestamp: <ISO-8601>
  commit_sha: <orchestrator-filled>
```

`deferred_items[]` is **mandatory** — every folder skipped via `build-on-demand` rules in Stage 3 + every customer/product/area the user explicitly deferred MUST land here. This is what `--deepen <folder>` reads (Step 9.5 in orchestrator) and what `/enhance-context` checks for handoff suggestions (F8). A human-readable mirror lands in `<L1-root>/ai-ops/roadmap/backlog.md` (or wherever the team's backlog convention lives).

---

## Invocation modes

Branch on flags passed by the orchestrator:

| Flag | Mode | What runs |
|---|---|---|
| (none) | **Build L1** (default) | Steps 1–11 below + Gate 4 |
| `--upgrade=L2` | **L2 light draft** | L2 stub (see end of file) |
| `--upgrade=L3 --confirm` | **L3 light draft** | L3 stub (see end of file) |

---

## Step 1 — Read inputs

Load these into working context:
- `state.yaml` (especially: `company`, `products`, `team_functions`, `daily_collaborators`, `customers`, `decisions.*`, `candidate_features_found`, `candidate_tables_found`, `mcp_tools_indexed`, `source_pointers`, `quick_mode`, `variant`, `build_mode`, `sparse_checkout`)
- `blueprint-YYYY-MM-DD.md` — the final folder tree + per-decision record
- `_principles.md` — for sizing-tier decision (`team_routing_tier`)
- `_templates/` — all template files

If any input is missing or malformed: **halt** with a clear message naming what's missing and pointing to the resume mechanism (`/team-os-setup` from main branch will offer resume).

### Variant branching

When `state.yaml: variant == "personal"`:
- **Keep** `team/` folder (per Hannah's framework — even solo work has collaborators / mentors / beta users / contractors worth tracking). Inside:
  - **Generate `team-org.yaml`** seeded with the user as the single entry. Auto-add rule (per `_principles.md`) grows it from there.
  - **Generate `team/retros/`** with `_index.md` placeholder. Solo project + self-retros land here.
  - **Skip `team/onboarding-guides/`** — no team to onboard. Add a placement-note in `team/CLAUDE.md`: *"`onboarding-guides/` not yet scaffolded — solo work doesn't have a fan-out target. If you ever onboard a contractor or collaborator, create the folder then."*
- **Skip** per-customer scaffolding (no `customers/accounts/{slug}/` even if Stage 1 mentioned audiences).
- **Skip** `product-development/` wrapper if blueprint chose the flat shape (single-project personal use). Multi-product-independent personal use keeps `projects/{project-slug}/` siblings.
- The default L1 path becomes `../{{user-handle-or-vault-name}}-personal-os-v1/` instead of `{{company.slug}}-team-os-v1/`. Ask user to confirm in Step 2 as usual.
- `_pointers.md` is generated with the user's side-project repos as natural source pointers (Stage 1 reading pass already captured them in `source_pointers[]`).
- The Skills INDEX.md is still generated — solo work benefits from skill discoverability too.

When `state.yaml: build_mode == "L2"` or `"L3"`: dispatch to the L2/L3 light-draft sections at the bottom of this file rather than the default Step 3–11 build flow. The build-mode flag set at orchestrator Step 3.5 is what routes here.

---

## Step 2 — Confirm L1 target path

Default: `../{{company.slug}}-team-os-v1/`

Ask via AskUserQuestion (4 options, no `Other`-style fallback needed):

```
Header: "Where should I build your Level 1 Team OS?"
Description: "L1 is a new sibling folder with your structure scaffolded and pointers back to your existing docs. Nothing copies, nothing moves. If you don't like it, `git checkout main` and it's gone."

Options:
  - "Use default — ../{{company.slug}}-team-os-v1/" (Recommended)
  - "Pick a different parent directory" (free-text input next; skill appends company-slug + -team-os-v1/)
  - "Specify full custom path" (free-text input next)
  - "Cancel build"
```

**On chosen path:**
- If directory exists → AskUserQuestion: "That path already exists. Pick: [delete it and rebuild] / [pick a new path] / [cancel]." Never auto-delete without confirm.
- If chosen path is inside the source repo (`<repo-root>/...`) → warn: "Building inside your source repo means git tracks both the original and the L1. That's fine, but recovery via `git checkout main` won't restore the L1 directory if you push first. Continue anyway?"

Save chosen path to `state.yaml: target_l1_path`. Atomic write (`state.yaml.tmp` → rename).

---

## Step 3 — Walk blueprint, mkdir all folders

For each folder in the blueprint tree:
- If marked `build-on-demand` (per Stage 3 decisions) → **skip**; the parent CLAUDE.md will get a placement-note instead
- Otherwise → create the directory

Build-on-demand folders by default (per `_principles.md`):
- `product-development/product/customers/` (unless user opted in or `customers` list is non-empty)
- `product-development/engineering/bug-investigations/`
- `product-development/product/sales-enablement/` (unless `sales` is in `team_functions`)

All other folders from blueprint mkdir as-is.

### Per-work-folder `decisions/` + `meetings/` scaffolding (F4 — default-on)

For every **work folder** (a folder where PMOS skills will save artifacts — anything one or two levels under L1 root that is not `team/`, `.claude/`, or a meta-file like `_glossary.md`), Stage 4 also creates:

- `{work-folder}/decisions/_index.md` — reverse-chron decision/ADR table.
- `{work-folder}/meetings/_index.md` — reverse-chron meeting transcript / distilled-summary table.

Both `_index.md` files are seeded with this content:

```markdown
# {Decisions|Meetings} — {work-folder display name}

Reverse-chronological. New entries at top. Skills append rows here when they save artifacts to this folder.

| Date | {Decision title|Meeting topic} | File |
|---|---|---|
| _(empty — first artifact will land here)_ | | |
```

**Why default-on:** transcripts and decisions are both continuous-enrichment surfaces — without a designated home, every PMOS skill has to ask "where does this go?" and every user has to make the same call repeatedly. Real adoption surfaced this gap: a user with Granola/Otter MCP tools and regular 1:1s asked *"where do transcripts go?"* and the default L1 had no answer.

**Customization rules:**
- If the user picked Decision 1 (Stage 3) = "No meetings folder — decisions go to feature folders", **skip the `meetings/` folder** for every work folder.
- If the user picked Decision 1 = "Meetings folder + full transcripts", scaffold `meetings/` as above (the index covers either approach — distilled summary or full transcript file).
- If a work folder is build-on-demand (e.g., `customers/accounts/{slug}/`), the `decisions/` and `meetings/` subfolders inside it are also build-on-demand. Add to the customer CLAUDE.md placement-note: *"Create `decisions/` and `meetings/` here when you have artifacts to file."*
- For solo-PM cases (`function_team_size == "Just me"`), still scaffold `decisions/` everywhere but `meetings/` only at the launch-folder level — solo PMs run fewer cross-folder meetings.

Update root CLAUDE.md "Reading order for Claude" section so steps 5 and 6 (`decisions/_index.md` and `meetings/_index.md`) match the scaffolded shape.

---

## Step 3.5 — Derive PMOS routing rules

Before generating any CLAUDE.md, build `state.yaml: pmos_routing_rules` (consumed by the root CLAUDE.md template's "PMOS output routing" section + by all PMOS skills that read CLAUDE.md). If the user already populated `pmos_routing_rules` in state.yaml (custom override), skip derivation and use as-is.

**Default derivation algorithm** — walk the blueprint's top-level work folders (everything one level under L1 root that's not `team/`, `.claude/`, or a meta file) and emit one route per work folder. For multi-product mode `independent`, also emit per-product routes; for `tightly-coupled`, emit a route to the shared parent.

Output shape (matches template's `{{#each routes}}` consumer):

```yaml
pmos_routing_rules:
  principle: "Save artifacts to the MOST-SPECIFIC work-folder relevant to prompt content, not the launch folder"
  routes:
    - work_scope: "{Concrete description of work — e.g., 'Billing — PRDs, specs, dashboards, decisions'}"
      destination: "{path/from/L1-root}"
      ask_first: false   # set true for cross-cutting / company-level routes where ambiguity is likely
    - work_scope: "Cross-cutting / company-level"
      destination: "company/"   # or whichever folder holds it
      ask_first: true
  subfolder_creation: "Skills create sub-folders (specs/, decisions/, prds/, plans/, meetings/, research/) on first use, in destination folder"
```

If the blueprint doesn't include named work folders (e.g., a generic Hannah-shape build), default to the standard set:
- `product/PRDs/{feature-area}/` — PRDs
- `engineering/plans/{feature-area}/` — plans
- `engineering/rfcs/{feature-area}/` — RFCs
- `analytics/{queries,schemas,metrics,dashboards}/{feature-area}/` — analytics artifacts
- `team/` — team-level decisions (with `ask_first: true`)

Surface to user via AskUserQuestion if ambiguous:

```
Header: "PMOS routing"
Description: "Skills will save artifacts to these work folders. Confirm or override."

Options:
  - "Use defaults (Recommended)"
  - "Show me the routes and let me edit"
  - "Skip — I'll write CLAUDE.md routing rules myself"
```

The routes flow into the root CLAUDE.md's "PMOS output routing" section via the `{{#each routes}}` template loop.

---

## Step 4 — Generate CLAUDE.md at every required level

### Root `<L1-root>/CLAUDE.md`

Pick variant by `state.yaml: company.team_routing_tier`:

| Tier | Template |
|---|---|
| `inline` | `_templates/claude-md-root-inline.md` |
| `split` | `_templates/claude-md-root-split.md` |
| `pointer-only` | `_templates/claude-md-root-pointer-only.md` |

Resolve all `{{...}}` placeholders against `state.yaml`. See "Placeholder resolution" below.

### `<L1-root>/product-development/CLAUDE.md`

Use `_templates/claude-md-folder.md` with `folder.name = "Product Development"` and `folder.one_line_purpose = "Cross-functional work shared across product, engineering, design, analytics, and data-engineering. Each function has its own subfolder."`. Cross-references include `feature-index.yaml` and the function siblings.

### Each function CLAUDE.md (`<L1-root>/product-development/{function}/CLAUDE.md`)

Use `_templates/claude-md-function.md`. For each function in `team_functions`:
- `function.name`, `function.slug`, `function.name_titlecase`
- `hot_path_team` = the 3–5 names from `daily_collaborators` whose `function` matches this folder (if no match, leave the table empty with a note: *"Hot-path team not yet identified for this function. Add 3–5 daily collaborators when known."*)
- `subfolders` = folders that exist under this function in the blueprint, with one-line purposes
- `pillars` = leave `[NOT YET FILLED]` placeholder unless user provided pillars in Stage 1 free-text

For `inline` tier: hot-path table can be omitted (root has the full team) — use the function template's `Hot-path team` section conditional logic.

### Each folder CLAUDE.md (recursively, where useful)

Use `_templates/claude-md-folder.md` for these per-folder CLAUDE.md placements (drawn from Hannah's example repo):
- `product-development/product/PRDs/CLAUDE.md`
- `product-development/product/customers/CLAUDE.md` *(only if customers/ scaffolded)*
- `product-development/product/meetings/CLAUDE.md`
- `product-development/engineering/CLAUDE.md` — **set `is_engineering_root = true` so the template adds the rollout-discipline note from `_principles.md`**
- `product-development/engineering/plans/CLAUDE.md` — **set `is_plans_folder = true` so the template adds the "plan files are first-class artifacts" note**
- `product-development/engineering/rfcs/CLAUDE.md`
- `product-development/analytics/CLAUDE.md` — gets `data-catalog.yaml` reference + **set `is_analytics_root = true` for the rollout-discipline note**
- `product-development/product/workflows/CLAUDE.md` — **set `is_workflow_folder = true`**
- `product-development/product/strategy/CLAUDE.md` — add a one-line note: *"Subfolders organize by quarter (e.g., `vision/Q2-2026/`). Past quarters get pruned per NEVER #5."*
- `team/CLAUDE.md`
- `team/onboarding-guides/CLAUDE.md`

**Rollout-discipline placement-note text** (used in `engineering/CLAUDE.md` and `analytics/CLAUDE.md`):
> *"Before any feature rollout: PRD updated (status + revision history), metric definitions written, queries documented, schemas linked, `feature-index.yaml` cross-referenced. Otherwise Claude works against context rot."*

### Per-customer CLAUDE.md scaffolding (NEW — when customers list non-empty)

If `state.yaml: decisions.customers_default == "scaffold-per-customer"` (set by Stage 1 based on business model + customer detection), Stage 4 creates a customer folder + CLAUDE.md per named customer:

For each `customer` in `state.yaml: customers[]`:
1. `mkdir -p product-development/product/customers/accounts/{customer.slug}/summaries/`
2. `mkdir -p product-development/product/customers/accounts/{customer.slug}/transcripts/`
3. Render `_templates/claude-md-customer.md` into `product-development/product/customers/accounts/{customer.slug}/CLAUDE.md`
3. **Variable depth based on pre-fill files:**
   - If `customer.pre_fill_files` is non-empty (Stage 1's reading pass found customer-specific files): pass each file's extracted content into the template's `docs_found` variable + populate `key_contacts`, `segment` from extracted data where possible
   - If empty: minimal stub with `[NOT YET FILLED]` markers, doc-index points to "no files yet — drop summaries here"
4. If `customer.pre_fill_files` non-empty AND user opted in at Stage 1: also `cp` the source files into the customer folder (this is a v2 action; for v1, just leave the pointer in `_pointers.md`)

`product-development/product/customers/CLAUDE.md` (the parent) is generated using `claude-md-folder.md` template with `is_customers_root = true` and lists the scaffolded customer slugs in its doc index.

If `state.yaml: decisions.customers_default == "build-on-demand"` (B2C / pre-revenue / customers list empty), the customers folder is NOT created — placement-note in product/CLAUDE.md handles the "create when first named customer happens" pattern.

### Workflow archetype scaffolding (NEW — when Stage 3 picked one)

If `state.yaml: decisions.workflows_folder.archetype != "none"`:

| Archetype | What to scaffold |
|---|---|
| `bi-weekly-update` | Copy `_templates/sample-workflow-bi-weekly-update/` (full directory) → `<L1>/product-development/product/workflows/bi-weekly-update/`. Resolve placeholders via standard rules. Add `output/.gitkeep` and `reference/.gitkeep`. |
| `feature-launch` | Generate `<L1>/product-development/product/workflows/feature-launch/` with the same shape but stubbed content: `CLAUDE.md` (cadence: per-feature, owner: PM), `workflow-spec.md` (inputs: PRD + readiness signals; outputs: launch announcement + post-launch metrics review at 7-day + 30-day), 4 step stubs (pre-launch-readiness → launch-execution → 7-day-metrics-review → 30-day-retro), `output/` and `reference/` subfolders. Each step file = one paragraph of `[NOT YET FILLED — describe what this step does for your team]` + a paragraph of "what the step's output should look like." |
| `quarterly-planning` | Same shape as feature-launch but stubbed for: retro-of-last-quarter → strategy-review → objectives-draft → roadmap-proposal. |
| `none` (skipped) | No workflow folder content; just the parent `workflows/CLAUDE.md` describing the shape (per Decision 8 yes-result). |

For all archetypes: ensure the workflow's CLAUDE.md is generated using `_templates/claude-md-folder.md` with `is_workflow_folder = true` so the conditional shape-description renders.

### Skills INDEX.md (NEW — always generated when L1 is built)

Generate `<L1-root>/.claude/skills/INDEX.md`. Hannah's framing pairs "doc indexes" with "skill indexes" as two halves of repo discoverability. We generate doc indexes everywhere; this is the matching skill index.

Initial content:

```markdown
# Skills index — {{company.name}}

Every shared skill the team uses lives in `.claude/skills/{name}/`. This file is the index — Claude reads it when asked "is there a skill for X?"

| Skill | Auto-trigger / invocation | Purpose |
|---|---|---|
| team-os-setup | "set up team os" or `/team-os-setup` | Bootstrap or refresh the Team OS (this chain). Run once per company; re-run with `--refresh-pointers` quarterly. |

## Convention

- Add a row when you create a new shared skill at `.claude/skills/{name}/`
- Skills here are *team-shared* — everyone runs them the same way. Personal voice / writing-guide skills live in `~/.claude/skills/` (user-level, not in this repo).
- Hannah's framing: skills auto-invoke ~70% of the time — when a plan really needs a specific skill, name it explicitly so Claude doesn't miss it.

## Companion indexes

- `.claude/agents/INDEX.md` — agents this team has registered (if any)
- `.claude/commands/INDEX.md` — slash commands this team has registered (if any)
```

If `<L1-root>/.claude/agents/` or `<L1-root>/.claude/commands/` exist (currently they don't ship with v1, but adopters add them later): also generate matching `INDEX.md` files. For v1, skip these unless those folders are populated.

### Rollout-checklist artifact (NEW — when Stage 3 said yes)

If `state.yaml: decisions.rollout_checklist.choice == "yes"`:
1. `mkdir -p product-development/product/processes/`
2. Render `_templates/rollout-checklist.md` into `product-development/product/processes/rollout-checklist.md`
3. Resolve `{{company.name}}` placeholder from state.yaml; everything else is generic enough to not need personalization
4. Add a one-line entry to `product-development/product/processes/CLAUDE.md` doc index: *"`rollout-checklist.md` — pre-rollout gate (Hannah's context-rot discipline)"*

If `decisions.rollout_checklist.choice == "no"`: skip; rollout discipline lives only in the engineering + analytics CLAUDE.md notes.

For folders that ARE scaffolded but where a CLAUDE.md isn't required (e.g., individual feature-area subfolders), skip — Hannah's pattern is one CLAUDE.md per logical level, not per leaf.

### For build-on-demand folders that were skipped

Add a one-line note to the parent CLAUDE.md (after generation) like:
> *"`customers/` not yet scaffolded — create when you have a named customer to seed. Use `customers/accounts/{customer-slug}/` pattern."*

---

## Step 5 — Generate `team/team-org.yaml`

Use `_templates/team-org.yaml.seed`. Resolve placeholders:
- `daily_collaborators` from `state.yaml`
- `departments` derived from `team_functions` (one dept per function, or merge if user grouped them in Stage 1)

Always create this file, even at `inline` tier — future-proof for org growth.

If `team_org_yaml_enabled: false` (user explicitly opted out at inline tier): skip. Add a note in root CLAUDE.md: *"`team-org.yaml` opt-out. Add it later if your team grows past 15 people."*

---

## Step 6 — Generate `feature-index.yaml`

Pre-fill logic:
- If `candidate_features_found` from Stage 1 is non-empty → AskUserQuestion:
  ```
  Header: "I found these features in your existing docs:"
  Description: [list candidates with their source paths]
  Options:
    - "Pre-populate feature-index.yaml with all of them" (Recommended)
    - "Pick which to include" (multi-select next)
    - "Skip — start with one sample and expand later"
  ```
- If empty → use `_templates/feature-index.yaml.seed` with one sample feature (`{{first_feature.slug}}` = "your-first-feature")

Each feature entry: leave `eng-rfc`, `figma`, `tickets`, `schemas`, `queries`, `dashboards`, `experiments` as `null` or `[]` — these populate as artifacts land. `owner` defaults to the user's slug if known, else `[NOT YET FILLED]`.

Save to `<L1-root>/product-development/feature-index.yaml`.

---

## Step 7 — Generate `analytics/data-catalog.yaml` + `analytics/dashboards.md`

### `data-catalog.yaml`

Same pre-fill logic as Step 6 but using `candidate_tables_found`.

If empty → one sample table from the seed.

Save to `<L1-root>/product-development/analytics/data-catalog.yaml`.

### `dashboards.md` index

Generate a top-level index listing dashboards the team has (or will have). Hannah's pattern: lists every dashboard, what question it answers, where it lives, who owns it.

Initial content:

```markdown
# Dashboards — {{company.name}}

Index of every dashboard owned by the {{company.name}} team. Same role as `feature-index.yaml` but scoped to analytics surfaces.

| Dashboard | Question it answers | Location | Owner |
|---|---|---|---|
| [NOT YET FILLED] | | | |

## Convention

- Per-feature dashboards live in `dashboards/{feature-area}/{dashboard-name}.md` with the spec
- The actual visualizations live in your BI tool (Looker, Tableau, Sigma, Mode); link out from the dashboard's spec doc
- When a feature ships: add a row here + a per-feature spec doc + cross-reference in `feature-index.yaml`
```

Save to `<L1-root>/product-development/analytics/dashboards.md`.

### Skip both if `analytics` is NOT in `team_functions`

Parent `product-development/CLAUDE.md` gets a placement-note instead.

---

## Step 8 — Drop sample artifacts (gated by Stage 3 Decision 11)

Read `state.yaml: decisions.sample_artifacts.include`:

- `false` (the default) → **skip Step 8 entirely.** Add a one-line note to the relevant function CLAUDE.md doc index: *"No sample artifacts at L1 — generate real PRDs via `/prd-generator` (saves to `product/PRDs/{feature-area}/`), real plans via `/spec-to-plan`, etc."*
- `partial` → dispatch sub-agents only for the artifact types in `decisions.sample_artifacts.types` (subset of `[prd, rfc, plan]`).
- `true` → dispatch sub-agents for all 3 artifact types.

When dispatching:

### Step 8.0 — Pre-create the temp directory

Before the parallel dispatch block, ensure the temp directory exists:

```bash
mkdir -p team-os-build/tmp/
```

Without this, sub-agents writing to `team-os-build/tmp/{artifact-type}.md` would fail silently with "no such directory" errors — and a sub-agent that fails to write its temp file might still return a confirmation-shaped string, which the orchestrator could mis-read as success.

### Step 8.1 — Parallel sub-agent dispatch

**Dispatch to sub-agents in parallel — but each writes to a TEMPORARY FILE first, not directly to its final L1 path.** Hannah's rule: parallel sub-agents returning content directly to the parent simultaneously will crash and lose work. Always have them write to temp, then the orchestrator copies into final position after all return.

Each artifact is ~70–80 lines; doing them in parallel saves wall-time and keeps the orchestrator's context lean. Per `_principles.md` Context Budget rules.

Sub-agent prompt template (run 3 times in parallel via the Agent tool, `general-purpose` type — bug-investigation is build-on-demand and skipped here):

> You are personalizing one sample artifact for a Team OS Level 1 build. Read these files:
> - Template: `_templates/sample-{artifact-type}.md`
> - State: `team-os-build/state.yaml`
> - Voice rules: `_voice.md`
>
> Resolve all `{{...}}` placeholders against state.yaml. Keep `[NOT YET FILLED]` markers and ⚠️ inferred markers exactly as-is. Do not invent content for `[NOT YET FILLED]` slots — leave them for the user to fill.
>
> Voice: warm, peer-to-peer, junior-employee metaphor. Don't switch to formal/imperative when the topic gets technical.
>
> **Output: write the resolved markdown to a TEMP file at `team-os-build/tmp/{artifact-type}.md`. Do NOT write to the final L1 path. Return only a one-line confirmation: "Wrote to {temp-path}, {N} lines, {M} unresolved [NOT YET FILLED] markers preserved."** The orchestrator will copy temp → final after all sub-agents return cleanly.
>
> Word cap: don't add length. The template is the right size.

**After all 3 sub-agents return:** orchestrator runs an explicit check on each temp file BEFORE reading content:

```bash
for artifact in prd rfc plan; do
  TEMP="team-os-build/tmp/${artifact}.md"
  if [ ! -f "$TEMP" ]; then
    halt "Sub-agent for $artifact didn't write its temp file. Likely write error or sub-agent crash. Re-run Step 8 — other artifacts are safe."
  fi
  if [ ! -s "$TEMP" ]; then
    halt "Sub-agent for $artifact wrote an empty temp file. Likely template-resolution failure. Re-run Step 8 for $artifact only."
  fi
done
```

This catches the "sub-agent returned confirmation-shaped text but didn't actually write a file" failure mode. The string-match on confirmation text is unreliable; file existence + size is deterministic.

After existence/size check passes: read each temp file, run the unresolved-`{{...}}` placeholder check (Step 11) against each, then copy into final L1 paths. Delete `team-os-build/tmp/` after copy.

If any sub-agent crashes or fails to write its temp file: halt with a clear error naming the failed artifact. The other two artifacts are safe in their temp files; the user can retry just the failed one.

Targets:
- `<L1-root>/product-development/product/PRDs/{{products[0].slug}}/{{first_feature.slug}}.md` (sample PRD)
- `<L1-root>/product-development/engineering/rfcs/{{products[0].slug}}/{{first_feature.slug}}-rfc.md` (sample RFC)
- `<L1-root>/product-development/engineering/plans/{{products[0].slug}}/{{first_feature.slug}}.md` (sample plan)
- *Bug investigation* — skip dropping a sample at L1; bug-investigations is build-on-demand. Add a placement-note in `engineering/CLAUDE.md` instead.

---

## Step 9 — Create `<L1-root>/_pointers.md`

This is the L1's reference index back to the user's source folders + indexed MCP tools. Claude reads it when asked "where's the original X?" Sample shape:

```markdown
# External Reference Pointers — {{company.name}}

This file lists where your existing knowledge lives outside this Team OS. Nothing here is mirrored locally. When Claude needs source material, it follows these pointers.

## Source folders / repos

{{#each source_pointers}}
- **{{path}}** ({{kind}}) — {{notes_from_research}}
{{/each}}

## Indexed MCP tools

{{#each mcp_tools_indexed}}
- **{{name}}** — {{scope}}. {{access_note}}
{{/each}}

## When to update this file

- New source repo / folder you want Claude to know about → add a pointer line here, NOT a copy of the contents
- A pointer becomes stale (folder deleted, repo archived) → remove the line so Claude doesn't waste a fetch on it
- An MCP tool gets connected → re-run `/team-os-setup --refresh-pointers` (v1.next will add this command)

---

*Pointers are the lightest form of context — one line, one path. Use them aggressively; copy files into the Team OS only when you've decided they belong here permanently.*
```

---

## Step 10 — Create `<L1-root>/SETUP-COMPLETE.md`

This is the user-facing landing page when they open the L1 directory. Sections:

1. **What was built** — short bullet summary of the tree (functions, key files)
2. **Decisions you made** — table of Stage 1–3 decisions with rationale
3. **L2 preview** — list of source files L2 would clone, formatted as a copy-paste-ready manual command list (since L2 is a light draft in v1)
4. **L3 preview** — same with destructive deletes flagged
5. **What's NOT YET FILLED** — list of `[NOT YET FILLED]` markers across the tree, with file paths
6. **Next steps** — open in IDE, ask Claude a test question, push to remote when ready

Voice-check this file specifically — it's the longest user-facing output and the most likely to drift formal.

---

## Step 11 — Validation (run before Gate 4)

Hard checks. If any fail: report which, halt before Gate 4.

| Check | Method |
|---|---|
| Every blueprint folder exists (excluding build-on-demand) | Walk blueprint, stat each path |
| Every CLAUDE.md non-empty (>10 lines) | Read + line count per file |
| `feature-index.yaml`, `data-catalog.yaml`, `team-org.yaml` parse + have ≥1 entry | YAML parse + count top-level keys |
| `_pointers.md` lists ≥1 pointer (or notes "fresh start" if user chose 'none' in Stage 1) | grep pointer table rows |
| Sample artifacts non-empty (only if `decisions.sample_artifacts.include == true`) | Read + line count |
| **No unresolved `{{...}}` placeholders anywhere in the L1 tree** | grep `-rE '\{\{[^}]+\}\}'` across `<L1-root>/`; if matches, **halt** and list which files contain unresolved placeholders |
| **No unresolved `[NOT YET FILLED]` markers in CLAUDE.md FILES** (sample artifacts are allowed to keep them) | grep CLAUDE.md only; warn (don't halt) if found — these mean Stage 1 should have asked something it didn't |
| **Sparse-checkout disk presence** (only if `state.yaml: sparse_checkout.cone_mode == true`) | After all writes complete, walk the blueprint and stat each path with the OS file system. Any path that exists in git but not on disk = sparse-checkout still excludes it. **Halt** with: *"L1 wrote to git but `{paths}` aren't on disk — sparse-checkout cone still excludes them. Run `git sparse-checkout add {target_l1_path}` and re-run validation."* |

The unresolved-placeholder check is critical — without it, an adopter could ship a CLAUDE.md to their repo with raw Mustache syntax visible.

The sparse-checkout disk-presence check catches the silent "files commit to git but don't appear on disk" failure mode that cone-mode sparse-checkout introduces.

---

## Step 12 — Gate 4 (final user check)

Print 1-paragraph summary of the L1 build (folder count, CLAUDE.md count, sample artifact count, build-on-demand count). Then AskUserQuestion:

```
Header: "L1 built at {{target_l1_path}}. Open it in your IDE and ask Claude a question to test."
Description: "Quick test: ask Claude 'what's our team structure?' — it should answer using your CLAUDE.md and team-org.yaml. If the answer is good, finalize. If something looks off, iterate."

Options:
  - "Looks good — finalize"
  - "Iterate — something's off" (free-text note next; regenerate failing parts)
  - "Show L2 preview details"
  - "Show L3 preview details"
```

On **finalize**:
- Write `gates_passed[3]` to `state.yaml` with timestamp
- Hand control back to orchestrator (which auto-commits + presents the final closer)

On **iterate**:
- Capture free-text note
- Determine which step needs re-running (likely Step 4 CLAUDE.md generation or Step 8 sample artifacts)
- Re-run that step only; re-validate; re-present Gate 4

On **show L2/L3 preview details**:
- Print the embedded section from `SETUP-COMPLETE.md`
- Re-present Gate 4 (don't advance)

---

## Placeholder resolution rules

When resolving `{{...}}` in any template:

- `{{company.name}}` → `state.yaml: company.name`
- `{{company.slug}}` → `state.yaml: company.slug`
- `{{products[0].name}}` → first product's name; if `products` is empty, halt with "Stage 1 didn't capture a product name — re-run Stage 1"
- `{{first_feature.slug}}` → first entry of `candidate_features_found` if non-empty, else `"your-first-feature"`
- `{{first_feature.owner_slug}}` → first entry of `daily_collaborators` if non-empty, else `[NOT YET FILLED]`
- `{{today}}` → today's date in YYYY-MM-DD
- `{{#each X}}...{{/each}}` → expand for each item in the collection; if empty, omit the section entirely (don't leave a header with no rows)
- `{{#if X}}...{{else}}...{{/if}}` → branch on truthy state.yaml value

**If a placeholder doesn't resolve cleanly:** leave it as `[NOT YET FILLED — {placeholder description}]` rather than rendering raw `{{...}}`. The validation step will catch and surface these.

---

## Hard rules (NEVER do these)

- **NEVER** create the L1 directory if it already exists without explicit user confirmation
- **NEVER** auto-delete an existing L1 directory; always ask
- **NEVER** advance `gates_passed[3]` if validation fails
- **NEVER** ship a CLAUDE.md with unresolved `{{...}}` placeholders (validation catches this)
- **NEVER** copy or move files from source pointers in default L1 build (that's L2/L3)
- **NEVER** auto-commit; the orchestrator handles git ops
- **ALWAYS** use the `AskUserQuestion` tool for path-confirmation, feature-list pre-fill, and Gate 4 — never inline prose questions. Prose-shaped questions blend into description text and users can read past them.

---

## L2 light draft (`--upgrade=L2`)

When invoked with `--upgrade=L2`:

1. **Locate L1.** Read `state.yaml: target_l1_path`. If missing or path doesn't exist, AskUserQuestion: "Where's the L1 you want to upgrade?"
2. **Re-compute L2 clone list** against current source pointer state. Walk each pointer (Glob for local, WebFetch for GitHub URL — dispatch sub-agent for large pointers per `_principles.md` Context Budget rules). Match each source file to its target slot in the L1 tree:
   - `*-prd.md`, `*-PRD.md` → `product-development/product/PRDs/{matched-product}/{slug}.md`
   - `*-rfc.md` → `engineering/rfcs/{matched-product}/{slug}-rfc.md`
   - `*-plan.md`, `*-implementation.md` → `engineering/plans/{matched-product}/{slug}.md`
   - `*-schema.md`, `*_schema.sql` → `analytics/schemas/{matched-product}/{slug}.md`
   - Slugs/products matched by string contains; ambiguous matches go in a "needs hand-routing" list
3. **Render the resulting tree as a preview.** Print the L1 tree showing:
   - Existing L1 files (no change marker)
   - Newly-added clones (`+` marker, with source path in parens)
   - Folders that will be created (`+` marker on the folder)
   - Ambiguous matches grouped at the bottom under "Files needing hand-routing"

   This is what the user sees BEFORE any commands are generated. The principle: users always want to verify the resulting folder structure before any clone/move runs. L2 isn't destructive but the preview pattern applies — see L3 below for the destructive case where the preview becomes critical.
4. **AskUserQuestion** after the tree preview:
   ```
   Header: "L2 preview"
   Description: "Above is the tree shape after L2. {N} files would clone, {M} folders would be created, {K} files need hand-routing."

   Options:
     - "Generate manual command list" (Recommended)
     - "Show me the full file-by-file mapping first"
     - "Pick a different L1 path"
     - "Cancel"
   ```
4. **On confirm:** save to `<L1>/L2-commands-{date}.txt`. Format:
   ```
   # L2 manual clone commands — generated {{today}}
   # v1 doesn't execute these. Review and run yourself if confident.
   # v2 will execute with rollback contracts.

   cp "/path/to/source/billing-prd.md" "<L1>/product-development/product/PRDs/billing/credit-usage-dashboard.md"
   cp "/path/to/source/auth-rfc.md" "<L1>/product-development/engineering/rfcs/auth/rfc.md"
   # ... + N more
   
   # Files needing hand-routing (ambiguous matches):
   # - /path/to/source/strategy.md → ?
   ```
5. **Print summary** + next-step: "Review and run the commands when ready. v2 will execute these with safety checks."
6. **No auto-commit.** No file changes by skill.

---

## L3 light draft (`--upgrade=L3 --confirm`)

When invoked with `--upgrade=L3 --confirm`:

1. Same locate + recompute as L2.
2. Compute additional list: originals to delete after clone (i.e., the source paths from step 2 above).
3. **Render the resulting tree TWICE** (the principle: users want to verify the post-move structure BEFORE any destructive change runs):

   **Preview A — what the L1 will look like after L3:**
   ```
   <L1-root>/
   ├── product-development/
   │   ├── product/PRDs/billing/
   │   │   └── credit-usage-dashboard.md       ← from /source/billing-prd.md (will be MOVED)
   │   ...
   ```

   **Preview B — what disappears from sources:**
   ```
   /source/billing-prd.md                       ✗ deleted after move
   /source/auth-rfc.md                          ✗ deleted after move
   ...
   ```

   Both previews print BEFORE any prompts. The user can scroll back and verify before committing.

4. **Hard prompt** (AskUserQuestion):
   ```
   Header: "L3 destructive move"
   Description: "{N} source files will be MOVED into the L1 (delete from origin after copy). v1 generates manual command pairs; v2 will execute with atomic rollback. Review the preview above before continuing."

   Options:
     - "Show the full file-by-file mapping first" (Recommended for first L3 run)
     - "Generate the command list — I've reviewed the preview"
     - "Cancel — pick L2 instead (non-destructive)"
     - "Cancel"
   ```

5. **On "generate":** save to `<L1>/L3-commands-{date}.txt`:
   ```
   # L3 manual move commands — generated {{today}}
   # v1 doesn't execute these. Review CAREFULLY before running — these delete originals.
   # v2 will execute with atomic rollback contracts.
   #
   # SAFETY: each line is a cp + rm pair. Run cp's first, verify, THEN run rm's.
   # Quote every path — paths with spaces will break otherwise.

   cp "/source/billing-prd.md" "<L1>/.../credit-usage-dashboard.md" && rm "/source/billing-prd.md"
   # ... + N more
   ```
6. Print summary + recovery line: *"If you ran the cp's but want to abort before the rm's: nothing's lost yet. Originals + clones both exist. Skip the rm lines and clean up later."*
7. No deletions, no auto-commit.

**Path-with-spaces note:** Windows users frequently have paths like `C:/Users/Foo/My Documents/...`. The quoted form in the command file handles this; if a user ever scripts these without quotes, things break silently. The command file's header comment calls this out explicitly.

---

## Gate 4 output (success)

Write to `state.yaml`:
- `gates_passed[3]` = {stage: 4, timestamp, commit_sha: null} (orchestrator fills commit_sha after auto-commit)
- `last_gate_passed` = 4

Hand back to orchestrator. The orchestrator handles auto-commit + final closer.

# Architecture Principles — `_principles.md`

This file holds the **opinionated defaults** the chain applies — the rules sub-skills consult when they need to make a structural call. Some come straight from Hannah's published guidance. Others extend it (clearly marked).

Sub-skills read this file alongside `_voice.md` before generating any structural output (folder trees, CLAUDE.md variants, yaml seeds, sample artifacts).

---

## Source attribution

| Block | Source |
|---|---|
| Hierarchical context loading | Hannah, "CLAUDE.md Files" article |
| 100–150 instruction budget | Hannah, "CLAUDE.md Files" |
| Repeating product-area pattern | Hannah, `team-os-example-repo` |
| feature-index.yaml + data-catalog.yaml | Hannah, `team-os-example-repo` |
| NEVER list | Hannah, "CLAUDE.md Files" |
| Format-by-depth (markdown / YAML / MCP) | **Extension** — synthesized from Hannah's lean-context principle + her MCP article |
| Org-size routing tiers | **Extension** — Hannah's example assumes ~10 ppl; this fills the gap for larger orgs |
| Build-on-demand defaults for `customers/`, `bug-investigations/`, `sales-enablement/` | **Extension** — direct application of Hannah's NEVER #5 (no dead context) |
| Reading order doctrine | **Extension** — every root CLAUDE.md gets a 9-step "Reading order for Claude" section enforcing the orientation pass |
| Subfolder conventions (decisions/, meetings/, prds/, plans/, research/) | **Extension** — opinionated subfolder layout each work folder uses |
| Auto-scaffold `decisions/` + `meetings/` per work folder | **Extension** — continuous-enrichment surfaces get a designated home from day one |
| `deferred_items[]` machine-readable mirror in state.yaml | **Extension** — drives `--deepen` mode + `/enhance-context` handoff |
| PMOS output routing as a load-bearing protocol | **Extension** — codified in `_doctrine/output-routing.md`; PMOS skills walk root CLAUDE.md routing rules on save |

**When generating anything that depends on an Extension rule:** sub-skills must surface this in user-facing copy — *"Hannah doesn't directly cover this; here's our opinionated default. You can override."*

---

## Format-by-depth (the rule for which file format goes where)

| Data shape | Format | Why | Examples |
|---|---|---|---|
| **Always-loaded context** | Markdown (`CLAUDE.md`, `AGENTS.md`) | Claude reads it on every relevant query; format must be human-friendly and scannable | Root CLAUDE.md, function CLAUDE.md, folder CLAUDE.md |
| **Lookup-on-demand registries** | YAML | Claude reads only when asked; structured data wins on lookups; no parsing cost on every call | `team/team-org.yaml`, `feature-index.yaml`, `analytics/data-catalog.yaml` |
| **Dynamic temporal data** | Source-of-truth tool (MCP); do NOT duplicate to repo | Transcripts, calendar, slack messages change constantly; mirroring them creates stale context | Granola transcripts, Slack channels, calendar invites, Linear tickets |

**Operational rule:** if you're tempted to mirror data from a tool into a markdown file in the repo, ask: *does this data change more than weekly?* If yes, leave it in the tool of origin and link out.

---

## Hierarchical context loading

Three levels, smallest scope wins on conflict:

1. **Root CLAUDE.md** — always loaded. Doc index, team data (or pointer to `team-org.yaml`), terminology, top-level conventions. Budget: ≤500 lines for inline-tier; ≤200 for split-tier; ≤100 for pointer-only-tier.
2. **Function/folder CLAUDE.md** — loaded when querying that area. **Doc index FIRST** (this is the primary purpose — without it, Claude runs explore agents to search the repo, burning context). Plus function-specific terminology, hot-path team table, area conventions. Budget: ≤300 lines per file.
3. **Content files** — loaded on explicit request or when Claude follows a doc-index pointer. PRDs, RFCs, schemas, plans. No budget cap; one file per artifact.

**The 80% rule:** *"What you want in a CLAUDE.md is things you need on 80% of sessions."* If a piece of context isn't relevant to most sessions where the file loads, it doesn't belong there — link it from the doc index instead.

Total instruction budget across all files loaded for any given query: **100–150 instructions max**. If you're approaching 150, something must come out before something goes in.

---

## Repeating product-area pattern

Hannah's example repo organizes feature-area folders the same way under every function. That structure makes Claude's pathfinding predictable:

```
product/PRDs/{billing, deployment, home-page, ...}/
engineering/plans/{billing, deployment, home-page, ...}/
engineering/rfcs/{billing, deployment, home-page, ...}/
analytics/queries/{billing, deployment, home-page, ...}/
analytics/schemas/{billing, deployment, home-page, ...}/
analytics/metrics/{billing, deployment, home-page, ...}/
analytics/dashboards/{billing, deployment, home-page, ...}/
analytics/experiments/{billing, deployment, home-page, ...}/
```

**Rule:** the same feature-area folder names repeat across every function that has feature-area-segmented work. Sub-skills must enforce this; if a user adds `auth` as a feature area, it shows up in all relevant function trees, not just one.

---

## Master indexes (the cross-cutting layer)

Two YAML files act as the cross-functional traceability layer:

### `feature-index.yaml`

Maps each feature to all its artifacts across functions. Every feature with cross-functional work gets one entry:

```yaml
billing:
  credit-usage-dashboard:
    prd: product/PRDs/billing/credit-usage-dashboard.md
    eng-plan: engineering/plans/billing/credit-usage-dashboard.md
    eng-rfc: engineering/rfcs/billing/credit-usage-dashboard-rfc.md
    figma: https://figma.com/file/forge-credit-dashboard
    tickets: [FORGE/1025, FORGE/1031]
    schemas: [analytics/schemas/billing/credit_transactions.md]
    queries: [analytics/queries/billing/credit-burn-rate.sql]
    dashboards: analytics/dashboards/billing/credit-usage-dashboards.md
```

Solo work doesn't need this from day 1; populate as cross-functional artifacts land.

### `analytics/data-catalog.yaml`

Maps each data-warehouse table to its metadata: owner, refresh cadence, upstream source, downstream consumers, schema doc.

```yaml
credit_transactions:
  database: analytics_warehouse
  schema: public
  owner: aaron@company.com
  refresh: hourly
  upstream: stripe.charges
  used_by: [billing.credit-usage-dashboard, billing.low-balance-warning]
  schema_doc: analytics/schemas/billing/credit_transactions.md
```

---

## Org-size routing tiers (Extension — beyond Hannah's published guidance)

Hannah's example repo inlines the full team in the root CLAUDE.md. That works at ~10 people. Past that, lean-context discipline forces a split. Three tiers:

| Org size | Tier | Root CLAUDE.md team data | `team/team-org.yaml` |
|---|---|---|---|
| <15 | **`inline`** | Full team inlined in root | Optional (skip unless growing) |
| 15–50 | **`split`** | Doc index + pointer to `team-org.yaml`; leadership inlined only | **Recommended** — full org structured by dept |
| 50+ | **`pointer-only`** | Pointer-only — never inline team data | **Recommended** — full org structured by dept |

### Inline rules for `split` / `pointer-only`

- **Function-area CLAUDE.md** inlines a small team table — hot path only (manager + 3–5 direct daily collaborators).
- Final line of every function-area CLAUDE.md reads: *"For any other person referenced in this session, look up `team/team-org.yaml` first."*

### Auto-add rule for `team-org.yaml`

When Claude can't find a referenced person in `team-org.yaml`, sessions should:
1. Surface the gap: *"{Name} isn't in `team-org.yaml` yet."*
2. Ask the user whether to add (capture name, role, area, dept, manager).
3. If yes, append the entry in-session.

This rule is reproduced in the function-area CLAUDE.md template footer so it's always loaded when Claude operates in that area.

---

## Rollout discipline (avoid context rot)

Hannah's framing: a feature isn't shipped until the repo is updated. Otherwise Claude works against stale context — she calls this "context rot."

**Operational rule:** before any feature rollout, the team updates:
- The PRD (with shipped status + revision history line)
- The metric definitions for the feature (in `analytics/metrics/{feature-area}/`)
- The queries backing those metrics (in `analytics/queries/{feature-area}/`)
- The schemas backing those queries (in `analytics/schemas/{feature-area}/`)
- `feature-index.yaml` cross-references (PRD ↔ RFC ↔ plan ↔ tickets ↔ analytics)

This rule lives in `engineering/CLAUDE.md` and `analytics/CLAUDE.md` placement-notes (auto-added by Stage 4) so it's always in front of the team when they're working in those areas.

---

## Build-on-demand defaults

Two folders that **don't** scaffold by default at L1, even though Hannah's example repo includes them:

- `engineering/bug-investigations/` — only when first bug investigation happens; CLAUDE.md note explains the `bug-{date}-{description}/` pattern
- `product/sales-enablement/` — only if user has a sales function

**Why:** Hannah's NEVER #5 is "no dead context from finished projects." An empty folder is dead context. Scaffolding folders the user hasn't earned yet pollutes the navigation surface.

**Operational rule:** when omitting a build-on-demand folder, the parent CLAUDE.md gets a one-line note: *"`{folder-name}/` not yet scaffolded. Create when first {use-case}."*

### Customer folder is B2B-aware (revised default)

`product/customers/accounts/{slug}/` is **conditionally scaffolded** based on Stage 1's business-model question:

| Business model | Customers list non-empty | Customer folder behavior |
|---|---|---|
| `b2b` | yes | **Scaffold per-customer CLAUDE.md** for each named customer (richer for those with pre-fill files; minimal stubs for those without) |
| `mixed` | yes | Same as B2B for the named ones; build-on-demand for the rest |
| `b2c` / `self-serve` / `pre-revenue` | (any) | Build-on-demand — no customer folder created |
| any | empty | Build-on-demand — no customer folder created |

This matches Hannah's framing that per-customer CLAUDE.md is "the pattern, not optional" for B2B teams, while staying lean for non-B2B contexts.

**Variable depth rule:** customers with files found in source pointers get richer scaffolds; customers without get minimal stubs. Not all customers need same level of detail — a 3-customer B2B startup and a 200-customer enterprise team both work the same way.

---

## Decision-time defaults (sub-skills consult these)

Quick-reference for Stage 3 (`team-os-advanced`) decisions when user doesn't override:

| Decision | Default |
|---|---|
| Multi-product | Single product (asked in Stage 1, not 3) |
| Versioning | Living docs (one PRD per feature, edited in place, revision-history at bottom) |
| Meetings | Meetings folder + distilled summaries (not raw transcripts); per-work-folder `meetings/` scaffolded by default (F4) |
| Customer accounts | Build on demand |
| Bug investigations | Build on demand |
| Sales enablement | Build on demand |
| Onboarding fan-out | General + per-role (mirrors Hannah's example) |
| Strategy folder | Full `strategy/{business-context, plans, roadmaps, vision}/` — each subfolder organizes **by quarter** (`vision/Q2-2026/`, `roadmaps/Q2-2026/`). Past quarters get pruned per NEVER #5. |
| Design integration | CLAUDE.md only with Figma links (no design specs in repo) |
| Data catalog scope | Warehouse tables only (event schemas are v2) |
| Workflows folder | Yes — `processes/` and `workflows/` separation |
| Retros cadence | Per-quarter folder |
| Sample artifacts at L1 build | **Skip** — generate real artifacts via PMOS skills (`/prd-generator`, `/spec-to-plan`, etc.) rather than hand-edit stubs |
| Decisions/ subfolder per work folder | Scaffold by default with `_index.md` (F4) |
| Build mode (L1/L2/L3) | L1 — pointer-only (asked explicitly at preflight, F6) |

---

## Naming conventions (enforced by sub-skills)

- **Folders:** `lowercase-with-hyphens` (e.g., `home-page`, not `HomePage` or `home_page`)
- **Files:** `lowercase-with-hyphens.md` (e.g., `credit-usage-dashboard.md`)
- **Bug folders:** `bug-{YYYY-MM-DD}-{slug}/` (e.g., `bug-2026-05-01-payment-timeout/`)
- **AI context filename:** `CLAUDE.md` (default) or `AGENTS.md` (Cursor users) — pick one and use it everywhere
- **Master indexes:** `{type}-index.yaml` (e.g., `feature-index.yaml`, `cross-product-index.yaml`)

---

## Context Budget & Sub-Agent Dispatch (referenced from `team-os-setup/SKILL.md` line 14)

Hannah's lean-context principle ("every line costs thinking room") applies recursively to the orchestrator's own context window. Sub-skills produce structured findings; the orchestrator should NOT carry the raw reading-pass output from Stage 1 in its window all the way through Stage 4.

### The >2k-token rule for sub-agent dispatch

When the orchestrator or a sub-skill is about to do work that would consume >2k tokens of its own window — large file reads, full-folder Glob walks, GitHub repo fetches, multi-file YAML parses — dispatch to a sub-agent (Agent tool) instead. The sub-agent returns a structured summary; the parent's window stays lean.

| Work shape | Where it runs |
|---|---|
| Inline: AskUserQuestion calls, state.yaml read/write, single-file Read, atomic checkpointing, gate-handling decisions | Parent window (orchestrator or sub-skill) |
| Sub-agent: source-pointer reading pass (>50 files), GitHub repo discovery, parallel sample-artifact generation, large-pointer recompute for L2/L3 | Spawned via Agent tool |
| Separate skill invocation: stage transitions (research → propose → advanced → execute) | `Skill()` invocation when available; Agent dispatch as fallback when nested skills aren't registered top-level (the documented current state for team-os-* sub-skills) |

The "separate skill invocation" pattern matters because each sub-skill gets its own context window — state carries between them via `state.yaml` only. This is what makes a 4-stage chain feasible at all without context overflow.

### Why parallel sub-agents must write to temp first

If 3 sub-agents return content directly to the parent simultaneously, the parent's tool-result handling can race. Hannah's standing rule: parallel sub-agents always write to temp files (`team-os-build/tmp/{name}.md`); the parent reads + copies into final positions sequentially after all return. Stage 4 Step 8 enforces this for sample-artifact generation.

---

## Reading order for Claude (codified from F3)

When generating a root `CLAUDE.md` template, every variant (`inline`/`split`/`pointer-only`) MUST include a "Reading order for Claude (any task)" section near the top. The default sequence:

1. Root `CLAUDE.md` — routing rules, glossary pointer, conventions.
2. `_glossary.md` — canonical terms.
3. Work-folder `CLAUDE.md` — area-specific context.
4. Parent work-folder `CLAUDE.md` if launch folder is nested.
5. `decisions/_index.md` in launch folder.
6. `meetings/_index.md` in launch folder.
7. Stakeholder lens — `team.md` / `personas.md` / `team-org.yaml`.
8. `goals.md` — when relevant.
9. Other root files — skim only.

Steps 1–4 are the **mandatory orientation pass**; 5–9 are **scoped deep-reads**. Stage 4's root-CLAUDE.md generation logic enforces this; the generated section names the steps explicitly so a Claude session loading the file can't infer its own (often wrong) order.

**Why this matters:** without explicit guidance in the root CLAUDE.md, Claude infers its own context-loading order — and frequently picks the wrong one (reads `_glossary.md` late, skips the root CLAUDE.md with the routing rules, etc.). Explicit reading order means Claude can't know which files are mandatory vs scoped — undermining the whole "feed only relevant info" point of the structure.

---

## Subfolder conventions (codified from F5)

Routing rules tell skills which **work folder** to save to. Subfolder conventions tell them which **subfolder within that folder** to use. Both must live in the root CLAUDE.md to be load-bearing.

The canonical subfolder set:

| Artifact type | Subfolder | Filename pattern |
|---|---|---|
| Decision / ADR | `decisions/` | `YYMMDD-{slug}.md` |
| Meeting transcript / distilled summary | `meetings/` | `YYMMDD-{topic-slug}.md` |
| PRD / spec | `prds/` (or `specs/` if team prefers) | `{feature-slug}.md` (living) or `YYMMDD-{slug}.md` (date-prefixed) |
| Implementation plan | `plans/` | `{feature-slug}.md` |
| Research / synthesis | `research/` | `YYMMDD-{topic-slug}.md` |
| Sample / reference / template | `reference/` | `{slug}.md` |

**Operational rules:**
- `decisions/` and `meetings/` are scaffolded by default at L1 build time for every work folder (F4 default-on rule). The other subfolders are created on first use.
- Each subfolder gets a `_index.md` (reverse-chron table — date | title | file) when ≥3 entries land. Skills append rows to `_index.md` as they save artifacts.
- Filename pattern follows the versioning decision from Stage 2:
  - `update_in_place` choice → `{feature-slug}.md` (single living file)
  - `date_prefixed` choice → `YYMMDD-{slug}.md` (new file per major rev)
  - `archive_on_supersede` choice → current file flat; superseded ones move to `_archive/`

PMOS skills implement this via the F9 doctrine doc (`03 Resources/AI Skills/_doctrine/output-routing.md`).

---

## Build-on-demand vs auto-scaffold (decision tree)

The trigger question for any folder: *"Does the user have ≥3 artifacts of this type already, or a near-term commitment to produce them?"*

| Folder | Default at L1 | Trigger to scaffold later |
|---|---|---|
| `decisions/` (per work folder) | **Auto-scaffold** with empty `_index.md` (F4) | n/a — already scaffolded |
| `meetings/` (per work folder) | **Auto-scaffold** with empty `_index.md` (F4 — unless Stage 3 Decision 1 said no meetings folder) | n/a — already scaffolded |
| `prds/`, `plans/`, `research/`, `reference/` (per work folder) | Create on first use | When first artifact lands |
| `customers/accounts/{slug}/` | Build-on-demand unless `business_model == b2b` AND `customers[]` non-empty | First customer-specific artifact |
| `bug-investigations/` | Build-on-demand | First bug investigation |
| `sales-enablement/` | Build-on-demand unless `sales` in `team_functions` AND user opted in | First sales-enablement doc |
| `experiments/{feature-area}/` | Build-on-demand within `analytics/` | First experiment design |
| Sample-artifact files (PRD/RFC/plan stubs) | **Skip** — generate real via PMOS | If user explicitly opts in at Stage 3 Decision 11 |

**Why auto-scaffold `decisions/` and `meetings/`:** these are continuous-enrichment surfaces — every PMOS skill needs to know where to save. Empty `_index.md` is fine because the index file itself is the convention signal; it's not "dead context" per Hannah's NEVER #5, it's "structural skeleton." The trade-off: one extra file per work folder vs. every PMOS skill having to ask "where do these go?" every time.

**Why default-skip sample artifacts:** users typically prefer real PMOS-generated artifacts to hand-edited stubs. Stubs that linger as `[NOT YET FILLED]` placeholders are exactly the kind of dead context Hannah's NEVER #5 warns against.

---

## Audit + Migration doctrine (`--audit`, `--migrate`)

Both modes share the same audit core (the inventory + diff pass). `--audit` stops there; `--migrate` continues into propose + execute. The doctrine below covers both. The orchestrator-level steps live in SKILL.md → "Step 9.6 — Audit + Migration mode".

### Single source of truth (load-bearing rule)

After a migration, every piece of content lives in EXACTLY ONE folder. No mirroring, no symlinks to both, no "the real one is over here but I copied it over there too." The whole point of consolidation is to free Claude (and the user) from having to choose which copy to read.

This is what makes migration destructive: the source folder is moved to archive after a verified copy. Archive is a safety net for rollback, not a working location. **Folders in `04 Archive/` are explicitly NOT loaded as context** — every root CLAUDE.md should include this rule.

### When migration is needed (vs. a fresh L1 or a `--deepen`)

Migration is the right mode when:
- A folder existed before `/team-os-setup` ran (e.g., a Hannah-style PMOS instance, a hand-built knowledge folder, an old "01 Company" structure)
- The L1 built by the skill is mostly empty (pointer-only), and the user wants the real content TO LIVE in the L1 instead of being referenced from outside
- The user wants ONE canonical company folder for both context and project tracking, with the old folder safely archived

Migration is NOT the right mode when:
- The L1 is the canonical home and the source is a different concern (e.g., side project, personal notes) — keep them separate
- The source folder has artifacts you want to keep ACTIVELY referencing (transcripts, live code, datasets) — consider `--upgrade=L2` instead, which copies without archiving the source
- The user hasn't built an L1 yet — run `/team-os-setup` fresh first

### Audit pass (M1) — what to measure

The audit is a parallel-read pass: one sub-agent reads the L1, one reads the source. They report to the orchestrator, which synthesizes. Each agent captures the same things:

1. **Folder shape** — tree to depth 3, surfacing the architecture.
2. **Top-level files** — name, size, first ~80 chars of content (signals stub vs populated).
3. **Content density per area** — for each top-level folder: is it populated, stub, or empty? File count + total size.
4. **Live artifacts** — transcripts, code, trackers (`checkins.md`, `open-questions-log.md`, `stakeholder-tracker.md`), populated YAMLs. These are the things that matter to preserve.
5. **Convention markers** — does this folder have CLAUDE.md? `_index.md` files in `decisions/`/`meetings/`? Reading-order section? Routing tables?

The synthesis produces the **overlap matrix**: every top-level folder in L1 mapped to its conceptual analog in source, marked "thicker in A / thicker in B / unique." This is the load-bearing artifact — it makes the migration manifest writable.

### Consolidation target — naming heuristics

The target folder is the consolidated home. Its name follows these heuristics, in priority order:

1. **User-specified.** If the invocation included `--target=<name>`, use it.
2. **Established named folder wins.** If one side has an existing convention-named folder (e.g., `01 Expion/`) and the other is a generic build artifact (e.g., `expion-team-os-v1/`), use the established name with a doctrine suffix (e.g., `01 Expion - team-os/`). This preserves muscle memory + lineage.
3. **Lived-in folder wins.** If one side has live trackers + populated content and the other is a skeleton, adopt the live folder's name.
4. **Ask the user.** Two named folders both populated? Two plausible vault conventions? Surface the AskUserQuestion with both options.

The doctrine suffix (e.g., `- team-os`, `- pmos`, `- ops`) signals "this folder follows that doctrine" — useful when vault has multiple companies/teams each with their own OS.

### Archive doctrine — what archive is for

`04 Archive/` is the rollback-safety folder. Contents:
- A full copy of every migrated source folder, named `<basename> (archived-{YYYYMMDD})/`
- The `.git/` directory is REMOVED from archived copies if the source was its own repo (single source of truth for the repo itself)
- The archive is NEVER read by Claude as context — the root CLAUDE.md must declare this

Archive contents stay until the user decides they're safe to delete. The skill doesn't auto-prune.

### `.git` relocation strategy

Migration's trickiest piece: if the source folder is its own git repo (e.g., the source's `.git/` directory lives directly inside it, not in a parent), the repo identity must follow the canonical folder. Strategy:

- **Detection** (M0 pre-check): `git -C <source> rev-parse --show-toplevel` returns the source path itself → it's `own_repo`. Returns a parent path → it's `nested`. Fails → `not_a_repo`.
- **`own_repo` strategy:**
  1. Copy source content into L1 first (don't touch `.git` yet).
  2. Verify the archive copy of source is complete.
  3. Move `.git/` from source → L1 (atomic `mv`).
  4. Delete the now-`.git`-less source folder (its content lives in archive + L1).
  5. Rename L1 to target. The `.git` rides along.
  6. `git status` inside target now shows the working tree differs from the index (because the layout changed). Commit captures the reorganization.
- **`nested` strategy:** the source is already tracked by a parent repo. Use `git mv` for in-tree moves; no `.git` relocation needed. The reorganization commit happens in the parent repo.
- **`not_a_repo` strategy:** simplest case. Just copy, archive, rename, init the new repo if desired.
- **`submodule` strategy:** halt + ask the user. Submodule migration is not yet supported in v1.

### Why migration is its own mode (not just `--upgrade=L2`)

`--upgrade=L2/L3` copies/moves source files INTO an existing L1 in place. It does NOT:
- Rename the L1 folder
- Archive the source folder
- Handle `.git` relocation
- Produce a single-source-of-truth outcome (after `--upgrade`, both L1 and source still exist)

Migration is the superset that gets you to one canonical folder. Use `--upgrade` when you want both folders to coexist; use `--migrate` when you want consolidation.

---

## When this file gets updated

- A user's run reveals a default that's wrong for their org (Stage 3 outstanding-questions log surfaces it)
- Hannah publishes new guidance that changes a rule
- v1.next adds quick-start mode (decision-time defaults table grows a "quick-start preset" column)
- v2 hardens L2/L3 (adds rollback contract section)

Sub-skills should **not** edit this file mid-run. Updates land via PR review.

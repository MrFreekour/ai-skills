# {{company.name}}

This is the root index for {{company.name}}'s Team OS. Claude reads this file on every relevant query — at {{company.org_size_bracket}} ppl, keeping it lean is non-negotiable.

## Reading order for Claude (any task)

Load files in this order on any new session. Steps 1–4 are the **mandatory orientation pass** — read every time. Steps 5–9 are **scoped deep-reads** — only when the task touches that surface area.

1. **This file** (`CLAUDE.md` at the L1 root) — routing rules, glossary pointer, conventions.
2. **`_glossary.md`** — canonical terms (especially anything customer-hierarchy-shaped).
3. **The work-folder `CLAUDE.md`** for the launch folder — function/area-specific context, hot-path collaborators, current focus.
4. **The parent work-folder `CLAUDE.md`** if the launch folder is nested — context inherits up the tree.
5. **`decisions/_index.md`** in the launch folder — recent ADRs, reverse-chron.
6. **`meetings/_index.md`** in the launch folder — recent transcripts/summaries by date.
7. **Stakeholder lens** — `team.md` + `personas.md` (or whatever the team-routing tier surfaces).
8. **`goals.md`** — current quarter's goals if user mentions OKRs/quarterly work.
9. **Other `company/` files** — skim only; don't read in full unless the task explicitly references them.

When in doubt about where a file lives, walk the **PMOS routing rules** below before guessing.

## Where to find people

<!-- POINTER-ONLY TIER (50+ ppl): no team data inlined here at all. -->

**Full org directory:** `team/team-org.yaml` — every person, role, dept, reporting line, manager. **Always check there first** when a name comes up in a session. If someone's referenced but not yet listed, ask the user to add them.

Hot-path collaborators (the 3-5 people you work with daily) are inlined in the relevant function-area CLAUDE.md — see `product-development/{function}/CLAUDE.md`.

## What we work on

{{#each products}}
- **{{name}}** — {{one_line_description}}
{{/each}}

## Where things live

| Looking for… | Path |
|---|---|
| Product specs (PRDs) | `product-development/product/PRDs/{feature-area}/` |
| Engineering plans / RFCs | `product-development/engineering/{plans,rfcs}/{feature-area}/` |
| Analytics queries / schemas / dashboards | `product-development/analytics/{queries,schemas,dashboards}/{feature-area}/` |
| Customer accounts | `product-development/product/customers/accounts/{customer-slug}/` *(build on demand)* |
| Sprint planning / standup notes | `product-development/product/meetings/{sprint-planning, standup}/` |
| Onboarding guides | `team/onboarding-guides/` |
| Quarterly retros | `team/retros/` |
| **People, roles, reporting** | `team/team-org.yaml` |

**Cross-cutting feature lookup:** `product-development/feature-index.yaml` maps each feature to all its artifacts.

**Data warehouse tables:** `product-development/analytics/data-catalog.yaml`.

## PMOS output routing (skills must follow this)

When a PMOS / product-os skill (PRDs, decisions, plans, summaries, etc.) produces a file output, save it to the **most-specific work folder** relevant to the prompt content — not the launch folder, not the L1 root.

{{#each pmos_routing_rules.routes}}
- **{{work_scope}}** → `{{destination}}`{{#if ask_first}} *(skill must ask user before saving)*{{/if}}
{{/each}}

If unsure of the work scope, the skill must ask before saving. Default fallback: launch folder. Never default to user home or `~/Documents/`.

## Subfolder conventions (within any destination folder)

Once the routing rules pick a destination folder, this table tells the skill which subfolder to use within it. Skills create the subfolder on first use; same names repeat across every work folder.

| Artifact type | Subfolder | Filename pattern | Skills that produce this |
|---|---|---|---|
| Decision / ADR | `decisions/` | `YYMMDD-{slug}.md` | `decision-document-creator`, `decision-maker`, any decision-shaped skill |
| Meeting transcript / distilled summary | `meetings/` | `YYMMDD-{topic-slug}.md` | `granola-pull`, `onboarding-extraction`, `meeting-notes-processor` |
| PRD / spec | `prds/` (or `specs/` if the team uses that name) | `{feature-slug}.md` (living doc) or `YYMMDD-{slug}.md` (date-prefixed) | `prd-generator`, `problem-to-prd`, `pragmatic-mrd-generator` |
| Implementation plan | `plans/` | `{feature-slug}.md` | `spec-to-plan`, `feature-decomposition-tool` |
| Research / synthesis | `research/` | `YYMMDD-{topic-slug}.md` | `research-synthesis-engine`, `batch-interview-analysis` |
| Sample / template / reference | `reference/` | `{slug}.md` | (manual placement) |
| Generated outputs (workflows) | `output/` | (varies — workflow-spec defines) | workflow-execution skills |

**Each subfolder gets a `_index.md`** (reverse-chron table — date | title | file) when ≥3 entries land. Skills should append a row to `_index.md` when they save a new artifact.

**Versioning convention:** living docs (PRDs that get edited in place) follow `versioning_strategy.update_in_place`; date-prefixed files (decisions, ADRs, retros, synthesis docs) follow `versioning_strategy.date_prefixed`. See state.yaml or the Stage 2 versioning decision for the locked choice.

## Terminology

{{#each terminology}}
- **{{term}}** — {{definition}}
{{/each}}

## Communication

At {{company.org_size_bracket}} ppl, the Slack channel surface is too large to mirror here. Connect the Slack MCP for live channel context. Granola / Otter / Fireflies for meeting transcripts — these stay in their tool of origin.

## Working with Claude here

Function-area CLAUDE.md files carry the function-specific context. This root file is just the index — Claude descends into the right area when a query is function-scoped.

**Person lookups always go through `team/team-org.yaml`.** This is non-negotiable at this org size — inlining 100+ people in a CLAUDE.md would blow the thinking-room budget on every query.

## What does NOT live here

- Any team data (lives in `team/team-org.yaml`)
- Secrets / API keys
- Sprint goals, current PR links, this-week's-priorities
- Email / PRD / brief templates (use Claude commands instead)
- Multi-project mixing
- Dead context from finished projects

---

*Every line in a CLAUDE.md file costs thinking room. At {{company.org_size_bracket}} ppl, that math is brutal — keep this file lean.*

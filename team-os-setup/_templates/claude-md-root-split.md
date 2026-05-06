# {{company.name}}

This is the root index for {{company.name}}'s Team OS. Claude reads this file on every relevant query — keep it lean.

## Reading order for Claude (any task)

Load files in this order on any new session. Steps 1–4 are the **mandatory orientation pass** — read every time. Steps 5–9 are **scoped deep-reads** — only when the task touches that surface area.

1. **This file** (`CLAUDE.md` at the L1 root) — routing rules, glossary pointer, conventions.
2. **`_glossary.md`** — canonical terms.
3. **The work-folder `CLAUDE.md`** for the launch folder — function/area-specific context, hot-path collaborators, current focus.
4. **The parent work-folder `CLAUDE.md`** if the launch folder is nested.
5. **`decisions/_index.md`** in the launch folder.
6. **`meetings/_index.md`** in the launch folder.
7. **Stakeholder lens** — `team.md` + `personas.md` (or `team-org.yaml` for unfamiliar names).
8. **`goals.md`** — current quarter's goals when relevant.
9. **Other `company/` files** — skim only; don't read in full unless the task explicitly references them.

When in doubt about where a file lives, walk the **PMOS routing rules** below before guessing.

## Leadership & key contacts

<!-- SPLIT TIER (15-50 ppl): leadership + cross-functional anchors inlined; full team in team-org.yaml. -->

| Name | Role | Function | Slack |
|---|---|---|---|
{{#each leadership}}
| {{name}} | {{role}} | {{function}} | @{{slack_handle}} |
{{/each}}

**Full org directory:** `team/team-org.yaml` — every person, dept, reporting line. When Claude can't find someone referenced in a session, look them up there first; if missing, ask the user to add them.

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
| Full org / reporting lines | `team/team-org.yaml` |

**Cross-cutting feature lookup:** `product-development/feature-index.yaml` maps each feature to all its artifacts. Check it first when looking up anything that crosses functions.

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
| PRD / spec | `prds/` (or `specs/`) | `{feature-slug}.md` (living) or `YYMMDD-{slug}.md` (date-prefixed) | `prd-generator`, `problem-to-prd`, `pragmatic-mrd-generator` |
| Implementation plan | `plans/` | `{feature-slug}.md` | `spec-to-plan`, `feature-decomposition-tool` |
| Research / synthesis | `research/` | `YYMMDD-{topic-slug}.md` | `research-synthesis-engine`, `batch-interview-analysis` |
| Sample / template / reference | `reference/` | `{slug}.md` | (manual placement) |
| Generated outputs (workflows) | `output/` | (varies — workflow-spec defines) | workflow-execution skills |

**Each subfolder gets a `_index.md`** (reverse-chron table — date | title | file) when ≥3 entries land. Skills should append a row to `_index.md` when they save a new artifact.

## Terminology

{{#each terminology}}
- **{{term}}** — {{definition}}
{{/each}}

## Communication channels

{{#if mcp_slack_indexed}}
| Channel | Purpose |
|---|---|
{{#each slack_channels}}
| #{{name}} | {{purpose}} |
{{/each}}
{{else}}
*Slack channel map not indexed. Connect Slack MCP and re-run `/team-os-setup --refresh-channels` to populate.*
{{/if}}

## Working with Claude here

Function-area CLAUDE.md files (e.g., `product-development/product/CLAUDE.md`) carry the function-specific context. This root file is just the index — Claude descends into the right area when a query is function-scoped.

For team data: hot path lives in each function's CLAUDE.md (manager + 3-5 daily collaborators); the rest lives in `team/team-org.yaml`.

## What does NOT live here

- Secrets / API keys
- Sprint goals, current PR links, this-week's-priorities (stale within a session)
- Email / PRD / brief templates (use Claude commands instead)
- Multi-project mixing (each function gets its own CLAUDE.md)
- Dead context from finished projects

---

*Every line in a CLAUDE.md file costs thinking room. Keep it lean.*

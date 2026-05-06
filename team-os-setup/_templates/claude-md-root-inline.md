# {{company.name}}

This is the root index for {{company.name}}'s Team OS. Claude reads this file on every relevant query — keep it lean.

## Reading order for Claude (any task)

Load files in this order on any new session. Steps 1–4 are the **mandatory orientation pass** — read every time. Steps 5–9 are **scoped deep-reads** — only when the task touches that surface area.

1. **This file** (`CLAUDE.md` at the L1 root) — routing rules, team table, conventions.
2. **`_glossary.md`** if present — canonical terms.
3. **The work-folder `CLAUDE.md`** for the launch folder — area-specific context.
4. **The parent work-folder `CLAUDE.md`** if the launch folder is nested.
5. **`decisions/_index.md`** in the launch folder.
6. **`meetings/_index.md`** in the launch folder.
7. **Stakeholder lens** — `personas.md` (people are inlined here at this tier).
8. **`goals.md`** — current quarter's goals when relevant.
9. **Other root files** — skim only; don't read in full unless the task explicitly references them.

When in doubt about where a file lives, walk the **PMOS routing rules** below before guessing.

## Who's on the team

<!-- INLINE TIER (<15 ppl): full team listed here. -->

| Name | Role | Function | Slack |
|---|---|---|---|
{{#each team_members}}
| {{name}} | {{role}} | {{function}} | @{{slack_handle}} |
{{/each}}

For deeper org context — reporting lines, leadership, contractors — see `team/team-org.yaml`.

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

**Cross-cutting feature lookup:** `product-development/feature-index.yaml` maps each feature to all its artifacts (PRD, RFC, plan, schema, query, dashboard, tickets). Check it first when looking up anything that crosses functions.

**Data warehouse tables:** `product-development/analytics/data-catalog.yaml` documents every table — owner, refresh, upstream, downstream.

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
| Decision / ADR | `decisions/` | `YYMMDD-{slug}.md` | `decision-document-creator`, `decision-maker` |
| Meeting transcript / distilled summary | `meetings/` | `YYMMDD-{topic-slug}.md` | `granola-pull`, `onboarding-extraction`, `meeting-notes-processor` |
| PRD / spec | `prds/` (or `specs/`) | `{feature-slug}.md` or `YYMMDD-{slug}.md` | `prd-generator`, `problem-to-prd` |
| Implementation plan | `plans/` | `{feature-slug}.md` | `spec-to-plan`, `feature-decomposition-tool` |
| Research / synthesis | `research/` | `YYMMDD-{topic-slug}.md` | `research-synthesis-engine`, `batch-interview-analysis` |
| Sample / template / reference | `reference/` | `{slug}.md` | (manual placement) |

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

- Drop research files in `product-development/product/customers/accounts/{customer}/` if customer-specific.
- Drop interview transcripts as **distilled summaries** in `product-development/product/meetings/{type}/` — raw transcripts stay in their tool of origin (Granola, Otter, etc.); link out, don't paste in.
- When Claude can't find someone referenced in a session, ask the user to add them to `team/team-org.yaml` (only if `team-org.yaml` is in use; for inline tier, edit this file directly).

## What does NOT live here

- Secrets / API keys (use environment variables or 1Password)
- Sprint goals, current PR links, this-week's-priorities (these go stale within a session)
- Email / PRD / brief templates (use Claude commands instead)
- Dead context from finished projects (trim aggressively; commit history has the rest)

---

*This file is the index for {{company.name}}'s Team OS. Every line costs Claude's thinking room — keep it lean.*

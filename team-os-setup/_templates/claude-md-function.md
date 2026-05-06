# {{function.name_titlecase}} at {{company.name}}

This is the {{function.name}} function's index. Loaded when Claude operates inside `product-development/{{function.slug}}/`.

The primary purpose of this file: tell Claude which sub-folder to read for any natural-language query about {{function.name}}, so Claude doesn't waste context running explore agents. *"What you want in a CLAUDE.md is things you need on 80% of sessions."*

## Hot-path team (the people you work with daily)

<!-- For split + pointer-only tiers: 3-5 people max. For inline tier: this section can be omitted (root has full team). -->

| Name | Role | When to ping them |
|---|---|---|
{{#each hot_path_team}}
| {{name}} | {{role}} | {{when_to_ping}} |
{{/each}}

**For anyone else referenced in this session, look up `team/team-org.yaml` first.** If they're not there, ask the user to add them — capture name, role, function, dept, manager — and append the entry in-session.

## What this function owns

{{function.scope_description}}

{{#each pillars}}
- **{{name}}** — {{description}}
{{/each}}

<!--
  Pillars are 3-5 short capitalized phrases naming what this function is responsible for. Hannah's example product/CLAUDE.md uses "Five Core Pillars": Generation Quality, Developer Experience, Deployment, Collaboration, Enterprise. Each gets a one-line definition. Keep them short — these are scope reminders for Claude, not detailed strategy. If you don't have pillars yet, leave [NOT YET FILLED] and define them when the team next aligns.
-->


## Where things live (within {{function.slug}}/)

{{#each subfolders}}
- `{{path}}/` — {{purpose}}
{{/each}}

## Terminology specific to this function

{{#each function_terminology}}
- **{{term}}** — {{definition}}
{{/each}}

## Cross-references

- Cross-functional feature lookup → `../feature-index.yaml`
- Data tables → `../analytics/data-catalog.yaml`
- Other functions: {{#each sibling_functions}}`../{{slug}}/`{{#unless @last}}, {{/unless}}{{/each}}

## Conventions inside this function

{{#each conventions}}
- {{convention}}
{{/each}}

---

*Function-level CLAUDE.md. Keep it focused on what's specific to {{function.name}} — anything that applies across functions belongs in the root index.*

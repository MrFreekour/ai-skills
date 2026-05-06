# {{folder.name_titlecase}}

{{folder.one_line_purpose}}

## Doc index — start here

This is what this folder's CLAUDE.md is mostly for. Without this index, Claude runs explore agents to search the repo every time, burning context. The index points Claude directly to the right file in one read.

{{#each contents}}
- `{{filename}}` — {{description}}
{{/each}}

{{#if has_feature_areas}}
## Feature-area subfolders

Each feature area has its own subfolder. Same names repeat across the function tree (e.g., `billing/` exists in `PRDs/`, `plans/`, `rfcs/`, `queries/` — predictable for Claude to walk).

{{#each feature_areas}}
- `{{slug}}/` — {{description}}
{{/each}}
{{/if}}

## Naming convention

{{folder.naming_convention}}

## Cross-references

{{#each cross_refs}}
- {{label}}: `{{path}}`
{{/each}}

{{#if is_workflow_folder}}
## Workflow convention

When a folder is a workflow (multi-step repeatable process), the CLAUDE.md describes the *process* — what gets run, in what order, who runs it, on what cadence. The shape Hannah's example uses for `bi-weekly-update/`:

```
{workflow-name}/
├── CLAUDE.md          # this file — describes the process + cadence + ownership
├── workflow-spec.md   # detailed spec: inputs, outputs, success criteria
├── step-1-{slug}.md   # first step's instructions
├── step-2-{slug}.md   # second step
├── step-3-{slug}.md   # ...
├── output/            # where rendered outputs land (gitignored if large/temporal)
└── reference/         # supporting docs Claude reads while executing
```

This way Claude executes the workflow by reading CLAUDE.md → workflow-spec.md → step-1, step-2, step-3 in order. Outputs accumulate in `output/`; the workflow stays re-runnable.
{{/if}}

{{#if is_plans_folder}}
## Plan files are first-class artifacts here

Store complex multi-session plans here so the next Claude session can resume from them. Natural Claude plan files live in `~/.claude/` and are ephemeral (wiped 24–72hr); if a plan matters, copy it here when you finish it. (OpenAI's harness engineering team made plan files first-class artifacts in their shared repo for this same reason.)
{{/if}}

{{#if is_engineering_root}}
## Rollout discipline

Before any feature rollout: PRD updated (status + revision history), metric definitions written, queries documented, schemas linked, `feature-index.yaml` cross-referenced. Otherwise Claude works against context rot.

Bug investigations live at `bug-investigations/bug-{date}-{slug}/` (build-on-demand — create when first bug hits). Past investigations help the next person investigating a similar bug.
{{/if}}

{{#if is_analytics_root}}
## Rollout discipline

Before any feature rollout: metrics written, queries documented, schemas linked, dashboards referenced. The metric file lives in `metrics/{feature-area}/`, the queries in `queries/{feature-area}/`, the schemas in `schemas/{feature-area}/`. All three reference each other and connect back to the feature in `feature-index.yaml`.

Funnel-analysis playbooks and other analytics how-to docs live in `playbooks/`. When asked "why do users drop off during X?" Claude reads the matching playbook first.
{{/if}}

---

*Folder-level CLAUDE.md. Loaded when Claude descends into this folder — keep it scoped to what's specific here.*

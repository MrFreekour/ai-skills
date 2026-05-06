# Bi-weekly update — workflow

A repeatable workflow that produces a stakeholder-facing update every two weeks. Mirrors the shape Hannah's `team-os-example-repo` uses. Treat this as a worked example — rename, retime, and rewrite to match your team's actual cadence.

## Cadence

- **Frequency:** every other Friday (or your team's chosen day)
- **Owner:** {{first_feature.owner_slug}} — *([NOT YET FILLED] — set the actual owner)*
- **Audience:** [NOT YET FILLED — leadership, partner teams, exec staff?]
- **Time budget:** 30 minutes prep + 15 minutes review

## How to run

1. Read `workflow-spec.md` first — inputs, outputs, success criteria
2. Walk `step-1-pull-context.md` → `step-2-draft-summary.md` → `step-3-pick-highlights.md` → `step-4-format-and-send.md` in order
3. Outputs land in `output/{YYYY-MM-DD}.md`; previous outputs accumulate as the team's running record
4. Reference docs (audience expectations, format examples) live in `reference/`

## What this folder contains

- `workflow-spec.md` — full spec
- `step-1-pull-context.md` through `step-4-format-and-send.md` — instructions per step
- `output/` — rendered updates by date
- `reference/` — supporting context (sample updates, audience notes)

## When to update this workflow

- Cadence changes (bi-weekly → weekly, monthly): update this file + workflow-spec
- Audience changes: update `reference/audience-context.md`
- Format changes: update `step-4-format-and-send.md`

---

*Worked example seeded by `/team-os-setup`. Adapt to your team's actual rhythm.*

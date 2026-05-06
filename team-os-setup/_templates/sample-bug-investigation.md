# Bug investigation — {{sample_bug.short_description}}

**Investigator:** [NOT YET FILLED]
**Reported:** [NOT YET FILLED — date + reporter]
**Severity:** [P0 / P1 / P2 / P3]
**Status:** Investigating *([NOT YET FILLED] — choose: investigating / root-caused / fix-in-progress / shipped / wont-fix)*
**Lives in:** `product-development/engineering/bug-investigations/bug-{{today}}-{{sample_bug.slug}}/investigation-plan.md`

**Related artifacts:**
- Affected feature: `product-development/product/PRDs/{{products[0].slug}}/{{first_feature.slug}}.md`
- Tickets: [NOT YET FILLED]

---

## Symptom

What the user / system actually saw. Be precise. Include error messages verbatim, screenshots, request IDs, timestamps. Don't editorialize the symptom into a hypothesis.

⚠️ *Initial report — symptom may evolve as investigation continues.*

## Reproduction

Numbered steps. If it can't be reliably reproduced, say so explicitly.

1. [Step]
2. [Step]
3. [Observed: …]
4. [Expected: …]

**Reproducible:** [always / sometimes / never-reproduced-yet]

## Scope

What surface area is affected? Which users, which environments, which versions, which traffic share. Bound the blast radius before going deep.

[NOT YET FILLED]

## Infra touched

What systems / services / databases / external dependencies are in the call path. Cite the relevant repos or service names. The next person investigating a similar bug will start here to know where to look.

[NOT YET FILLED]

## Data examples + queries used

Concrete payloads, request/response samples, or rows from the affected tables. Plus the SQL queries that pulled the data above (link to `analytics/queries/{feature-area}/` or paste inline if one-off). This is the "show your work" section — without it, future investigators reproduce the analysis from scratch.

[NOT YET FILLED]

## Investigation log

Chronological. Append entries — don't edit prior ones (they're how the next person learns the shape of the problem).

### {{today}} — initial triage

- Reviewed [logs / traces / commits / dashboards]
- Ruled out: [what it's NOT]
- Current hypothesis: [leading theory]
- Next step: [what we're checking next]

### [Date] — [next session]

[Same shape]

## Root cause

Filled in once identified. One-paragraph explanation.

[NOT YET FILLED]

## Fix

- **Approach:** [what's being changed]
- **PR:** [link]
- **Migration / rollback:** [if needed]

## Prevention

What changes so this class of bug doesn't recur? Test added? Monitoring? Process change? Be specific — "more careful code review" is not prevention.

[NOT YET FILLED]

## Decision log

| Date | Decision | Rationale |
|---|---|---|
| {{today}} | [Severity assigned: …] | [Why] |

---

*Bug investigation template seeded by `/team-os-setup`. The folder lives at `engineering/bug-investigations/bug-{date}-{slug}/`; create when first bug investigation happens, not preemptively.*

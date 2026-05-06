# Implementation plan — {{first_feature.name}}

**Engineer:** [NOT YET FILLED]
**Status:** Draft *([NOT YET FILLED] — choose: draft / in-progress / shipped)*
**Last updated:** {{today}}
**Lives in:** `product-development/engineering/plans/{{products[0].slug}}/{{first_feature.slug}}.md`

**Related artifacts:**
- PRD: `product-development/product/PRDs/{{products[0].slug}}/{{first_feature.slug}}.md`
- RFC: `product-development/engineering/rfcs/{{products[0].slug}}/{{first_feature.slug}}-rfc.md`

---

## Goal

One sentence — what shipping this delivers. Pulled from the PRD, not re-litigated.

## Approach

Reference the RFC's accepted option. Don't redo the architectural reasoning — it's in the RFC.

## Work breakdown

Phases, in shippable order. Each phase must be independently mergeable (won't break main if shipped without later phases).

### Phase 1 — [scaffold name]

- [ ] [Task] — [estimated days]
- [ ] [Task]

**Done means:** [observable outcome — what's working at end of phase]
**Risk:** [biggest unknown that could blow the estimate]

### Phase 2 — [next scaffold]

[Same shape]

### Phase 3 — […]

[Same shape]

## Migrations / breaking changes

| Change | Backwards-compatible? | Rollout |
|---|---|---|
| [Schema change] | [yes / no — if no, describe migration] | [strategy] |
| [API change] | | |

If any row says "no" — flag in the PRD's risk section.

## Testing strategy

- **Unit tests:** [what's covered]
- **Integration tests:** [scenarios]
- **Manual QA:** [if needed; specify steps]
- **Production validation:** [post-deploy checks]

## Verification — how Claude (or you) self-verifies the work is done

Hannah's frame: Claude needs an explicit way to know the work is done and done well. Don't leave this implicit.

For each phase above, define what success looks like as something Claude can check:

- **Phase 1:** [e.g., "TypeScript compiles cleanly with no `any`-types added; `npm test` passes; new endpoint responds with the expected shape"]
- **Phase 2:** [...]
- **Phase 3:** [...]

For research-heavy plans: cite sources/URLs in the output. For UI plans: hook up Playwright MCP and verify in a loop. For data plans: validate row counts or distribution properties match the spec.

[NOT YET FILLED — fill verification per phase before kicking off the build]

## Tickets

[NOT YET FILLED — list of tracker tickets, mirroring `feature-index.yaml`]

- [TICKET-ID] — [Phase 1 work]
- [TICKET-ID] — [Phase 2 work]

## Open questions

Block the implementation. Owner closes.

- [ ] [Question] — owner:

## Revision history

- **{{today}} v1** — initial draft

---

*Plan template seeded by `/team-os-setup`. Plans cover the "how" — phases, tickets, migrations. The "why" lives in the PRD; the "which approach" lives in the RFC.*

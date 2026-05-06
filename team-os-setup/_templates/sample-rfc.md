# RFC — {{first_feature.name}} architecture

**Author:** [NOT YET FILLED]
**Status:** Draft *([NOT YET FILLED] — choose: draft / under-review / accepted / rejected / superseded)*
**Reviewers:** [NOT YET FILLED — names from team-org.yaml]
**Last updated:** {{today}}
**Lives in:** `product-development/engineering/rfcs/{{products[0].slug}}/{{first_feature.slug}}-rfc.md`

**Related artifacts:**
- PRD: `product-development/product/PRDs/{{products[0].slug}}/{{first_feature.slug}}.md`
- Implementation plan: `product-development/engineering/plans/{{products[0].slug}}/{{first_feature.slug}}.md` *(written after this RFC is accepted)*

---

## Decision to make

One-sentence framing of the architectural call this RFC makes. Examples:
- "Should we extend the existing billing service or build a new credit-ledger service?"
- "Sync vs. async for the credit-burn webhook?"
- "How do we partition the credit_transactions table?"

If this RFC tries to answer more than one decision, split it.

## Context

What you need to know to evaluate the options. Keep it tight — link to the PRD for product context, link to existing schemas/code for technical context. Don't repeat what's in those docs.

⚠️ *Inferred — confirm with the engineering lead before relying on this section.*

## Options considered

### Option A — [name]

**How it works:** [1-paragraph sketch]
**Pros:** [bullets]
**Cons:** [bullets]
**Effort:** [t-shirt size or person-weeks]

### Option B — [name]

[Same shape]

### Option C — [name, if applicable]

[Same shape]

## Recommendation

**Option [X], because [load-bearing reason].**

Trade-off accepted: [the cost of choosing X over Y/Z, stated honestly].

## What this changes

- [Schema migrations needed]
- [New services / dependencies]
- [Backwards-compatibility implications]
- [Rollout plan summary — full plan goes in the implementation doc]

## What this does NOT change

Useful for reviewers who'd otherwise ask. Bullet list.

## Open questions

Block the implementation but not the RFC.

- [ ] [Question] — owner:

## Decision log

| Date | Discussion | Outcome |
|---|---|---|
| {{today}} | [Initial RFC posted] | [Status] |

## Revision history

- **{{today}} v1** — initial draft

---

*RFC template seeded by `/team-os-setup`. RFCs document architectural decisions and the reasoning behind them. Implementation details belong in the matching plan doc, not here.*

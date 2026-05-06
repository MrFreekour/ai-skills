# PRD — {{first_feature.name}}

**Owner:** {{first_feature.owner_name}}
**Status:** Proposed *([NOT YET FILLED] — choose: proposed / in-progress / shipped / sunset)*
**Last updated:** {{today}}
**Lives in:** `product-development/product/PRDs/{{products[0].slug}}/{{first_feature.slug}}.md`

---

## Problem

[NOT YET FILLED — what user/customer problem does this feature solve? Cite evidence: support tickets, sales-call patterns, retention data, customer interviews. Hannah's principle: if you can't name the user pain in one sentence, you don't have a PRD yet — you have an idea.]

⚠️ *Inferred from product context — confirm with the customer-facing team before building.*

## Why now

[NOT YET FILLED — what changed? New competitor move, customer threshold reached, infra finally enabling it, regulatory deadline, etc. If "why now" is "we just thought of it," reconsider.]

## Who it's for

[NOT YET FILLED — name the persona. Use customer language from your personas doc, not PM jargon. "Operations leads at 50-200 person companies running multi-product portfolios" beats "users".]

## What we're building

A short bullet list. Each bullet is one capability the feature delivers. Resist the urge to write paragraphs here — the detail goes in the engineering plan.

- [Capability 1 — what the user can do that they couldn't before]
- [Capability 2]
- [Capability 3]

## What we're NOT building (out of scope)

Equally important. Bullet list. If a stakeholder asks "what about X?" and X is intentionally excluded, list it here so the answer is fast.

- [Out-of-scope item 1 — and why deferred]
- [Out-of-scope item 2]

## Success metrics

| Metric | Current | Target | Measurement |
|---|---|---|---|
| [Primary] | [baseline] | [target] | [how + cadence] |
| [Secondary] | | | |
| [Guardrail — what we don't want to break] | | | |

Cross-reference: dashboard + queries land in `product-development/analytics/` once shipped. Add to `feature-index.yaml` at that point.

## Open questions

Things that don't block the spec but block the build. Owner = the person who closes the question.

- [ ] [Question] — owner: {{first_feature.owner_slug}}
- [ ] [Question] — owner:

## Decision log

When stakeholders push back, capture the resolution here so future readers don't relitigate it.

| Date | Question | Decision | Made by |
|---|---|---|---|
| {{today}} | [Initial scope] | [Chose A over B because…] | [name] |

## Revision history

- **{{today}} v1** — initial draft (this version)

---

*PRD template seeded by `/team-os-setup`. Edit in place; append to revision history when you ship a meaningful change. For multi-version handling, see `_principles.md` versioning section.*

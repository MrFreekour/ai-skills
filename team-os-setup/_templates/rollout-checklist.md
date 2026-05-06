# Rollout checklist — {{company.name}}

Hannah's rule: a feature isn't shipped until the repo is updated. Otherwise the team works against context rot — Claude runs the next session with stale information and gives subtly wrong answers nobody notices.

This file is the team's gate before any feature goes live. Run through it every time. It's intentionally short — if a checkbox doesn't apply to your feature, write *N/A: {reason}* next to it; don't leave it ambiguous.

---

## Per-feature pre-rollout checklist

Copy this block and fill in for each feature shipping. Owner = the PM (or whoever drove the feature).

```
Feature: [name]
Owner: [name]
Target ship date: [YYYY-MM-DD]

### Product
- [ ] PRD updated to status: shipped, with revision history line for the launch
- [ ] PRD's "Success metrics" table reviewed against actual instrumentation

### Engineering
- [ ] RFC updated to status: accepted (or superseded if it changed during build)
- [ ] Implementation plan updated to status: shipped
- [ ] Any architecture changes reflected in the RFC; any new services documented in `engineering/CLAUDE.md`

### Analytics
- [ ] Metric definitions written for this feature in `analytics/metrics/{feature-area}/`
- [ ] Queries backing those metrics documented in `analytics/queries/{feature-area}/`
- [ ] Schemas for any new tables documented in `analytics/schemas/{feature-area}/`
- [ ] Dashboard linked or noted in the metrics file
- [ ] Any experiments planned for the feature added to `analytics/experiments/{feature-area}/`

### Cross-functional traceability
- [ ] `feature-index.yaml` entry created/updated — PRD path + RFC path + plan path + tickets + figma + schemas + queries + dashboard all referenced

### Customers
- [ ] If named-customer-relevant: added a note to the relevant `customers/accounts/{slug}/CLAUDE.md` files about the change

### Communication
- [ ] Launch email drafted in `launch-emails/`
- [ ] Sales enablement updated if the feature affects how we sell
- [ ] Slack announcement scheduled for the right channel
```

---

## How to use this

- **Before any feature ships:** the owner walks the checklist with the team. Each item gets checked or marked N/A with a reason.
- **The CHECKLIST IS THE GATE.** If items are open, the feature doesn't ship — the OS isn't ready, which means the next session will work against bad context.
- **Tailor over time.** Some teams add more (legal review, support enablement, security sign-off); some teams remove items that don't apply. The checklist evolves.

---

*Rollout-checklist artifact, scaffolded by `/team-os-setup`. Hannah's framing: "everyone is an owner of our knowledge repository." This is the gate that keeps the OS healthy.*

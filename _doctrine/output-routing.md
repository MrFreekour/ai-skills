# Output Routing Doctrine — PMOS / product-os skills

**Scope:** Every skill under `product-os/skills/` and any general skill that produces a file output.

---

## The rule

> **Skills that produce file outputs MUST read `CLAUDE.md` from the launch folder and walk parent `CLAUDE.md` files up to the L1 root or repo root. If a routing table or subfolder-conventions table is found in any of those files, follow it. If unsure of work scope, ask before saving.**

This is the only piece of cross-skill governance that makes the team-os-setup-built routing rules actually load-bearing. Without it, the routing table in the root `CLAUDE.md` is decorative — every skill writes wherever its author hardcoded the path.

---

## Why this matters

When a user runs `/team-os-setup`, Stage 4 generates a root `CLAUDE.md` containing:

1. **PMOS output routing** — a table mapping work-scope (e.g., "Billing — PRDs and dashboards") to a destination folder (e.g., `product-development/product/PRDs/billing/`).
2. **Subfolder conventions** — a table mapping artifact type (decision, meeting, PRD, plan, research) to the subfolder name (`decisions/`, `meetings/`, etc.).

For these tables to do their job, every PMOS skill needs to:
1. Locate the routing tables.
2. Pick the most-specific work-scope match.
3. Pick the right subfolder for the artifact type.
4. Save the file there — not to the launch folder, not to user home, not to `~/Documents/`.

This gap surfaces real-world: when a user asks *"how would `/granola-pull`, `/onboarding-extraction`, `/enhance-context` know to use the `meetings/` folder?"* — the honest answer is "they wouldn't, unless they walk CLAUDE.md routing rules first." This doctrine is what makes them.

---

## The 4-step protocol

Every PMOS skill that writes a file output MUST follow this sequence:

### Step 1 — Locate routing rules

Walk from the launch folder up to the repo root, reading `CLAUDE.md` (or `AGENTS.md`) at each level. Stop when you find the L1 root (a CLAUDE.md containing a `## PMOS output routing` heading) or hit the repo root.

If no routing rules are found in any walked CLAUDE.md → **fall back to launch folder** + ask user to confirm before saving. Never default to user home, `~/Documents/`, or hardcoded paths.

### Step 2 — Pick the work-scope match

From the routing table found in Step 1, pick the row whose `work_scope` description most closely matches the prompt content. Use these heuristics:

- **Most-specific match wins.** If the prompt is about "Billing — credit-burn dashboard" and the table has both "Billing" and "Analytics team-level," pick the more specific Billing route.
- **Cross-cutting routes have `ask_first: true`** — if the user-facing scope is ambiguous (e.g., "company-level" or "team-level"), ask the user before saving.
- **No match** → ask the user: *"This doesn't fit any routed work scope. Save to {launch folder} or pick a different folder?"*

### Step 3 — Pick the subfolder

From the **subfolder conventions table** (also in the L1 root CLAUDE.md), pick the row matching the artifact type:

| Artifact you produce | Subfolder | Filename pattern |
|---|---|---|
| Decision / ADR | `decisions/` | `YYMMDD-{slug}.md` |
| Meeting transcript / distilled summary | `meetings/` | `YYMMDD-{topic-slug}.md` |
| PRD / spec | `prds/` (or `specs/`) | `{feature-slug}.md` (living) or `YYMMDD-{slug}.md` (date-prefixed) |
| Implementation plan | `plans/` | `{feature-slug}.md` |
| Research / synthesis | `research/` | `YYMMDD-{topic-slug}.md` |
| Sample / template / reference | `reference/` | `{slug}.md` |

If the subfolder doesn't exist yet, **create it** — that's the established pattern (per `state.yaml: pmos_routing_rules.subfolder_creation`).

### Step 4 — Save + index

Save to `{destination}/{subfolder}/{filename}`.

If `{destination}/{subfolder}/_index.md` exists, **append a row** to the reverse-chron table:

```markdown
| {YYMMDD} | {Title or one-line description} | [{filename}](./{filename}) |
```

If the index doesn't exist yet but the subfolder has ≥3 entries after this save, prompt the user: *"`{subfolder}/_index.md` doesn't exist. Want me to create it now?"*

---

## What "ask before saving" looks like

Use the `AskUserQuestion` tool, never inline prose:

```
Header: "Save destination"
Description: "I produced a {{artifact_type}}. Routing rules suggest `{{matched_destination}}` — confirm?"

Options:
  - "Yes — save to {{matched_destination}}/{{subfolder}}/" (Recommended)
  - "Save to launch folder ({{launch_folder}}) instead"
  - "Pick a different folder" (free-text via Other)
```

---

## Audit checklist for existing PMOS skills

This doctrine ships with an expectation that authors of existing skills audit them for compliance. Sample 5–10 skills across product-os/skills/ and verify:

- [ ] The skill reads `CLAUDE.md` from the launch folder before saving.
- [ ] If it can't find routing rules, it asks (rather than hardcoding a path).
- [ ] It uses the subfolder convention (`decisions/`, `meetings/`, etc.) when applicable.
- [ ] It appends to `_index.md` when present.
- [ ] It does NOT default to user home, `~/Documents/`, or any path outside the launch folder's git tree.

Skills to prioritize for first audit (high-volume output producers):
- `decision-document-creator`
- `prd-generator`
- `problem-to-prd`
- `meeting-notes-processor`
- `research-synthesis-engine`
- `enhance-context` (already audited — uses launch-folder context)
- `onboarding-extraction`
- `spec-to-plan`

---

## Long-term: shared `_routing.md` partial

Eventually every PMOS skill should `@import` or reference a shared `_routing.md` partial that encodes this protocol once. For v1 this doctrine document is the source of truth; skill authors copy the relevant lines into their SKILL.md.

When the partial lands, this doctrine doc gets demoted to a "why we do this" companion and the partial becomes the operational source.

---

## Why this exists

Different PMOS skills have different output-saving conventions out of the box: some save to the launch folder, some prompt for a path, some default to `~/Documents/`. The routing rules in `CLAUDE.md` only work if every skill respects them — this doctrine is the protocol that makes it consistent.

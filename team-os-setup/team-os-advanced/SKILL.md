---
name: team-os-advanced
description: Stage 3 of the team-os-setup chain. Walks the user through 5–10 architecture decisions raised by Stage 2 — meeting transcripts placement, customer accounts depth, bug investigation cadence, etc. Each decision: explain Hannah's principle, show trade-off, recommend default, AskUserQuestion to confirm or override. Mid-stage checkpointing means a crash loses ≤1 decision.
---

# Stage 3 — Advanced (decision walk)

You are Stage 3 of the `team-os-setup` chain. By the time you run, the user has approved a high-level proposal at Gate 2. Your job: walk them through the 5–10 decisions that shape the L1 build, finalize the blueprint, and surface any outstanding questions for the user's specific context.

This stage is the longest user-facing step. Mid-stage checkpointing matters here — Stage 1's question loop and your decision walk are the two places where users invest 10–20 minutes that we DON'T want to lose to a crash.

**Read first:**
- `_voice.md` — voice rules
- `_principles.md` — decision-time defaults table (rows 149–166)
- `<repo-root>/team-os-build/state.yaml` — full state from Stages 1 + 2
- `<repo-root>/team-os-build/research/proposal-YYYY-MM-DD.md` — Stage 2 proposal + decision queue

**You write:**
- `<repo-root>/team-os-build/research/blueprint-YYYY-MM-DD.md` — final blueprint
- Updates to `state.yaml`: `decisions.*`, `mid_stage_checkpoint`, `gates_passed[2]`, `outstanding_questions_log`

---

## Step 1 — Read inputs

Load:
- `state.yaml` (note: read `quick_mode` and `variant` flags set at preflight)
- Stage 2 proposal (especially the "Decision queue for Stage 3" section — 5–10 numbered open questions)
- `_principles.md` decision-time defaults table

**Resume check:** if `state.yaml: mid_stage_checkpoint.stage == 3`, ask user: *"Found prior progress at decision #{last_decision_resolved + 1}. Resume from there, or restart Stage 3?"*

### Mode branching

This stage has two mode-driven behavior changes:

- **`state.yaml: quick_mode == true`** → Step 3 ("Walk decisions one at a time") is replaced with a single auto-apply pass per Step 3.5 below. No per-decision AskUserQuestion. The user lands directly at Gate 3 with a resolved blueprint and an "iterate any decision?" affordance.
- **`state.yaml: variant == "personal"`** → the active queue (Step 2) is pruned harder than the default. Drop:
  - Decision 4 (Onboarding fan-out) — solo work has no onboarding-fan-out target
  - Decision 9 (Sales enablement) — irrelevant for indie / personal-projects
  - Decision 11 (Sample artifacts) — already default-skip
  - Decision 6 (Design integration) — keep only if user explicitly named design work in Stage 1's source pointers
  - Decision 2 (Customer accounts depth) — replaced upstream in Stage 1 by the lighter "audience" question; skip here.

  **Keep Decision 10 (Retros cadence)** — solo project retros + self-retros are valuable. Default still applies: per-quarter folder.

  The remaining queue for `variant=personal` is typically: 1 (meetings), 3 (bug investigations — relevant if user codes), 5 (strategy folder shape — useful for multi-project planning), 7 (data catalog — only if user does analytics for their projects), 8 (workflows + archetype), 10 (retros cadence), 12 (rollout checklist — keep). Active queue size: 5–7 decisions.

The mode branches compose: `--quick --variant=personal` runs the personal-pruned queue with auto-applied recommended defaults, lands at Gate 3 with everything resolved.

---

## Step 2 — Build the active decision queue

Start from the Stage 2 queue. Apply pruning rules:

| Skip a decision if… | Decision dropped |
|---|---|
| Stage 1 said "no sales function" | Sales enablement folder |
| Stage 1 said `customers` is empty / pre-revenue | Customer accounts depth (default: build on demand, no question) |
| Stage 1 multi-product = "single" + only one team function | Strategy folder shape (default: full) |
| `analytics` not in team_functions | Data catalog scope (skip — no `analytics/` folder generated) |
| `design` not in team_functions | Design integration (skip) |

Default queue (12 decisions max, after pruning):

1. **Meeting transcripts placement** *(gap question — see exact shape below)*
2. **Customer accounts depth** — only if customers list is non-empty
3. **Bug investigation cadence**
4. **Onboarding guide fan-out**
5. **Strategy folder shape**
6. **Design integration** — only if `design` in team_functions
7. **Data catalog scope** — only if `analytics` in team_functions
8. **Workflows folder + archetype**
9. **Sales enablement folder** — only if `sales` in team_functions
10. **Retros cadence**
11. **Sample artifacts at L1 build** *(default: skip; see exact shape below)*
12. **Rollout checklist artifact** *(see exact shape below)*

---

## Step 3 — Walk decisions one at a time

**Skip this step entirely if `quick_mode == true`** — jump to Step 3.5.



**Stage opener** (print once before the first decision):

> *This is the substantive stage — the architecture decisions Hannah's framework leaves to your team's judgment. Take as long as you need on each. Ask me to clarify or expand whenever it helps; if the recommended default feels wrong for your situation, push back and we'll adjust. The goal isn't speed, it's getting the OS right for your team.*

For each decision in the active queue:

1. **Read the principle** (1–2 sentences from `_principles.md` or Hannah's published guidance, with verbatim quote where it fits)
2. **State the trade-off** (1 sentence — what you gain vs. what you give up)
3. **Show 2–3 options** with consequences, plus a "diagnostic" option (only on the multi-product / versioning / meetings gap questions — not all 10)
4. **Mark recommended default**
5. **AskUserQuestion** — confirm or override

After each decision: write `state.yaml: mid_stage_checkpoint.last_decision_resolved` and `decisions.<key>`. **Atomic write.** This is the "≤1 decision lost on crash" guarantee.

### Decision template (apply to each)

```
Header: "{One-sentence question framing this decision}"
Description: "{Hannah's principle, 1-2 sentences} — {trade-off, 1 sentence}."

Options:
  - "{Option A — recommended default}" [RECOMMENDED]
  - "{Option B}"
  - "{Option C, if relevant}"
  - "{Diagnostic option}" (only on gap questions)
```

---

## Step 3.5 — Auto-apply (when `quick_mode == true`)

Walk the active queue (post-pruning per Step 2). For each decision, set `decisions.<key>` to the recommended default from the decision's "exact shape" in Step 4 (the option marked `[RECOMMENDED]`). Do NOT call AskUserQuestion. Set `decisions.<key>.rationale = "auto-applied via --quick mode"`.

Atomic write `state.yaml: mid_stage_checkpoint.last_decision_resolved` after each one (same checkpointing as the manual walk — keeps resume mechanics consistent).

After the queue is fully resolved, jump straight to Step 6 (Generate the blueprint) — Step 5 (closing outstanding-questions prompt) is also skipped in quick mode (the closing prompt is meant for users who walked the decisions in detail; quick-mode users explicitly opted into "just give me the defaults"). Gate 3 (Step 7) gives them an iteration affordance if they want to go back.

**Recommended defaults by decision** (used by the auto-apply):

| # | Decision | Recommended default |
|---|---|---|
| 1 | Meetings placement | "Meetings folder + distilled summaries" |
| 2 | Customer accounts depth | "Build on demand" |
| 3 | Bug investigation cadence | "Build on demand" |
| 4 | Onboarding guide fan-out | "General + per-role" |
| 5 | Strategy folder shape | "Full sub-tree (business-context, plans, roadmaps, vision)" |
| 6 | Design integration | "CLAUDE.md only with Figma links" |
| 7 | Data catalog scope | "Warehouse tables only" |
| 8 | Workflows folder | "Yes — processes/ + workflows/ separation"; archetype: `bi-weekly-update` |
| 9 | Sales enablement folder | "Build on demand" |
| 10 | Retros cadence | "Per-quarter folder" |
| 11 | Sample artifacts at L1 build | "Skip — generate real via PMOS" |
| 12 | Rollout checklist artifact | "Yes" if `analytics` in team_functions OR `customers` non-empty, else "Skip" |

These mirror the decision-time defaults table in `_principles.md`; if that table changes, this one stays in sync (the `[RECOMMENDED]` markers in Step 4's exact shapes are the source of truth).

---

## Step 4 — The 10 decisions (exact shapes)

### Decision 1 — Meeting transcripts placement (GAP question)

```
Header: "Where do meeting transcripts live?"
Description: "Hannah's principle: 'The fuller it gets, the less thinking room your junior employee has.' Raw transcripts are HIGH volume, LOW signal density — they're context noise unless distilled. The question isn't really *where* — it's *how to handle them so Claude can use them without choking*."

Options:
  - "Meetings folder + distilled summaries" [RECOMMENDED]
    → product/meetings/{sprint-planning, standup, team-bi-weekly}/ holds short summary docs only (decisions, action items, owners). Raw transcripts stay in their tool of origin (Granola, Otter, Fireflies); link out, don't paste in.
  - "Meetings folder + full transcripts"
    → Same shape, raw transcripts pasted. Heavier on context budget. Only if your meetings tool doesn't link well.
  - "No meetings folder — decisions go to feature folders"
    → Sprint-planning decisions land in the relevant PRD's revision history; standup blockers land in the relevant engineering plan. Most context-efficient; loses the cross-feature meeting view.
  - "Help me figure out which fits — diagnostic"
```

**Diagnostic path:** if user picks diagnostic, ask 2–3 follow-ups:
- "Do you use a meetings tool with API access (Granola, Otter, Fireflies)?"
- "Do you want Claude to read raw transcripts, or just decisions?"
- "How often do you reference past meetings — weekly, monthly, rarely?"

Then recommend based on answers + offer to confirm or pick another.

Save to `state.yaml: decisions.meetings.choice` (and `.rationale`).

### Decision 2 — Customer accounts depth

Only ask if `state.yaml: customers` is non-empty.

```
Header: "How deep should customer-account folders go?"
Description: "Hannah's example puts each named customer in `product/customers/accounts/{slug}/`. The question is what goes inside that folder."

Options:
  - "Build on demand" [RECOMMENDED]
    → Skip the folder for now. Add `customers/accounts/{slug}/` when you have a real customer-specific artifact to drop in. CLAUDE.md note explains the pattern.
  - "Flat — one CLAUDE.md per customer"
    → product/customers/accounts/{customer-slug}/CLAUDE.md (1 file per customer, account snapshot).
  - "Sub-tree per customer"
    → product/customers/accounts/{customer-slug}/{summaries, transcripts, contracts, research}/. Heavier; only useful if you have 5+ artifacts per customer already. Note: the canonical skill paired with this depth is `customer-call-summary` (Hannah's example ships a 6-section summary spec) — auto-scaffolding the skill is parked for v1.next; for now, write your own and Claude picks it up.
```

### Decision 3 — Bug investigation cadence

```
Header: "When does the bug-investigations/ folder get created?"
Description: "Hannah's pattern is `engineering/bug-investigations/bug-{date}-{slug}/`. Question: scaffold the folder now (empty), or only when the first bug investigation happens?"

Options:
  - "Build on demand" [RECOMMENDED]
    → No folder at L1. CLAUDE.md note in engineering/ explains the pattern. Create when first bug hits.
  - "Always — empty folder + .gitkeep"
    → Scaffold now. Risk: empty folder is dead context (NEVER #5). Useful only if you expect bugs in the first week.
```

### Decision 4 — Onboarding guide fan-out

```
Header: "How granular should onboarding guides be?"
Description: "team/onboarding-guides/ holds the docs you give new hires. Hannah's example has both a general guide and per-role guides — plus an interactive `onboarding.md` agent that walks new hires through the relevant docs (the agent is v1.next companion work; the guides ship in v1)."

Options:
  - "General + per-role" [RECOMMENDED]
    → Mirrors Hannah's example. team/onboarding-guides/{general, pm, eng, design, ...}.md. Roles match your team_functions.
  - "General only"
    → Single team/onboarding-guides/general.md. Add per-role guides later if hiring concentrates.
  - "Skip — no onboarding folder for now"
    → Build on demand when first hire is incoming.
```

### Decision 5 — Strategy folder shape

```
Header: "How structured should the strategy folder be?"
Description: "Hannah's example has product/strategy/{business-context, plans, roadmaps, vision}/. Question: full sub-tree or flatter?"

Options:
  - "Full sub-tree" [RECOMMENDED]
    → strategy/{business-context, plans, roadmaps, vision}/. Each gets its own CLAUDE.md note.
  - "Flat"
    → strategy/ with no subfolders. Add structure when you have 5+ docs.
  - "Skip — no strategy folder yet"
    → Build on demand.
```

### Decision 6 — Design integration

Only ask if `design` in team_functions.

```
Header: "How does design integrate with the repo?"
Description: "Hannah's example puts design specs in Figma and uses product-development/design/CLAUDE.md as a pointer-only index — no design files in the repo."

Options:
  - "CLAUDE.md only with Figma links" [RECOMMENDED]
    → design/CLAUDE.md indexes Figma URLs by feature. Keeps the repo lean; design tool stays source of truth.
  - "Pull design specs into the repo"
    → design/{feature-area}/ holds exported Figma frames as PNG/SVG + spec docs. Heavier; only if Figma access is restricted.
```

### Decision 7 — Data catalog scope

Only ask if `analytics` in team_functions.

```
Header: "What goes in data-catalog.yaml?"
Description: "Hannah's pattern is data-catalog.yaml = warehouse tables + their metadata. Event schemas (analytics events) are separate concern."

Options:
  - "Warehouse tables only" [RECOMMENDED]
    → data-catalog.yaml documents tables. Event schemas land in analytics/schemas/{feature-area}/ as separate docs.
  - "Warehouse tables + event schemas merged"
    → Both in data-catalog.yaml. Heavier file; useful if your team treats them as one concern.
```

### Decision 8 — Workflows folder + archetype to seed

Two-part decision. First the structure:

```
Header: "Workflows folder — yes or no?"
Description: "Hannah's example has product/processes/ AND product/workflows/ as separate folders. Processes = how the team works; workflows = named multi-step procedures with the shape: CLAUDE.md (process + cadence) + workflow-spec.md + step-N-{slug}.md files + output/ + reference/. Hannah ships `bi-weekly-update/` as a worked example."

Options:
  - "Yes — processes/ + workflows/ separation" [RECOMMENDED]
    → Mirrors Hannah's pattern. Each workflow gets the multi-file shape; each process gets a single doc.
  - "No — fold both into processes/"
    → Single processes/ folder. Less structure; useful for smaller teams.
  - "Skip — build on demand"
    → No folder until first process is documented.
```

If user picks "Yes" or "fold into processes/", ask the second part — which archetype to seed:

```
Header: "Pick a workflow archetype to seed as a worked example:"
Description: "Stage 4 will drop a fully-fleshed sample workflow into your repo so the shape is self-documenting. Rename and rewrite it to match your team's actual cadence — it's a starting point, not a final."

Options:
  - "bi-weekly-update — recurring stakeholder update" [RECOMMENDED — modeled on Hannah's bi-weekly-update example shape]
    → workflows/bi-weekly-update/ with 4 step files (pull-context → draft → pick-highlights → format-and-send) + workflow-spec.md + output/ + reference/. Best fit if your team has a regular leadership update cadence.
  - "feature-launch — pre+post launch readiness"
    → workflows/feature-launch/ with stub steps for pre-launch checklist + post-launch metrics review. Best fit if you ship multiple features per quarter.
  - "quarterly-planning — full quarterly planning cycle"
    → workflows/quarterly-planning/ with stub steps for retro → strategy review → objectives draft → roadmap proposal. Best fit if your team owns strategic planning end-to-end.
  - "Skip — empty workflows/ folder, no archetype"
    → Just the folder + a CLAUDE.md note. You build your own when ready.
```

Save to `state.yaml: decisions.workflows_folder.choice` and `state.yaml: decisions.workflows_folder.archetype` (`bi-weekly-update` | `feature-launch` | `quarterly-planning` | `none`).

### Decision 9 — Sales enablement folder

Only ask if `sales` in team_functions.

```
Header: "Sales enablement folder — yes or no?"
Description: "Hannah's example has product/sales-enablement/ for battle cards, demo scripts, objection handling. Useful when sales-PM collaboration is high; dead context when sales runs separately."

Options:
  - "Build on demand" [RECOMMENDED — even if sales function is present]
    → Add when first sales-enablement doc is written. CLAUDE.md note explains the pattern.
  - "Yes — scaffold the folder now"
    → Empty folder + CLAUDE.md placeholder.
```

### Decision 10 — Retros cadence

```
Header: "Retros cadence — how do you organize team/retros/?"
Description: "Hannah's example has team/retros/ with a .gitkeep — fills over time. Question: per-sprint folders, per-quarter folders, or flat?"

Options:
  - "Per-quarter folder" [RECOMMENDED]
    → team/retros/{2026-Q2, 2026-Q3, ...}/. Easy to find, prunable when too old.
  - "Per-sprint folder"
    → team/retros/{2026-04-15-sprint-23, ...}/. More granular; useful for fast-shipping teams.
  - "Flat"
    → All retros in team/retros/ with date-prefixed filenames. Simplest; can refactor later.
```

### Decision 11 — Sample artifacts at L1 build

```
Header: "Drop sample PRD/RFC/plan stubs at L1 build?"
Description: "Stubs demonstrate the routing pattern (PRDs land here, plans here, etc.) but they're stubs — most users prefer to generate real artifacts via PMOS skills (`/prd-generator`, `/spec-to-plan`) instead of hand-editing scaffolds. Default: skip stubs unless you specifically want them."

Options:
  - "Skip — generate real artifacts via PMOS skills" [RECOMMENDED]
  - "Yes — drop a sample PRD + RFC + plan personalized with my first feature"
  - "Partial — just one sample (e.g., PRD only) to demonstrate the pattern"
```

Save to `state.yaml: decisions.sample_artifacts.include` (`false` | `true` | `partial`) and `decisions.sample_artifacts.types` (subset of `[prd, rfc, plan]` if `partial`).

Stage 4 reads this in Step 8: if `include == false`, skip the parallel sub-agent dispatch entirely; if `partial`, dispatch only the chosen subset; if `true`, dispatch all 3. The Step 11 validation check for sample-artifact non-empty is conditional on `include != false`.

### Decision 12 — Rollout checklist artifact

```
Header: "Want a rollout checklist artifact at product/processes/rollout-checklist.md?"
Description: "Hannah's 'context rot' framing: a feature isn't shipped until the repo is updated. Without an explicit checklist, teams skip the discipline. We've already added rollout-discipline notes to engineering + analytics CLAUDE.mds — this is one step further: an explicit checkbox-style file the team walks before any rollout."

Options:
  - "Yes — scaffold rollout-checklist.md" [RECOMMENDED if you have analytics in your team_functions]
    → product/processes/rollout-checklist.md gets created with PRD/RFC/plan/metrics/queries/schemas/feature-index sections, all checkboxes. Hannah's discipline made explicit.
  - "Skip — keep rollout discipline as inline CLAUDE.md notes only"
    → Lighter scope. Notes still exist in engineering + analytics CLAUDE.mds. Team enforces discipline by convention, not artifact.
```

Default: **yes** if `analytics` is in `team_functions` OR `customers` list is non-empty. Save to `state.yaml: decisions.rollout_checklist.choice` (`yes` | `no`).

---

## Step 5 — Closing prompt for outstanding questions

After the decision queue is walked, AskUserQuestion:

```
Header: "Any other questions or use cases you want to lock down before we build?"
Description: "Free text — describe and I'll either fold it in or flag it as out-of-scope-for-v1. This goes in your blueprint as the 'open considerations' log."

Options:
  - "No, ready to build"
  - "Yes, I have something" (free-text input next)
```

If user provides free text: parse and respond:
- If you can map it to an existing decision or principle: tell them where it lands
- If it's a genuinely new question outside the scope of L1: log it to `state.yaml: outstanding_questions_log[]` with timestamp, and note in the blueprint as "Open consideration — for the user to revisit post-build"

---

## Step 6 — Generate the blueprint

Write `<repo-root>/team-os-build/research/blueprint-YYYY-MM-DD.md`:

```markdown
# Blueprint — {{company.name}} Team OS L1

**Generated:** {today}
**Routing tier:** {{company.team_routing_tier}}
**Multi-product mode:** {{decisions.multi_product.choice}}

---

## Final folder tree

[Render the proposal tree from Stage 2, modified by Stage 3 decisions — folders flagged as build-on-demand removed (replaced with placement-notes); folders the user opted into added.]

## Decisions resolved

| # | Decision | Choice | Rationale |
|---|---|---|---|
| 1 | Meeting transcripts | {{choice}} | {{rationale}} |
| 2 | ... | ... | ... |

## Folders flagged as "build on demand"

| Folder | Placement-note text (goes in parent CLAUDE.md) |
|---|---|
| `product-development/product/customers/` | "customers/ not yet scaffolded — create when you have a named customer to seed. Use customers/accounts/{customer-slug}/ pattern." |
| ... | ... |

## Sample-artifact list (for Stage 4)

Stage 4 will personalize and drop these:
- 1 sample PRD at: product-development/product/PRDs/{{products[0].slug}}/{{first_feature.slug}}.md
- 1 sample RFC at: product-development/engineering/rfcs/{{products[0].slug}}/{{first_feature.slug}}-rfc.md
- 1 sample plan at: product-development/engineering/plans/{{products[0].slug}}/{{first_feature.slug}}.md
- (No sample bug-investigation — that's build-on-demand)

Pre-fill candidates from research:
- Features: {{candidate_features_found list}}
- Tables: {{candidate_tables_found list}}

## L1 target path

{{target_l1_path}} *(user-confirmable in Stage 4)*

## Open considerations

[Log of `outstanding_questions_log` entries from Step 5, if any.]

---

*Blueprint generated by `team-os-advanced`. Stage 4 (`team-os-execute`) builds from this.*
```

---

## Step 7 — Gate 3 (final approval before build)

Print 1-paragraph summary: "{N} decisions resolved. Routing tier: {tier}. Build target: {path}. {M} folders flagged build-on-demand. {K} candidate features found in research."

AskUserQuestion:

```
Header: "Blueprint resolved with {N} decisions. Approve to build L1?"
Options:
  - "Approve and build L1"
  - "Iterate on decision #X" (multi-select from the 10 decisions; re-asks just those)
  - "Stop here"
```

On **approve:**
- Write `state.yaml: gates_passed[2]` (timestamp, commit_sha=null)
- Write `state.yaml: last_gate_passed = 3`
- Clear `mid_stage_checkpoint` (no longer in mid-stage)
- Hand control back to orchestrator

On **iterate:**
- For each picked decision, re-ask. Re-write `decisions.*`.
- Re-generate blueprint.
- Re-present Gate 3.

On **stop here:**
- Save state. Exit cleanly. State preserves `mid_stage_checkpoint` so user can resume later.

---

## Hard rules

- **NEVER** advance `last_gate_passed` past 3 — Stage 4 does that
- **NEVER** skip the closing outstanding-questions prompt — it's the safety valve for unmet user needs
- **NEVER** save partial decisions; each decision either fully resolves or stays open
- **ALWAYS** use the `AskUserQuestion` tool for every decision in the queue and for Gate 3. Inline prose questions (`> "Where do meeting transcripts live?"` blockquotes) blend into description prose — the AskUserQuestion UI makes "this needs your answer" unmistakable.
- **ALWAYS** mark "Extension beyond Hannah's published guidance" if the decision involves an Extension rule from `_principles.md` (e.g., the inline/split/pointer-only tier framing — though that's resolved in Stage 1, this stage shouldn't introduce new extension claims)
- **ALWAYS** write `mid_stage_checkpoint` after each decision is resolved (atomic write)

---
name: team-os-propose
description: Stage 2 of the team-os-setup chain. Reads Stage 1 findings and generates a high-level architecture proposal — folder tree (Hannah's canonical shape personalized to the user's products/customers/team), team-routing tier reflected, versioning gap question answered, and a 5–10 item decision queue routed to Stage 3.
---

# Stage 2 — Propose (high-level architecture)

You are Stage 2 of the `team-os-setup` chain. Stage 1 captured the user's company, products, team, and source-folder findings. Your job: turn that into a concrete folder-tree proposal, ask the versioning gap question, and prepare the decision queue for Stage 3.

You do NOT re-interview Stage 1 facts. If a fact is missing, halt and route the user back to Stage 1 — don't paper over gaps.

**Read first:**
- `_voice.md`
- `_principles.md` — repeating product-area pattern, format-by-depth, build-on-demand defaults
- `<repo-root>/team-os-build/state.yaml` — full Stage 1 results
- `<repo-root>/team-os-build/research/findings-YYYY-MM-DD.md` — Stage 1 output

**You write:**
- `<repo-root>/team-os-build/research/proposal-YYYY-MM-DD.md`
- Updates to `state.yaml`: `decisions.versioning`, `gates_passed[1]`

---

## Step 1 — Read inputs

Load:
- `state.yaml` — confirm these fields are populated (halt if any required missing):
  - `company.name`, `company.slug`, `company.team_routing_tier`
  - `products` (≥1 entry)
  - `team_functions` (≥1 entry)
  - `decisions.multi_product.choice`
  - `daily_collaborators` (≥1 entry — needed for hot-path tables) — **skip this requirement if `variant == "personal"`** (solo work has no team)
  - Also read `quick_mode` and `variant` flags
- Stage 1 findings markdown
- `_principles.md` for canonical structure rules

### Variant branching

When `state.yaml: variant == "personal"`, the proposed tree shifts toward solo/multi-project shape but **keeps `team/` intact** (per Hannah's framework — even solo work has collaborators / mentors / beta users / contractors worth tracking):

- **Keep** `team/` folder. Inside:
  - `team-org.yaml` — seeded with just the user as the single entry; auto-add rule (in `_principles.md`) grows it as collaborators come up in sessions.
  - `retros/` — keep. Solo project retros + self-retros land here. Cadence per Stage 3 Decision 10.
  - **Drop** `onboarding-guides/` — no team to onboard. If the user wants a self-onboarding doc later, a single `team/onboarding-self.md` is build-on-demand. Add a placement-note in `team/CLAUDE.md`.
- **Skip** `customers/` scaffolding (Stage 1 already routed to "build on demand" for personal).
- **Default to** `multiple-independent` shape if `decisions.multi_product.choice == "multiple-independent"` — top-level `projects/{project-slug}/` siblings, each with their own `product-development/` subtree.
- For single-project personal use, drop `product-development/` wrapper (over-structured for one project) and use a flatter shape: `prds/`, `plans/`, `notes/`, `meetings/`, `decisions/` at root level. `team/` still lives at root.
- Keep all the F3 reading-order + F5 subfolder-conventions guidance in the root CLAUDE.md.

`quick_mode` doesn't change Stage 2 — versioning is the only question and it's worth asking even in quick mode.

---

## Step 2 — Generate the proposed folder tree

Build Hannah's canonical tree, personalized:

### Top level

```
{{company.slug}}-team-os/   # name TBD in Stage 4 — placeholder for now
├── CLAUDE.md                    [variant per team_routing_tier]
├── README.md                    [user maintains; ours just appends a "Setting up" pointer]
├── INSTALL.md                   [reference to /team-os-setup; pre-shipped in Hannah's repo]
├── .claude/
│   ├── agents/
│   ├── commands/
│   └── skills/team-os-setup/    [the chain itself, when shipped]
├── product-development/
│   ├── CLAUDE.md
│   ├── feature-index.yaml
│   ├── product/
│   ├── engineering/
│   ├── analytics/      [only if `analytics` in team_functions]
│   ├── data-engineering/  [only if `data-engineering` in team_functions]
│   └── design/         [only if `design` in team_functions]
└── team/
    ├── CLAUDE.md
    ├── team-org.yaml   [or skipped if user opted out at inline tier]
    ├── onboarding-guides/
    └── retros/
```

Function siblings under `product-development/` = exactly the functions in `state.yaml: team_functions`. Don't include functions the user didn't select.

### Inside `product-development/product/`

```
product/
├── CLAUDE.md
├── PRDs/
│   ├── CLAUDE.md
│   ├── {feature-area-1}/      [derived per multi-product mode below]
│   ├── {feature-area-2}/
│   └── ...
├── customers/                   [build-on-demand by default; only scaffolded if Stage 3 says yes]
├── competitive-research/
├── strategy/{business-context, plans, roadmaps, vision}/   # each subfolder organizes by quarter (e.g., vision/Q2-2026/, roadmaps/Q2-2026/) — past quarters get pruned
├── product-context/{business-context, jtbd, plans}/   # business framing, jobs-to-be-done docs, reference plans for product systems
├── launch-emails/
├── meetings/{sprint-planning, standup, team-bi-weekly}/   [or alternative per Stage 3]
├── processes/
├── workflows/
└── sales-enablement/            [only if `sales` in team_functions; build-on-demand by default]
```

### Inside `product-development/engineering/`

```
engineering/
├── CLAUDE.md
├── plans/
│   ├── CLAUDE.md
│   ├── {feature-area-1}/
│   └── ...
├── rfcs/
│   ├── CLAUDE.md
│   ├── {feature-area-1}/
│   └── ...
└── bug-investigations/           [build-on-demand by default]
```

### Inside `product-development/analytics/`

```
analytics/
├── CLAUDE.md
├── data-catalog.yaml
├── dashboards.md            # top-level index — what dashboards exist + what they answer
├── queries/{feature-area-1}/, {feature-area-2}/, ...
├── schemas/{feature-area-1}/, ...
├── metrics/{feature-area-1}/, ...
├── dashboards/{feature-area-1}/, ...
├── experiments/{feature-area-1}/, ...
├── investigations/
└── playbooks/
```

The `dashboards.md` top-level index is Hannah's pattern: lists every dashboard the team has, what question it answers, where it lives (URL or path), who owns it. Same role as `feature-index.yaml` but for dashboards specifically — Claude reads it when asked "is there a dashboard for X?"

### Inside `product-development/data-engineering/`

```
data-engineering/
├── CLAUDE.md
├── plans/{feature-area-1}/, ...
└── rfcs/{feature-area-1}/, ...
```

### Inside `product-development/design/`

```
design/
└── CLAUDE.md   [pointer-only to Figma per Hannah's pattern; scope confirmed in Stage 3]
```

### Inside `team/`

```
team/
├── CLAUDE.md
├── team-org.yaml                [optional at inline tier, recommended at split/pointer-only]
├── onboarding-guides/
│   ├── general.md               [+ per-role files added per Stage 3 onboarding fan-out decision]
│   └── {role}.md per team_function
└── retros/.gitkeep
```

---

## Step 3 — Derive feature-area folder names

Per `_principles.md` repeating-product-area pattern: same feature-area folder names appear across `product/PRDs/`, `engineering/plans/`, `engineering/rfcs/`, `analytics/{queries, schemas, metrics, dashboards, experiments}/`, `data-engineering/{plans, rfcs}/`.

Derivation depends on multi-product mode:

| Mode | Feature-area folders |
|---|---|
| `single` | Per-product features only. e.g., `{products[0].slug}` is irrelevant — features named directly: `auth/`, `billing/`, `home-page/`, etc. Source: `candidate_features_found` from Stage 1 (filter to top 3–5 by recency or salience). If none found, use placeholders: `feature-1/`, `feature-2/`. |
| `tightly-coupled` | Per-product subfolders OR shared subfolders. e.g., `PRDs/{product-a-slug}/`, `PRDs/{product-b-slug}/`, `PRDs/shared-billing/`. Sub-skill picks based on `candidate_features_found` content + product names. |
| `independent` | Top-level reorg: each product gets its own `products/{product-slug}/` sibling at the repo root, each with its own `product-development/` subtree. Heavier; only when user explicitly chose it. |

**For all modes:** if you can't confidently derive feature-area names, use placeholders + note in the proposal: *"Feature-area folder names are placeholders — rename in your repo when you have your first 2–3 features clearly defined."*

---

## Step 4 — Resolve the team-routing pattern in the proposal

Per `state.yaml: company.team_routing_tier`:

| Tier | Root CLAUDE.md template | Function-area CLAUDE.md inlines | team-org.yaml |
|---|---|---|---|
| `inline` | `claude-md-root-inline.md` (full team in root) | Optional hot-path table (omit if root has full team) | Optional |
| `split` | `claude-md-root-split.md` (leadership inlined; pointer for rest) | Hot-path table (3–5 daily collaborators per function) | Recommended |
| `pointer-only` | `claude-md-root-pointer-only.md` (no team data inlined) | Hot-path table (3–5 daily collaborators per function) | Required |

Document the chosen tier + rationale in the proposal under "Team-routing pattern selection."

**Mark this section as Extension** in the proposal: *"⚠️ This routing pattern extends Hannah's published guidance — her example assumes ~10 ppl. The tier you're getting is based on your stated org size of {N}. Override anytime in Stage 3 if your situation calls for it."*

---

## Step 5 — Versioning gap question (the only question this stage asks)

**Stage opener** (print before the question):

> *Quick orientation: I've read your folders, captured your company shape, and drafted a proposed architecture above. I have one decision to put in front of you here — the versioning question — and then a 5–10 item decision queue routed to Stage 3 (the "advanced" stage, where the substantive customizations happen). Take your time.*

```
Header: "How should PRD v1 → v2 carry forward?"
Description: "Hannah's principle: no dead context from finished projects, no frequently-changing data. The trade-off is between forgetting useful history and polluting Claude's thinking room with stale plans."

Options:
  - "Living docs" [RECOMMENDED]
    → One PRD per feature, edited in place. Append 'Revision history' section at the bottom with v1/v2/v3 + date + 1-line summary. feature-index.yaml points to the single file.
  - "Versioned files in folder"
    → product/PRDs/{feature}/v1.md, v2.md, current.md (symlink/copy). Heavier; useful only if you genuinely revisit old versions weekly.
  - "Archive on superseding"
    → Current PRDs flat; superseded ones move to product/PRDs/_archive/{feature}-{date}.md.
  - "Help me figure out which fits — diagnostic"
```

**Diagnostic path follow-ups:**
- "How often does your team revisit superseded PRDs — weekly, monthly, never?"
- "When a PRD changes scope, does the old one have decisions you might want to reference, or just outdated plans?"
- "Do you do quarterly retros on shipped features? (If yes, archive on superseding gets useful.)"

Then recommend.

Save to `state.yaml: decisions.versioning.choice` + `.rationale`.

---

## Step 6 — Build the decision queue for Stage 3

These are the 5–10 questions Stage 3 will walk. Some are pruned based on Stage 1 + 2 answers (per Stage 3's pruning rules).

Default queue:

1. Meeting transcripts placement *(gap question, framed as understand-and-customize)*
2. Customer accounts depth — only if `state.yaml: customers` non-empty
3. Bug investigation cadence
4. Onboarding guide fan-out
5. Strategy folder shape
6. Design integration — only if `design` in team_functions
7. Data catalog scope — only if `analytics` in team_functions
8. Workflows folder
9. Sales enablement folder — only if `sales` in team_functions
10. Retros cadence

Number them in the proposal. Stage 3 walks them in order.

---

## Step 7 — Generate the proposal markdown

Write `<repo-root>/team-os-build/research/proposal-YYYY-MM-DD.md`:

```markdown
# Proposal — {{company.name}} Team OS architecture

**Generated:** {today}
**Routing tier:** {{company.team_routing_tier}} (extension beyond Hannah's published guidance)
**Multi-product mode:** {{decisions.multi_product.choice}}
**Versioning:** {{decisions.versioning.choice}}

---

## Proposed structure

[Render tree using box-drawing characters: ├── └── │ . Indent each level 4 spaces. Show the resolved feature-area folders by name, not as `{feature-area}` placeholders.]

## Naming choices

- Folders: lowercase-with-hyphens
- Files: lowercase-with-hyphens.md
- AI context filename: {{company.ai_context_filename}}
- Bug folders: bug-{YYYY-MM-DD}-{slug}/
- Master indexes: {type}-index.yaml

## Team-routing pattern selection

⚠️ **Extension beyond Hannah's published guidance.** Hannah's example assumes ~10 ppl. Based on your stated org size ({{company.org_size_bracket}}), you're getting the **{{company.team_routing_tier}}** tier:

[Inline the relevant row from `_principles.md` Org-size routing tiers table.]

You can override at any later point — see `_principles.md` for the rule.

## Versioning decision

**Choice:** {{decisions.versioning.choice}}
**Why:** {{decisions.versioning.rationale}}

## Decision queue for Stage 3

These 5–10 decisions shape the build. Stage 3 walks them one at a time, with Hannah's principle for each + a recommended default.

1. Meeting transcripts placement *(gap question)*
2. {... (numbered list of active decisions)}

## What you'd see at L1

- ~{N} folders mkdir'd
- ~{M} CLAUDE.md files (root + function + folder)
- 3 YAML registries (feature-index, data-catalog, team-org)
- 3 sample artifacts (PRD, RFC, plan) personalized with {{products[0].name}} + {{first_feature.slug}}
- 1 _pointers.md indexing your source folders + MCP tools

L2/L3 upgrade paths are previewed but not executed in v1.

---

*Proposal generated by `team-os-propose`. Stage 3 (`team-os-advanced`) walks the decision queue and produces the final blueprint.*
```

---

## Step 8 — Gate 2 (user picks path)

Print 1-paragraph summary: tree shape (top-level dirs + key counts), routing tier, multi-product mode, versioning choice. Then AskUserQuestion:

```
Header: "Proposal ready. Pick your path:"
Options:
  - "Approve and continue to advanced settings (Stage 3)" [v1 default — full deep-dive]
  - "Iterate on the proposal" (free-text note next; regenerate the relevant section)
  - "Iterate on versioning" (re-ask the versioning question only)
  - "Stop here"
```

[v1.next: a 5th option "Approve and skip to L1 build (use defaults for advanced settings)" gets added — quick-start path. v1 doesn't include it.]

On **approve:**
- Write `state.yaml: gates_passed[1]` (timestamp, commit_sha=null)
- Write `state.yaml: last_gate_passed = 2`
- Hand control back to orchestrator

On **iterate:**
- Capture note. Re-generate the affected section (likely the tree visualization or the feature-area derivation).
- Re-present Gate 2.

On **iterate versioning:**
- Re-ask the versioning gap question (Step 5).
- Re-write `decisions.versioning.*`.
- Update proposal.
- Re-present Gate 2.

On **stop here:**
- Save state. Exit cleanly.

---

## Hard rules

- **NEVER** re-ask Stage 1 questions; if state is missing required fields, halt
- **NEVER** advance `last_gate_passed` past 2
- **ALWAYS** use the `AskUserQuestion` tool for the versioning question and Gate 2 — both are real decision points the user owns. Never frame them as inline prose questions; the AskUserQuestion UI is what makes "this needs your answer" unmistakable.
- **ALWAYS** mark the team-routing section as "Extension beyond Hannah's published guidance"
- **ALWAYS** show the tree as a rendered visualization, not a bulleted list (folder trees scan faster as boxes)
- **NEVER** invent feature-area names with high confidence if `candidate_features_found` is empty — use placeholders + flag for user

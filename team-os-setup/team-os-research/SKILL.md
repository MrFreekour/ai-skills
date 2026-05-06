---
name: team-os-research
description: Stage 1 of the team-os-setup chain. Reads the user's existing folders/repos (via sub-agent dispatch when large), runs a deep-dive interview that only asks what couldn't be extracted from research, captures the multi-product gap question, and writes findings.md for Stage 2.
---

# Stage 1 — Research (read-first, ask-second)

You are Stage 1 of the `team-os-setup` chain. The orchestrator has already finished preflight (GitHub auth, MCP inventory, gitignore check, branch creation). Your job: understand the user's company well enough to personalize the rest of the chain — by reading what they already have first, and asking only what's left.

The reading pass is the **first source of truth.** Every interview question below is "ask only if not already extractable from the reading pass." Don't waste user time asking what their CLAUDE.md already says.

**Read first:**
- `_voice.md`
- `_principles.md`
- `<repo-root>/team-os-build/state.yaml` — preflight already wrote `mcp_tools_indexed` here. Also read `quick_mode` and `variant` flags.

### Variant branching

- **`state.yaml: variant == "personal"`** activates personal-projects mode. This stage:
  - Skips Q4 (function/dept size — auto-set to `"Just me"`).
  - Skips Q5 (daily collaborators — leave `daily_collaborators[]` empty).
  - Replaces Q7 (business model + named customers) with the lighter "audience" question (see Q7-personal below).
  - Reframes Q6 (multi-product) — for personal projects, "multiple-independent" is usually right (main project + side project + side project), but the diagnostic still applies if the user has a shared infra layer they're factoring out.
  - Auto-derives `team_routing_tier = "inline"` regardless of org-size answer (because team is "Just me" — there's no org).
  - Auto-sets `business_model = "self-serve"` or `"pre-revenue"` based on the audience question's answer (skips the customer-hierarchy table entirely).

**You write:**
- `<repo-root>/team-os-build/research/findings-YYYY-MM-DD.md`
- Updates to `state.yaml`: `company.*`, `products`, `team_functions`, `daily_collaborators`, `customers`, `decisions.multi_product`, `source_pointers`, `candidate_features_found`, `candidate_tables_found`, `mid_stage_checkpoint`, `gates_passed[0]`

---

## Step 1 — Pre-stage: source pointer collection

Check `state.yaml: source_pointers`. If empty, AskUserQuestion:

```
Header: "Where does your team's current knowledge live?"
Description: "Paste folder paths or GitHub URLs (one per line). I'll read them so you don't have to re-explain everything I can already see. Type 'none' to start fresh."

Options:
  - "I'll paste them now" (free-text input next)
  - "None — start from blank" (skip reading pass)
```

If user pastes: parse each line, classify as `local-dir`, `local-file`, `github-url`, or `unknown`. Save to `state.yaml: source_pointers[]` with the kind.

For `unknown` shapes (e.g., Notion URL, Google Drive link): log + tell user *"v1 doesn't crawl {kind}. I'll skip these. v1.next will add ingest paths for Notion/GDrive via MCP."*

---

## Step 2 — Reading pass (dispatched to sub-agent for large pointers)

For each pointer, decide dispatch mode:

| Condition | Dispatch |
|---|---|
| `local-dir` with >50 files (estimate via `Glob` + count) OR `github-url` | **Sub-agent** (Agent tool, `Explore` for local, `general-purpose` for GitHub) |
| `local-dir` with ≤50 files OR single `local-file` | **Inline** (orchestrator reads directly) |

### Sub-agent prompt template (one call per large pointer)

> You're reading a source folder for a Team OS setup. Your output feeds Stage 1's findings. Pointer: `{path}` (kind: `{kind}`).
>
> **Read pattern:**
> - Local: `Glob **/*.{md,yaml,yml}` with ignore patterns: `node_modules/`, `.git/`, `_archive/`, `.obsidian/`, `dist/`, `build/`, `.next/`, `vendor/`, `target/`. Cap 100 files. Prioritize `*/CLAUDE.md`, `*/AGENTS.md`, `*/README.md`, `*/index.yaml`, `*/feature-index.yaml`, `*/data-catalog.yaml` first; then top 20 by mtime.
> - GitHub: WebFetch `/`, `/CLAUDE.md`, `/README.md`, `/docs/`, `/product/`, `/engineering/`, `/team/` (404s OK; report which paths exist).
>
> **Extract and return** (structured markdown, under 500 words):
> 1. Tree summary — top-level folders + 1-line purpose each
> 2. CLAUDE.md / AGENTS.md presence (yes/no per file found)
> 3. Naming style (kebab / snake / camel / mixed)
> 4. File-type histogram (count of .md, .yaml, etc.)
> 5. **Candidate features** — folder names matching `product/`, `PRDs/`, `features/`, `specs/` patterns; or filenames matching `*-prd.md`. Return as a list of `{name, source_path}`.
> 6. **Candidate tables** — anything matching `*_schema.md`, `data-catalog.yaml`, `tables.yaml`, or `*.sql` files in an `analytics/` or `data/` directory. Return as `{name, source_path}`.
> 7. **Candidate company name** — best guess from README title, package.json name, or repo name. (Stage 1 will confirm with user.)
> 8. **Candidate product names** — from product folders, README, or PRD frontmatter.
> 9. **Candidate daily collaborators** — names from CLAUDE.md team table, README contributors, or git log top 5 contributors.
>
> **Do NOT** invent details. If you can't find something, say so explicitly.

Save sub-agent results into `state.yaml`:
- `source_pointers[i].claude_md_present`, `naming_style`
- `candidate_features_found` (merge across pointers, dedupe)
- `candidate_tables_found` (same)
- Stash company/product/people candidates for Step 3 confirmation

For inline reading: do the same extraction directly with Glob + Read, but bounded.

---

## Step 3 — Interview (ask only what wasn't extracted)

**Stage opener** (print before the first AskUserQuestion):

> *Take as long as you need on these questions — ask me to clarify or expand whenever it helps. The interview is invitation, not interrogation; if a question feels off for your situation, push back and we'll adjust.*

For each question below: **first check** if state.yaml + research already has the answer. If yes, **show user for confirmation only** ("I found X — confirm or correct?"). Only AskUserQuestion if extraction returned nothing.

After each question's answer is captured, write `state.yaml: mid_stage_checkpoint.last_question_answered: {N}` (atomic). On crash, resume from N+1.

### Question 1 — Company name & slug

If candidate company name was extracted, AskUserQuestion:

```
Header: "Company name"
Description: "I see this looks like {candidate}. Confirm or correct."

Options:
  - "Yes — {candidate}" (Recommended)
  - "Different name" (free-text via Other)
```

If no candidate was found, AskUserQuestion:

```
Header: "Company name"
Description: "What's your company name?"

Options:
  - "Type it" (free-text via Other)
```

Auto-derive `company.slug` (lowercase, hyphenated, strip non-alphanumerics — handle apostrophes: "O'Reilly" → `oreilly`; unicode: pass through as-is, browsers + git handle it).

### Question 2 — Team functions

AskUserQuestion (always use the tool — never inline prose):

```
Header: "Team functions"
Description: "{If functions detectable: 'I see these functions in your folders: {list}. Multi-select to confirm or adjust.' / Else: 'Multi-select the team functions present at your company.'}"
multiSelect: true

Options (≤4 per AskUserQuestion call — split into two calls if more selected):
  - "product"
  - "engineering"
  - "design"
  - "analytics"
```

Then a follow-up AskUserQuestion for the remaining options:

```
Header: "Other functions"
Description: "Any of these also present?"
multiSelect: true

Options:
  - "data-engineering"
  - "customer-success"
  - "marketing"
  - "sales"
```

If "Other" needs to surface ops or another function, the user can use the Other affordance in either call.

Save to `state.yaml: team_functions`.

### Question 3 — Org size *(drives team-routing tier)*

```
Header: "How many people are in your org?"
Description: "This drives how we route team data — small orgs inline everyone in the root CLAUDE.md; bigger orgs split it into a team-org.yaml registry. (This is an extension beyond Hannah's published guidance — her example assumes ~10 ppl.)"

Options:
  - "<15"
  - "15–50"
  - "50–150"
  - "150+"
```

Derive `state.yaml: company.team_routing_tier`:
- `<15` → `inline`
- `15–50` → `split`
- `50+` (both 50–150 and 150+) → `pointer-only`

**At 50+:** prompt for org-mapping dispatch:

```
Header: "Want to map your full org now (with departments + reporting lines), or keep it light for v1?"
Description: "At {{company.org_size_bracket}} ppl, mapping the full org is open-ended work. You can spawn it as a new session (returns a team-org.yaml draft, doesn't bloat this conversation) or do it inline (fast but uses ~3k tokens of this window) — or skip and let team-org.yaml fill in via the auto-add rule as people come up in sessions."

Options:
  - "Spawn as new session (Recommended for 50+ ppl)"
  - "Do it inline now"
  - "Skip — team-org.yaml fills via auto-add"
```

If "spawn": tell the user to run `/spawn-claude-session` (or whichever spawn mechanism they have) with the prompt: *"Map the {{company.name}} org structure into a team-org.yaml file. Use the schema from `<this-skill>/_templates/team-org.yaml.seed`. When done, save to `<repo-root>/team-os-build/team-org-draft.yaml`. Return when done."* — and pause Stage 1 here. When user resumes, read the draft into state.yaml's `daily_collaborators` + dept structure.

If "inline": continue to Question 4 + 5 normally.

If "skip": skip to Question 6 (no daily-collaborator capture; team-org.yaml stays empty at L1).

### Question 4 — Function/dept size (skipped when `variant == "personal"`)

If `variant == "personal"`: auto-set `state.yaml: company.function_size_bracket = "Just me"` and skip to Q6. Solo work has no function-size question to ask.


```
Header: "How many people are in your function/dept?"
Description: "This drives what gets inlined in the function-area CLAUDE.md (manager + 3-5 daily collaborators)."

Options:
  - "Just me"
  - "2–5"
  - "6–15"
  - "15+"
```

### Question 5 — Daily collaborators (skipped when `variant == "personal"`)

If `variant == "personal"`: skip entirely. Leave `state.yaml: daily_collaborators = []`. Templates handle the empty case.


AskUserQuestion (use the tool — never inline prose):

```
Header: "Daily collaborators"
Description: "Name 3–5 people you work with daily — your manager + direct collaborators. These inline in the relevant function-area CLAUDE.md. The rest of your org goes in team-org.yaml. Type names comma-separated via Other."

Options:
  - "Type names" (free-text via Other; comma-separated)
  - "Skip — fill later via team-org.yaml auto-add"
```

If user types names, parse them. For each person, run one AskUserQuestion to capture role/function/manager/slack — batched together as a single 4-question call (one question per attribute):

```
Q1 Header: "Role"  →  free-text via Other
Q2 Header: "Function"  →  options from state.yaml: team_functions + Other
Q3 Header: "Manager"  →  options: existing daily_collaborator slugs + "None / external" + Other
Q4 Header: "Slack handle"  →  free-text via Other (optional, "skip" allowed)
```

Save to `state.yaml: daily_collaborators[]`.

If user said "spawn new session" in Q3: skip Q4 and Q5 here — those are answered by the spawned session.

### Question 6 — Multi-product gap question

```
Header: "How many products do you ship?"
Description: "Hannah's principle: every line in a CLAUDE.md file costs thinking room. Mixing unrelated products in one CLAUDE.md is one of her NEVERs — it dilutes context for every query. But shared infra (auth, billing, analytics) often DOES live across products and benefits from one home."

Options:
  - "Single product"
  - "Multiple, tightly coupled (shared infra, one team)" [RECOMMENDED for shared infra]
  - "Multiple, independent (separate teams, separate repos)"
  - "Help me figure out which fits — diagnostic"
```

**Diagnostic path follow-ups:**
- "What's in your current top-level folders? (paste tree or describe)"
- "Do all your products share the same database / auth / billing?"
- "Same codebase or separate?"

Then steer based on answers:
- All shared infra + one codebase → `tightly-coupled`
- Separate codebases + separate teams → `independent`
- One product → `single`

If candidate product names were extracted in research, prompt: "I found these products in your docs: {list}. Confirm or correct?"

Save to `state.yaml: products[]` and `state.yaml: decisions.multi_product.choice` + `.rationale`.

### Question 7 — Business model + named customers

**If `variant == "personal"`, replace this entire question with Q7-personal below.** The B2B/B2C/customer-hierarchy framing doesn't fit indie or side-project work; "audience" is the right shape for personal projects.



First ask the business model — drives whether `customers/accounts/{slug}/` folders get scaffolded by default at L1.

```
Header: "What's your business model?"
Description: "B2B teams almost always benefit from per-customer folders; B2C / self-serve teams usually don't. Hannah's example treats per-customer CLAUDE.md as 'the pattern, not optional' for B2B."

Options:
  - "B2B with named accounts (enterprise / mid-market / SMB)"
  - "B2C / consumer"
  - "Self-serve SaaS — long tail, no named-account focus"
  - "Pre-revenue / pre-customer"
  - "Mixed — some named B2B accounts, some self-serve"
```

Save to `state.yaml: company.business_model` (values: `b2b` | `b2c` | `self-serve` | `pre-revenue` | `mixed`).

**Then, if model is `b2b` or `mixed`** — AskUserQuestion (use the tool):

```
Header: "Named accounts"
Description: "Name 3–5 accounts to seed customers/accounts/. Each gets its own CLAUDE.md in your L1 — Hannah's pattern, not optional for B2B. If my reading pass already found customer-specific files, I'll pre-fill the richer scaffolds; customers without files get minimal stubs."

Options:
  - "Type names (comma-separated)" (free-text via Other)
  - "None yet — B2B intent but no named customers" (skip scaffolding for now)
```

**B2B auto-detection during reading pass:** if Stage 1's reading pass found files matching patterns like `customers/{slug}/`, `accounts/{slug}/`, `{customer-name}-{topic}.md` in source pointers, surface this and pre-fill the customer list: *"I found these customer-shaped files in your docs: {list}. Confirm which are real customers vs. examples."* This signal can flip business_model to `b2b` even if the user picked something else (with confirmation).

Save customers to `state.yaml: customers[]`. For each: if reading pass found per-customer files, attach `pre_fill_files: [paths]` so Stage 4 can populate the customer's CLAUDE.md with extracted content.

**Stage 4 customer-scaffolding rule** (set at this point in state.yaml as `decisions.customers_default`):
- `b2b` + customers list non-empty → scaffold per-customer CLAUDE.md (richer for those with pre-fill files, minimal stubs for those without)
- `mixed` + customers list non-empty → same as B2B for the named ones; build-on-demand for the rest
- `b2c` / `self-serve` / `pre-revenue` / customers list empty → keep build-on-demand

### Question 7-personal — Audience (used only when `variant == "personal"`)

AskUserQuestion (use the tool):

```
Header: "Audience"
Description: "For personal projects: who's the audience? Drives whether we scaffold any audience-shaped folders. Default for indie / side projects: skip the customers/ folder entirely; revisit if a project goes to market."

Options:
  - "Just me / nobody yet" (Recommended for early-stage side projects)
  - "Friends + family / closed beta"
  - "Public users / paying customers"
  - "Mixed across projects"
```

Save to `state.yaml: company.audience` (`solo` | `closed_beta` | `public` | `mixed`). Auto-derive `business_model`:
- `solo` / `closed_beta` → `pre-revenue`
- `public` → `self-serve`
- `mixed` → `mixed`

Set `customers[]` to empty (`[]`). Set `decisions.customers_default = "build-on-demand"`. No customer-hierarchy table; no per-customer scaffolding.

If user picked `public` or `mixed` AND has named customers/users they want to track, suggest they re-run `/team-os-setup --deepen customers/` later when that lands. Don't seed the customers folder at L1.

### Question 8 — Current pain point

AskUserQuestion (always — prose-shaped questions blend into description text and users can read past them; the AskUserQuestion UI is unmistakable):

```
Header: "Pain point"
Description: "What's the main reason you're setting this up? This tunes the sample artifacts and which scaffolds get prioritized in your L1 build."

Options:
  - "AI doesn't know our context — sessions repeat themselves"
  - "Onboarding is slow — new hires take weeks to ramp"
  - "Decisions aren't traceable — we relitigate the same calls"
  - "Other (describe)" (free-text via Other)
```

Save to `state.yaml: company.primary_pain_point`. Stage 4 uses this to lightly tune sample-artifact wording (e.g., the PRD's "Why now" section gets a hint pointing at the pain point).

### Question 9 — Existing AI context filename standard

If reading pass found CLAUDE.md or AGENTS.md files: confirm.
If both: ask which to standardize on.
If neither: AskUserQuestion:

```
Header: "Which AI context filename do you want to use?"
Description: "Both work; pick one and use it everywhere. CLAUDE.md is the default (for Claude Code); AGENTS.md is the Cursor convention."

Options:
  - "CLAUDE.md (Recommended unless your team uses Cursor)"
  - "AGENTS.md"
```

Save to `state.yaml: company.ai_context_filename`.

### Question 10 — MCP-detected tools to index

The orchestrator's preflight populated `state.yaml: mcp_tools_indexed` with what's connected. Confirm which the user wants surfaced in their L1.

```
Header: "I see these MCP tools are connected: {list}. Index any in your team-org.yaml or root CLAUDE.md?"
Description: "Slack MCP populates the channel map in root CLAUDE.md. Granola/Otter populate transcript pointers in _pointers.md. Notion populates cross-references."

Options:
  - "Index all" (Recommended)
  - "Pick which" (multi-select next)
  - "Skip — can re-index later"
```

If user has zero MCP tools connected: skip this question entirely. Empty `mcp_tools_indexed` is fine; templates handle the empty case.

### Question 11 — Anything else I should know?

```
Header: "Anything else about your setup I should know before we propose an architecture?"
Description: "Optional — free text. Things like industry-specific compliance (HIPAA, SOC2), unusual team structures, multi-region considerations, or tooling quirks. I'll log this in findings; Stage 2 will fold it into the proposal."

Options:
  - "No, that's everything"
  - "Yes, I have something" (free-text input next)
```

Save free-text answer to `state.yaml: outstanding_questions_log[]` with timestamp + tag `stage-1-context`.

---

## Step 4 — Generate findings.md

Write `<repo-root>/team-os-build/research/findings-YYYY-MM-DD.md`:

```markdown
# Findings — {{company.name}}

**Generated:** {today}
**Run by:** Stage 1 (`team-os-research`)

---

## Company snapshot

- **Name:** {{company.name}} ({{company.slug}})
- **Org size:** {{company.org_size_bracket}}
- **Team-routing tier:** {{company.team_routing_tier}} (extension beyond Hannah's published guidance)
- **Function/dept size:** {{company.function_size_bracket}}
- **AI context filename:** {{company.ai_context_filename}}
- **Primary pain point:** {{company.primary_pain_point}}

## Products

[List products with multi-product mode noted]

## Team functions present

[List from team_functions]

## Daily collaborators (hot-path team)

[Table: name | role | function | manager | slack]

## Customers (named accounts to seed)

[List or "none — pre-revenue / all self-serve"]

## Source-folder inventory

[Table: path | kind | claude_md present? | naming style | file-type histogram summary]

## Gap-vs-Hannah baseline

[3-column: Hannah baseline | What you have | Gap]
- product-development/ wrapper | {found / not found} | {needs creation / matches}
- feature-index.yaml | {found / not found} | ...
- team-org.yaml | {found / not found} | ...
- CLAUDE.md hierarchy | {found / not found} | ...

## Multi-product decision

**Choice:** {{decisions.multi_product.choice}}
**Why:** {{decisions.multi_product.rationale}}

## Candidate features found in source docs

[List with source paths — Stage 4 uses these to pre-fill feature-index.yaml]

## Candidate tables found in source docs

[List with source paths — Stage 4 uses these to pre-fill data-catalog.yaml]

## MCP tools indexed

[List from mcp_tools_indexed]

## Open considerations from user

[Q11 free-text log, if any]

## Routed to Stage 2

- Versioning gap question
- Multi-product structural implications

## Routed to Stage 3

- Meeting transcripts placement (gap question)
- 5–9 architecture decisions per `_principles.md` decision-time defaults

---

*Findings generated by `team-os-research`. Stage 2 (`team-os-propose`) reads this to generate the architecture proposal.*
```

---

## Step 5 — Gate 1

Print 1-paragraph summary: company snapshot, function count, product count, customer count, source-pointer count, candidate features/tables found. Then AskUserQuestion:

```
Header: "Stage 1 complete. Continue to architecture proposal?"
Options:
  - "Approve and continue"
  - "Iterate — I want to adjust something" (free-text note next; re-runs the affected interview question)
  - "Stop here"
```

On **approve:**
- Write `state.yaml: gates_passed[0]` (timestamp, commit_sha=null)
- Write `state.yaml: last_gate_passed = 1`
- Clear `mid_stage_checkpoint`
- Hand control back to orchestrator

On **iterate:**
- Capture note. Identify which question(s) need re-running. Re-ask. Update findings.
- Re-present Gate 1.

On **stop here:**
- Save state. Exit. State preserves `mid_stage_checkpoint` for resume.

---

## Hard rules

- **NEVER** ask a question whose answer is already extractable from the reading pass without first showing the user the candidate for confirmation
- **NEVER** invent answers — if extraction failed, ask explicitly; never pretend to know
- **ALWAYS** use the `AskUserQuestion` tool for every user-input request, including confirmations of extracted candidates and free-text follow-ups (use the "Other" affordance for free text). Inline blockquote-shaped questions (`> "What's your company name?"`) blend into prose and users can read past them; the AskUserQuestion UI makes "this needs your answer" unmistakable.
- **ALWAYS** write `mid_stage_checkpoint.last_question_answered` after each question (atomic)
- **NEVER** advance `last_gate_passed` past 1
- **NEVER** auto-commit; orchestrator handles git ops
- **ALWAYS** dispatch to a sub-agent for source pointers >50 files — keep the orchestrator's window lean per `_principles.md` Context Budget rules

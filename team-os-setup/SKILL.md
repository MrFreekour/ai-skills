---
name: team-os-setup
description: Set up a Team OS for your company, modeled on Hannah Stulberg's framework. Runs a 4-stage interview (research → propose → advanced → execute) and builds a Level 1 pointer environment in a sibling directory. Auto-triggers on "set up team os", "build team os", or "/team-os-setup". 30 minutes of setup; hours saved every week.
---

# `/team-os-setup` — Orchestrator

You're the orchestrator for the Team OS setup chain. Your job: run preflight, create a recovery branch, dispatch the 4 sub-skills in order, handle gates with auto-commits, and present the final closer.

You do NOT do interview, generation, or build work — that's the sub-skills. Your scope is: preflight + dispatch + gate handling + git ops + final closer.

**Read first:**
- `_voice.md` — voice rules
- `_principles.md` — Context Budget & Sub-Agent Dispatch section (you handle dispatch decisions)

**You write:**
- `<repo-root>/team-os-build/state.yaml` — initial creation
- Updates to `state.yaml` after each gate (auto-commit handling)
- Branch `team-os-setup-YYYY-MM-DD` (with auto-commits)

---

## Invocation modes

Branch on flags from the user's invocation:

| Flag | Mode |
|---|---|
| (none) or `/team-os-setup` | Full chain: preflight → Stage 1–4 → final closer |
| `--quick` | Full chain, but Stage 3 auto-applies recommended defaults (no user prompts on architecture decisions). Best for personal-use-at-company or small-team starter setups. |
| `--variant=personal` | Solo / personal side-projects shape: skips team interview, prunes `team/` + `onboarding-guides/` + `retros/`, swaps customer hierarchy for "audience". Combine with `--quick` for the fastest path. |
| `--upgrade=L2` | Skip to Stage 4 in L2 light-draft mode (wholesale — all deferred items) |
| `--upgrade=L3 --confirm` | Skip to Stage 4 in L3 light-draft mode (wholesale — all deferred items) |
| `--deepen <folder-path>` | Skip to Stage 9.5 below — scaffold ONE deferred item into a real work folder |
| `--audit` | (v1.next stub — see "Audit mode design note" in `_principles.md`.) Will read an existing L1 + state.yaml, diff against current `_principles.md` doctrine, and propose upgrades for user-confirmed Stage-4-style apply. v1 ships the design note only. |
| `--resume` | Read `state.yaml: last_gate_passed`; offer resume from N+1 |
| `--refresh-pointers` | (v1.next) Re-walk source pointers + MCP tools; update `_pointers.md` only — no folder rebuild |

For upgrade flags (`--upgrade=L2` / `--upgrade=L3`): skip preflight Steps 0 + 1 (assume the user already ran the full chain once); jump straight to Stage 4 dispatch.

For `--deepen`: see Step 9.5 below.

For `--audit` (v1 stub): immediately halt with this message, do NOT enter preflight:

```
Audit mode is parked for v1.next.

The design is captured in `_principles.md` → "Audit mode design note (--audit, v1.next)".
v1 ships the design intent; the implementation lands in the next release after
the structured rule registry is added to _principles.md.

For now, your options:
  - /team-os-setup --resume        — continue your most recent build if it's not finalized
  - /team-os-setup --deepen <path> — scaffold ONE deferred folder (per-folder targeted)
  - /team-os-setup                  — fresh chain (a re-run that overrides will lose your existing L1)

If you want to track the audit design or contribute, the design note is in `_principles.md`.
```

Exit cleanly (no state.yaml write, no branch creation). User can re-invoke with another flag.

For `--quick` and `--variant=personal`: run the full chain but with shape modifications. Set state.yaml flags at preflight + tell each sub-skill to read those flags:

```yaml
# state.yaml fragments set at preflight when these flags are present
quick_mode: true              # set when --quick is passed
variant: "personal"           # set when --variant=personal is passed
```

Sub-skill behavior changes:
- **Stage 1 (`team-os-research`)** — when `variant == "personal"`, skip Q4 (function/dept size — auto-set "Just me"), skip Q5 (daily collaborators interview — leave list lightweight), and replace Q7 (business model + customers) with a lighter "audience" question (free-text via Other). Pass `quick_mode` through; Stage 1 always runs full questioning regardless because user-context capture is what makes everything else work.
- **Stage 2 (`team-os-propose`)** — when `variant == "personal"`, **keep `team/` folder** (Hannah's framework treats it as fundamental — even solo work has collaborators / mentors / beta users / contractors worth tracking). Inside `team/`: keep `team-org.yaml` (seeded with just the user; auto-add rule grows it), keep `retros/` (project retros + self-retros are valuable solo too), **drop `onboarding-guides/`** (no team to onboard; if user wants self-onboarding doc, single `team/onboarding-self.md` is sufficient — but build-on-demand by default).  When `quick_mode == true`, still ask the versioning gap question (it's only one question and the answer matters).
- **Stage 3 (`team-os-advanced`)** — when `quick_mode == true`, auto-apply recommended defaults to every decision in the active queue (no AskUserQuestion calls per decision). Show the resolved blueprint at Gate 3 with: *"All 12 decisions resolved with recommended defaults. Iterate on any specific decision before building?"* (still a real Gate 3, just the per-decision walk is skipped). When `variant == "personal"`, additionally prune the queue: drop Decision 4 (onboarding fan-out — solo has no fan-out target), Decision 9 (sales-enablement), and Decision 11 (sample artifacts — already default-skip per the decision-time defaults table in `_principles.md`). **Keep Decision 10 (retros cadence)** — solo project retros land here.
- **Stage 4 (`team-os-execute`)** — when `variant == "personal"`, generate `_pointers.md` with the user's side-project repos as the natural source pointers (Stage 1 reading pass should have captured these). **Generate `team-org.yaml`** seeded with the user as the sole entry; the auto-add rule (per `_principles.md`) will populate it as collaborators come up in sessions. Skip per-customer scaffolding entirely (Stage 1 already routed to "build on demand"). Skip `onboarding-guides/` (no team).

---

## Step 0 — Preflight: GitHub + MCP + Gitignore

Runs **before** any git check. Halts unless noted.

### Check 0.1 — GitHub access (git-credentials OR gh CLI)

The point of this check is "can the user push/pull from GitHub?" — not "is gh installed?" Many users have credentialed git access without ever installing the `gh` CLI. Detection order:

1. **First, try `git ls-remote origin --heads` (silent, fast).** If exit 0 and output lists branches → user has credentialed git access. Pass. Don't ask anything.
2. **If `git remote -v` shows no `origin`** → no remote configured. Warn (not halt): *"No `origin` remote configured. Recovery still works locally; Hannah-handoff requires you push the branch when ready (`git remote add origin …` first)."*
3. **If `ls-remote` fails with auth error** → AskUserQuestion:
   ```
   Header: "GitHub auth check failed."
   Description: "git ls-remote returned: {error}. Pick how to proceed."
   
   Options:
     - "I'll fix the credentials and retry" (halts; user runs `git config credential.helper` or installs `gh auth login`)
     - "Continue without push capability" (proceeds; recovery is local-only via `git checkout main`)
     - "Cancel"
   ```
4. **If `gh` is on PATH AND `gh auth status` succeeds** → also pass (some users prefer gh-based auth). This is checked LAST, only if `git ls-remote` failed in a way that didn't indicate auth, since `gh` adds capability beyond credentialed git.

Do NOT halt on missing `gh`. The CLI is optional; only the underlying GitHub access matters.

### Check 0.2 — MCP tool inventory

Read `<repo-root>/.mcp.json` (project) and any global Claude Code settings file. Parse for connected MCP servers. Surface to user:

```
Header: "I see these MCP tools connected: {list, or 'none'}. Want to index any in your team-os build?"
Description: "Slack MCP populates the channel map in root CLAUDE.md. Granola/Otter populate transcript pointers. Notion populates cross-references."

Options:
  - "Index all" (Recommended)
  - "Pick which" (multi-select next)
  - "Skip — none for now"
```

Save user's selection to `state.yaml: mcp_tools_indexed[]` (creating state.yaml if it doesn't exist yet).

If `.mcp.json` doesn't exist or is malformed: log + tell user *"No MCP config found — skipping. You can connect MCP tools later and re-run with `/team-os-setup --refresh-pointers` (v1.next)."* Don't halt.

### Check 0.3 — Gitignore check

Grep `<repo-root>/.gitignore` (and parent `.gitignore` files up to home dir) for patterns that would exclude `team-os-build/`. Patterns to flag:
- `team-os-build/` exact
- `build/` (generic)
- `*-build/` (glob)
- `*-os-*/` (paranoid)

If any match:
```
Header: "Your .gitignore would exclude `team-os-build/`. Auto-commits at each gate would silently fail."
Description: "Found at: {file}:{line}, pattern: `{pattern}`."

Options:
  - "Add `!team-os-build/` exception to .gitignore" (Recommended)
  - "Proceed without auto-commits — single commit at end of build instead"
  - "Cancel"
```

On exception-add: append `!team-os-build/` to the user's project-level `.gitignore`. Note this in the auto-commit message later.

On "proceed without auto-commits": set `state.yaml: commit_mode = "end-of-build"`. Skill behavior changes: no per-gate commits; single commit at Gate 4.

### Check 0.4 — Pre-commit hook smoke test

Check for `.husky/`, `.pre-commit-config.yaml`, `.git/hooks/pre-commit` (and ensure non-empty if present). If any: warn:

```
"Detected pre-commit hooks: {list}. If a commit fails during the run, I'll halt at the affected gate and let you fix the underlying issue before continuing. (No --no-verify.)"
```

Don't halt — just warn.

### Check 0.5 — Sparse-checkout cone-mode detection

Run `git config core.sparseCheckoutCone` and `git config core.sparseCheckout` (silent). If sparse-checkout is enabled (either flag returns `true`):

1. Read the active cone via `git sparse-checkout list` — capture the patterns.
2. Compute the projected target L1 path: `../{{company.slug}}-team-os-v1/` (the default; user may override in Stage 4 Step 2).
3. AskUserQuestion:

```
Header: "Sparse-checkout active. Auto-add the build path to the cone?"
Description: "Cone-mode sparse-checkout is on. Without adding the L1 path + team-os-build/ to the cone, files commit to git but don't land on disk — your smoke test would open an empty folder. Patterns currently in cone: {list}."

Options:
  - "Yes — auto-add `team-os-build/` and the L1 target before Stage 4" (Recommended)
  - "I'll manage sparse-checkout myself" (skill skips the auto-add; you run `git sparse-checkout add …` manually)
  - "Cancel"
```

On "auto-add": save `state.yaml: sparse_checkout.cone_mode = true` and `sparse_checkout.auto_add = true`. The orchestrator runs `git sparse-checkout add team-os-build` immediately (so per-gate state writes land on disk) and re-runs `git sparse-checkout add {target_l1_path}` after Stage 4 Step 2 confirms the path. Verify each add with `git sparse-checkout list`.

On "manage myself": save `state.yaml: sparse_checkout.cone_mode = true` and `sparse_checkout.auto_add = false`. Surface a halt-style reminder before Stage 4 dispatch: *"Sparse-checkout self-managed. Run `git sparse-checkout add {target_l1_path} team-os-build` now or the L1 build will appear empty on disk after commit. Confirm done?"*

If sparse-checkout is NOT enabled: save `state.yaml: sparse_checkout.cone_mode = false` and continue.

---

## Step 1 — Preflight: Git checks

Order matters. First failure halts.

| Check | Action |
|---|---|
| Inside git repo (`git rev-parse --is-inside-work-tree`) | Halt: "Run `git init` first." |
| Working tree clean (`git status --porcelain`) | Halt: print uncommitted files; ask user to commit or stash |
| Git identity set (`git config user.email` non-empty) | Halt: "Set `git config user.name` and `user.email` so commits are attributable." |
| GitHub remote (`git remote -v` shows `origin`) | **Warn-only.** "No GitHub remote — recovery only works locally. Continue anyway? [y/N]" |
| Existing `team-os-build/` dir | **Warn-only.** Read state.yaml. If `last_gate_passed > 0`, offer resume (see "Resume mechanism" below). |

Additional edge-case checks (per Devil's Advocate review):
- Mid-rebase (`.git/rebase-merge/` exists) → halt: "You're mid-rebase. Resolve or abort first."
- `.git/MERGE_HEAD` exists → halt: "You're mid-merge. Resolve or abort first."
- Detached HEAD (`git symbolic-ref HEAD` fails) → halt: "Detached HEAD — checkout a branch first."

---

## Step 2 — Branch creation

```bash
DATE=$(date +%Y-%m-%d)
BRANCH="team-os-setup-${DATE}"
```

If branch exists locally: try `${BRANCH}-2`, `-3`, … up to `-9`. After `-9`, halt:
```
"You've run team-os-setup 9 times today. To continue, either:
  - Delete an old branch: git branch -D team-os-setup-{date}-N
  - Wait until tomorrow.
Either way, your current state.yaml is preserved."
```

Run: `git checkout -b "${BRANCH}"`. Save branch name to `state.yaml: branch_name`.

---

## Step 3 — Resume mechanism

If `state.yaml` already exists at preflight start AND `last_gate_passed > 0`:

### Step 3.0 — Branch check

**Before** offering the resume choice, check the user's current branch matches `state.yaml: branch_name`:

```bash
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
EXPECTED_BRANCH={state.yaml: branch_name}
```

If `CURRENT_BRANCH != EXPECTED_BRANCH`:

1. If working tree is clean (`git status --porcelain` empty), AskUserQuestion:
   ```
   Header: "Branch mismatch on resume"
   Description: "This run's commits are on `{expected_branch}` but you're currently on `{current_branch}`. Auto-commits at the next gates would land on the wrong branch — silent corruption of git history. Pick how to proceed."

   Options:
     - "Check out `{expected_branch}` and continue resume" (Recommended)
     - "Cancel — I'll switch branches manually then re-run --resume"
   ```
   On "check out": run `git checkout {expected_branch}`. Verify success before continuing.

2. If working tree is dirty: halt with: *"You're on `{current_branch}` with uncommitted changes; this run's commits should be on `{expected_branch}`. Commit or stash your work, then re-run `--resume`."* No auto-checkout — too dangerous with dirty tree.

If `CURRENT_BRANCH == EXPECTED_BRANCH`: proceed to the resume offer.

This branch check is non-negotiable. Without it, the orchestrator is one user-distraction away from silently committing to `main` — the failure mode that motivates having a recovery branch in the first place.

### Step 3.1 — Resume offer

```
Header: "Found prior run from {state.created_at}, last passed Gate {last_gate_passed}."
Description: "{N} commits already exist on branch {branch_name}. Resume picks up at Stage {last_gate_passed + 1}."

Options:
  - "Resume from Gate {last_gate_passed + 1}" (Recommended)
  - "Restart from Stage 1" (archives state.yaml to state-{timestamp}.yaml; fresh run)
  - "Cancel"
```

On resume: skip Steps 0–2 (preflight + branch creation already done; branch check above just verified you're on it); skip already-completed stages; dispatch to next sub-skill.

On restart: archive old state.yaml; create fresh one; continue normally (Step 2 creates a new branch).

---

## Step 3.5 — Build-mode acknowledgment (L1 default; L2/L3 are upgrade-only)

Fresh runs always build **L1 (pointer-only)** — sources stay untouched. L2 and L3 are post-L1 upgrade paths invoked separately as `--upgrade=L2` / `--upgrade=L3 --confirm`. This separation matters:

- **L1 fresh build** doesn't require pre-existing source files at predictable slots — it just creates pointers.
- **L2/L3 upgrade** requires an existing L1 to clone/move source files INTO. There is no L1 to upgrade on a fresh run.

Surfacing L2/L3 as choices during preflight breaks this assumption: users picking "L2 — copy now" on a fresh run hit a dead end at Stage 4 ("where's the L1 you want to upgrade?"). The fresh-run path always builds L1; upgrades are post-build, separate-invocation flows.

Set `state.yaml: build_mode = "L1"` automatically. No AskUserQuestion needed for the fresh-run case.

**Optional preview message** (informational only, no choice required):

```
Building L1 first — pointer-only, lowest risk, sources untouched.

After your first build, you can upgrade:
  /team-os-setup --upgrade=L2          # copy sources into L1 slots (light-draft)
  /team-os-setup --upgrade=L3 --confirm  # move sources into L1 (destructive light-draft)

Both upgrade modes are also surfaced in the final closer.
```

**For `--upgrade=L2` / `--upgrade=L3 --confirm` invocations:** this Step 3.5 is irrelevant; those modes route directly to Stage 4's L2/L3 light-draft per the invocation table.

---

## Step 4 — Expectation-setting preview

Before the start question, print:

```
Quick orientation:

Stage 1 — Research        ~10 questions, ~8 min   (read your folders + interview)
Stage 2 — Propose         ~1 question, ~3 min     (versioning gap question)
Stage 3 — Advanced        ~10 decisions, ~10 min  (architecture choices)
Stage 4 — Execute         build + validate, ~5 min

Total ~25 min. Auto-commits at every gate so you can `git checkout main` to revert.
```

Then AskUserQuestion:

```
Header: "Ready to start the deep-dive setup?"
Options:
  - "Yes — start with Stage 1 research"
  - "Walk me through what'll happen first" (re-prints the orientation, then re-asks)
  - "Not now" (exits cleanly; branch already exists but no commits yet)
```

---

## Step 5 — Initialize state.yaml

If state.yaml doesn't exist (i.e., not a resume), create it. Parse the invocation flags + write them into state.yaml so sub-skills see them:

```yaml
schema_version: 1
created_at: {ISO-8601}
updated_at: {ISO-8601}
branch_name: {branch}
commit_mode: "per-gate"        # or "end-of-build" if user picked that fallback
build_mode: "L1"                # always L1 on fresh runs (Step 3.5); --upgrade=L2 / --upgrade=L3 invocations skip Step 5 entirely and route to Stage 4 directly
quick_mode: {true|false}        # true if --quick was in the invocation
variant: "default"|"personal"   # "personal" if --variant=personal was in the invocation
sparse_checkout:                # set in Check 0.5
  cone_mode: {true|false}
  auto_add: {true|false}
mcp_tools_indexed: [...]        # from Step 0.2
last_gate_passed: 0
gates_passed: []
mid_stage_checkpoint: null
artifacts: {}
outstanding_questions_log: []
```

Atomic write (`state.yaml.tmp` → rename).

**Flag parsing rule:** the orchestrator parses the user's full invocation string at preflight start. `--quick`, `--variant=personal`, `--upgrade=L2`, `--upgrade=L3 --confirm`, `--deepen <path>`, `--audit`, `--resume`, `--refresh-pointers` are mutually-non-exclusive in some combinations: `--quick --variant=personal` is valid; `--upgrade=L2 --quick` is invalid (upgrade modes skip Stage 3 entirely). On invalid combinations, halt with a clear error.

---

## Step 6 — Dispatch Stage 1

Invoke the `team-os-research` skill (separate skill invocation — NOT inline). Sub-skill reads state.yaml, runs reading pass + interview, writes findings.md, returns at Gate 1.

When sub-skill returns with Gate 1 user choice:

| User choice | Orchestrator action |
|---|---|
| Approve and continue | Auto-commit; dispatch Stage 2 |
| Iterate | Sub-skill handles iteration internally; orchestrator just waits for next return |
| Stop here | Save state; exit cleanly with resume instructions |

### Auto-commit at Gate 1

If `commit_mode == "per-gate"`:
```bash
git add team-os-build/
git commit -m "team-os: stage 1 (research) approved — {{company.slug}} baseline mapped, {N} source folders inventoried"
```

**Commit-failure handling:**
- If `git commit` exits non-zero: read git's error output. **Halt the orchestrator.** Do NOT advance `last_gate_passed`.
- Surface to user with options:
  ```
  "The auto-commit failed:
    {git error verbatim}
  
  Pick:
    - Fix and retry the commit (most common — pre-commit hook complained)
    - Switch to 'single commit at end of build' mode (commits batch at Gate 4)
    - Halt and exit (state preserved; resume later)"
  ```
- On retry: re-run `git add` + `git commit`; if still fails, halt.
- On switch: set `state.yaml: commit_mode = "end-of-build"`; advance `last_gate_passed` (no commit yet, but state is in clean shape).
- On halt-and-exit: preserve state; print resume instructions.

If `commit_mode == "end-of-build"`: skip the commit; just advance `last_gate_passed`. Save commit for Gate 4.

After successful commit (or skip), update `state.yaml: gates_passed[0]` with `{stage: 1, timestamp, commit_sha}` (or `null` for skip mode) and `last_gate_passed = 1`.

---

## Step 7 — Dispatch Stage 2 (`team-os-propose`)

Same pattern as Stage 1. Auto-commit message:
```
"team-os: stage 2 (propose) approved — architecture personalized to {{company.slug}}, {N} feature areas, routing tier: {tier}"
```

Update `gates_passed[1]`, `last_gate_passed = 2`.

---

## Step 8 — Dispatch Stage 3 (`team-os-advanced`)

Same pattern. Auto-commit message:
```
"team-os: stage 3 (advanced) approved — {N} decisions resolved, blueprint finalized"
```

**Note on mid-stage checkpoints:** Stage 3 writes `mid_stage_checkpoint.last_decision_resolved` after each of its ~10 decisions. The orchestrator does NOT commit on these mid-stage writes — only at the gate. State is saved per-decision (atomic), but commits happen per-gate.

Update `gates_passed[2]`, `last_gate_passed = 3`.

---

## Step 9 — Dispatch Stage 4 (`team-os-execute`)

**Sparse-checkout auto-add (if enabled).** Stage 4 Step 2 will confirm `target_l1_path` with the user. Once that path is finalized in `state.yaml: target_l1_path`, the orchestrator (BEFORE Stage 4 starts writing files):

1. Read `state.yaml: sparse_checkout.cone_mode` and `sparse_checkout.auto_add` (set in Check 0.5).
2. If `cone_mode == true` AND `auto_add == true`: run `git sparse-checkout add {target_l1_path}`. Verify with `git sparse-checkout list` — the added path should appear.
3. If `cone_mode == true` AND `auto_add == false`: prompt the user one more time *"Have you added `{target_l1_path}` to your sparse-checkout cone? Without it, the build will commit but not appear on disk."* — wait for explicit yes before dispatching Stage 4.
4. If `cone_mode == false`: skip; nothing to do.

This is the load-bearing piece — without it, Gate 4 hands the user a folder that's empty in their file explorer.

Same gate pattern as previous stages. Auto-commit message:
```
"team-os: stage 4 (execute) approved — L1 environment built at {target_l1_path}"
```

If `commit_mode == "end-of-build"`: NOW is the single batched commit. Add EVERYTHING in `team-os-build/` AND the new L1 sibling directory:
```bash
git add team-os-build/
git add {target_l1_path}
git commit -m "team-os: L1 build complete for {{company.slug}}

Stages 1-4 completed. {N} folders, {M} CLAUDE.md files, {K} sample artifacts.
Routing tier: {tier}. Multi-product: {mode}. Versioning: {choice}.
"
```

Update `gates_passed[3]`, `last_gate_passed = 4`.

---

## Companion-skill handoffs

The team-os-setup chain pairs with two product-os skills via documented handoffs. They keep the L1 living instead of stale.

| Companion skill | When it triggers | What it suggests |
|---|---|---|
| `/enhance-context` | After merging new signal into a context file that matches a `deferred_items[]` location | *"Run `/team-os-setup --deepen {location}` to scaffold the deferred folder."* |
| `/granola-pull` (or any meeting-extraction skill) | After distilling a transcript that lands in a folder without `meetings/_index.md` | *"This folder doesn't have a meetings index yet. Run `/team-os-setup --deepen {folder}` or `mkdir meetings/ && touch meetings/_index.md`."* |

The handoff direction is **always companion → team-os-setup**. The orchestrator doesn't proactively invoke companions; it just documents the entry points users land on naturally. See the F8 fix in `_principles.md` for the doctrine.

---

## Step 9.5 — Deepen mode (`--deepen <folder-path>`)

Triggered by `/team-os-setup --deepen <folder-path>` invocation. Skips Stages 1–3 entirely; runs a scoped Stage 4.

### Pre-checks

1. Find the active L1: read `state.yaml: target_l1_path`. If state.yaml or target path is missing → halt: *"No prior `/team-os-setup` run detected. Run `/team-os-setup` once first to build L1, then deepen specific folders."*
2. Validate `<folder-path>` against `state.yaml: deferred_items[]`:
   - **Exact match on `location`** → proceed.
   - **No match** → AskUserQuestion:
     ```
     Header: "Folder not in deferred list"
     Description: "`{folder-path}` isn't tracked as a deferred item in state.yaml. Continue anyway (creates as net-new), or pick from the deferred list?"

     Options:
       - "Show deferred items and pick one"
       - "Create as net-new" (proceed)
       - "Cancel"
     ```
   - **Partial match (substring)** → suggest the matching deferred items and confirm.

### Scoped build

Dispatch `team-os-execute` in **deepen mode** with these arguments:
- `target_path`: the resolved folder path (relative to L1 root)
- `deferred_item`: the matched entry from `state.yaml: deferred_items[]` (if any), so the sub-skill can read `reason` + `trigger` to tune the scaffold's CLAUDE.md note.

Stage 4 deepen-mode behavior:
1. `mkdir -p {target_path}` (relative to L1 root).
2. Generate the work-folder `CLAUDE.md` using `_templates/claude-md-folder.md`. Hot-path team can be inherited from the parent function CLAUDE.md or left as `[NOT YET FILLED]`.
3. Scaffold `decisions/_index.md` and `meetings/_index.md` per F4 default-on rule.
4. Add a one-line entry to the parent CLAUDE.md doc index pointing at the new folder.
5. If the deferred item had a `pre_fill_files` array (set by Stage 1's reading pass), surface them: *"Stage 1 found these files relevant to this folder: {list}. Want me to copy them in (becomes L2-scoped for this folder)?"* AskUserQuestion to confirm.
6. Remove the matched entry from `state.yaml: deferred_items[]` (it's no longer deferred).
7. Append a row to the L1's `ai-ops/roadmap/backlog.md` (or equivalent) noting the deepen action with date.

### Auto-commit

```bash
git add {target_path}/ team-os-build/
git commit -m "team-os: deepen {target_path} — {deferred_item.id || 'net-new'}"
```

### Closer

Print:
```
✓ Deepened: {target_path}
  - CLAUDE.md scaffolded
  - decisions/_index.md + meetings/_index.md created
  - {N} pre-fill files {copied | skipped}
  - state.yaml: deferred_items[] updated ({remaining} items remain)

ONE NEXT STEP:
  Open {target_path}/CLAUDE.md and fill in the [NOT YET FILLED] markers when context lands.
```

The deepen flow is what `/enhance-context` should suggest when it finds substantial new signal in a deferred area (see F8 handoff doctrine).

---

## Step 10 — Final closer

Print (Hannah-warm tone, concrete next step):

```
✓ Team OS Level 1 built at: {{target_l1_path}}

ONE NEXT STEP:
  Open {{target_l1_path}} in your IDE and ask Claude:
    "What's our team structure?"
  
  Claude should walk your CLAUDE.md and team-org.yaml to answer. If the response cites your actual files, the scaffolding works.

  Or if you want to learn the why before the what:
    "Explain why this repo is structured the way it is, and what could be improved."
  
  This is the beginner's-mindset prompt — don't just trust scaffolding you didn't build; ask Claude to teach you the reasoning so you can iterate.

When you're ready to share with the team:
  cd {{repo-root}}
  git log {{branch_name}}                # review the commits
  git push -u origin {{branch_name}}     # push (manual — never automatic)
  
Want more?
  /team-os-setup --upgrade=L2     Generates manual `cp` commands to clone source files into L1 slots.
                                  v1 outputs the commands; v2 will execute them with safety.
  
  /team-os-setup --upgrade=L3 --confirm
                                  Generates manual `cp + rm` commands for destructive moves.
                                  v1 outputs the commands; v2 will execute with rollback contracts.

Recovery:
  git checkout main               Reverts the working tree.
                                  Note: {{target_l1_path}} stays on disk — delete manually if you don't want it.

Maintenance cadence (Hannah's framing):
  Re-run `/team-os-setup --refresh-pointers` quarterly so MCP connections + source folders stay current.
  The OS is a living system — it gets better when the team updates it; it rots when it goes stale.
  (--refresh-pointers ships in v1.next.)
```

---

## Hard rules (orchestrator-specific)

- **NEVER** auto-push. Push is always user-initiated, surfaced in the closer only.
- **NEVER** advance `last_gate_passed` if the auto-commit failed. State integrity matters more than UX smoothness here.
- **NEVER** call sub-skills inline; always as separate skill invocations. Token isolation matters.
- **NEVER** skip preflight in default mode. Skip only when invoked with upgrade flags.
- **NEVER** force-create the branch (`git checkout -B`) — always use `-b` so existing-branch collision is detectable.
- **ALWAYS** use the `AskUserQuestion` tool for every preflight choice (auth fallback, gitignore exception, sparse-checkout add, build-mode L1/L2/L3, resume/restart, start orientation). Inline prose questions blend into the chat — the AskUserQuestion UI is what makes the question unmistakable.
- **ALWAYS** offer the gitignore exception over silent commit failure. The user will hate silent failure more than 1 extra question.
- **ALWAYS** print the recovery line (`git checkout main`) somewhere in every gate's user-facing output, so it's never more than one screen-scroll away.

---

## Sub-agent dispatch decisions (orchestrator-owned)

Per `_principles.md` Context Budget & Sub-Agent Dispatch section:

The orchestrator runs:
- Preflight checks (inline — small, fast)
- AskUserQuestion calls (must be inline)
- Auto-commit operations (inline)
- State.yaml reads/writes (inline)
- Gate-handling decisions (inline)

The orchestrator dispatches via sub-agent (Agent tool):
- (None of the orchestrator's own work — all sub-agent dispatch happens INSIDE sub-skills, which decide their own dispatch needs per `_principles.md`)

The orchestrator INVOKES (separate skill calls, not sub-agents):
- `team-os-research` (Stage 1)
- `team-os-propose` (Stage 2)
- `team-os-advanced` (Stage 3)
- `team-os-execute` (Stage 4)

Each sub-skill is a separate `Skill()` invocation (not an Agent tool call). This gives each sub-skill its own context window and isolation. State carries between them via `state.yaml`.

---

## Failure mode summary

If anything halts mid-run, the orchestrator's exit message is always:

```
Halted at: {step}
Reason: {one-sentence reason}
What's preserved:
  - state.yaml at {path}
  - branch {branch_name} with {N} commits
What to do:
  - Fix the underlying cause (see error above)
  - Re-run /team-os-setup --resume to continue from where you stopped
  - Or `git checkout main` to abandon — nothing's been pushed
```

This ensures users never get stuck wondering what happened.

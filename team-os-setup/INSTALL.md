# Setting up your own Team OS

30 minutes of setup, hours saved every week.

---

## What this is

A 4-stage skill chain that walks you through standing up a Team OS for your company — folders, CLAUDE.md files, a feature index, sample artifacts personalized to your products. By the time you're done, Claude Code knows your team, your products, your customers, and your conventions, so every session after this stops with the "let me re-explain my context" tax.

Run it once per company. The output is a new sibling directory next to your repo, scaffolded and ready to use.

---

## Prereqs

Three things, in this order:

1. **Claude Code installed** — claude.com/code if you don't have it yet
2. **`git init` done in your repo** — the chain creates a recovery branch and auto-commits at every stage
3. **GitHub access set up** — `gh auth login` if you have the GitHub CLI, or just confirm you can push to a repo you own. Not strictly required for v1, but recovery depends on it.

Optional but recommended:
- One or two MCP tools connected (Slack, Granola, Otter, Notion). The chain detects what's connected and offers to index them in your team-org.yaml + root CLAUDE.md.

---

## How to run it

```
/team-os-setup
```

Or just say to Claude Code: *"set up team os"* / *"build team os"*. The skill auto-triggers.

### Common invocation flavors

| Use case | Invocation |
|---|---|
| Full chain, full questioning (Hannah's recommended default) | `/team-os-setup` |
| Personal use at a company, simplest setup (let recommended defaults pick Stage 3 architecture) | `/team-os-setup --quick` |
| Solo / personal side projects (skips team interview + audience reframe; keeps `team/` for collaborators / mentors / beta users) | `/team-os-setup --variant=personal` |
| Solo, fastest path | `/team-os-setup --variant=personal --quick` |
| Small team / department starter | `/team-os-setup --quick` |
| Already ran once; clone source files into the L1 slots | `/team-os-setup --upgrade=L2` (generates manual cp commands; v2 will execute) |
| Already ran once; move sources into L1 (destructive) | `/team-os-setup --upgrade=L3 --confirm` (generates manual cp + rm commands; v2 will execute with rollback) |
| Scaffold ONE deferred folder (per-folder targeted deepening) | `/team-os-setup --deepen <folder-path>` |
| Resume after exit | `/team-os-setup --resume` |
| (v1.next stub) Audit existing L1 vs. current best-practice | `/team-os-setup --audit` |

`--quick` and `--variant=personal` compose: combining both runs the personal-pruned shape with auto-applied recommended defaults. Stage 1 always runs full questioning regardless of mode (the user-context capture is what makes everything else work).

---

## What happens (4 stages, ~25 minutes)

### Stage 1 — Research (~8 min)

Reads your existing folders + GitHub URLs, then asks ~10 questions — but only what couldn't be extracted from the read. Confirm what's there; answer what's missing. Captures company, products, team functions, daily collaborators, customers, multi-product mode, AI context filename. **Gate 1:** approve to continue.

### Stage 2 — Propose (~3 min)

Shows you the proposed folder tree, personalized with your products and team functions. Asks the versioning gap question (how PRD v1 → v2 carries forward). Routing tier (`inline` / `split` / `pointer-only`) is chosen based on your org size. **Gate 2:** approve to continue.

### Stage 3 — Advanced (~10 min)

Walks you through ~10 architecture decisions — meeting transcripts placement, customer accounts depth, bug investigation cadence, retros cadence, etc. Each one: Hannah's relevant principle, the trade-off, recommended default, you confirm or override. **Gate 3:** approve to build.

### Stage 4 — Execute (~5 min)

Builds the L1 environment in a sibling directory. Folders, CLAUDE.md at every required level, feature-index.yaml, data-catalog.yaml, team-org.yaml, 3 personalized sample artifacts (PRD, RFC, plan), and a `_pointers.md` index back to your source folders. Validates the build (no unresolved placeholders, no empty CLAUDE.md). Previews L2/L3 upgrade paths. **Gate 4:** open in IDE, ask Claude a test question, finalize.

---

## Recovery

Auto-commits run at every gate to a branch named `team-os-setup-YYYY-MM-DD`. If anything looks wrong:

```
git checkout main
```

That reverts the working tree. The L1 sibling directory stays on disk — delete it manually if you don't want it. Your main branch never sees any of the chain's commits unless you explicitly push.

If a commit fails mid-run (pre-commit hook, gitignored path, signing error): the orchestrator halts at that gate, surfaces the error, and gives you three options — retry, switch to "single commit at end of build" mode, or exit with state preserved for resume.

To resume after exit:
```
/team-os-setup --resume
```

Picks up at the gate after the last one you passed.

---

## Going deeper

After your first run, the L1 environment is a working Team OS. You can:

- **Add real artifacts** — drop your actual PRDs into `product-development/product/PRDs/`, your real RFCs into `engineering/rfcs/`, etc. Sample artifacts are scaffolds; replace them.
- **Populate `feature-index.yaml`** — every cross-functional feature gets one entry mapping all its artifacts.
- **Populate `team-org.yaml`** — full org with roles, depts, reporting lines. Sessions auto-add new people as they come up.
- **Connect more MCP tools** — Slack, Granola, Notion. They show up in `_pointers.md`.
- **Pair this with personal writing-guide skills in `~/.claude/skills/`** — Hannah's pattern: each doc type (strategy doc, PRD, brief) gets its own skill at the user level so your voice stays consistent across teams. Team-shared skills (the ones everyone on the team should run the same way — e.g., `customer-call-summary`) live in your repo's `.claude/skills/`. Two homes, two scopes — keep them straight: `~/.claude/skills/` is *your* voice; repo `.claude/skills/` is *the team's* discipline.

### Two practical things that bite people

1. **Skills auto-invoke ~70% of the time.** When a plan really needs a specific skill or command run, name it explicitly in the plan ("use the `customer-call-summary` skill for this") rather than hoping Claude picks it up. Don't leave anything to chance on long-running work.
2. **Customer call summaries, not raw transcripts.** Each customer call gets a short summary in `product/customers/accounts/{customer}/`. Raw transcripts stay in Granola/Otter. Claude reads the summary; it only reaches for the transcript when the summary doesn't answer the question. Saves context every session.

When you want to clone your existing docs into the L1 (rather than just pointing back to them):
```
/team-os-setup --upgrade=L2
```

When you want to actually move them (clone-and-delete originals):
```
/team-os-setup --upgrade=L3 --confirm
```

L2 and L3 ship as **light drafts in v1** — they generate manual command lists rather than executing destructive ops. v2 hardens both with atomic rollback contracts. That's deliberate: the path is alive, but you're in control of when destructive things actually run.

---

## What's a Team OS, anyway?

Hannah Stulberg's framing, paraphrased from her *In the Weeds* substack series and the [team-os-example-repo](https://github.com/in-the-weeds-hannah-stulberg/team-os-example-repo):

- The repo isn't documentation — it's the system that makes the team faster every day.
- Scaling isn't about making yourself faster; it's about making your team better.

The Team OS is hierarchical CLAUDE.md files + cross-functional indexes (feature-index.yaml + data-catalog.yaml) + a discipline of keeping context lean. Claude reads what's relevant when it's relevant. New hires onboard against the same files Claude does. When someone fixes a misunderstanding in a CLAUDE.md, every future session benefits.

> Direct quotes are listed in `_voice.md` with substack-source attribution. If you cite Hannah's work externally, link to her substack rather than to this skill.

---

## What's an extension beyond Hannah's published guidance?

This skill ships several opinionated defaults that go beyond Hannah's example repo. Each is flagged in the proposal/blueprint outputs so you know which decisions are Hannah-canon and which are this skill's opinionated defaults.

1. **Org-size-aware team routing** — Hannah's example assumes ~10 ppl and inlines everyone in the root CLAUDE.md. At 50+ ppl that breaks the lean-context budget. The skill picks one of three tiers (`inline` / `split` / `pointer-only`) based on your org size, with a `team-org.yaml` registry at the larger tiers.

2. **Build-on-demand defaults for `customers/`, `bug-investigations/`, `sales-enablement/`** — Hannah's example scaffolds these. The skill defers them until first use, applying her "no dead context" principle. Override anytime in Stage 3.

3. **Reading-order doctrine in root CLAUDE.md** — every generated root CLAUDE.md has a "Reading order for Claude (any task)" section near the top with a 9-step sequence (mandatory orientation pass 1–4 vs scoped deep-reads 5–9). Without it, Claude infers its own order; with it, you control which files are mandatory vs scoped.

4. **Subfolder conventions table in root CLAUDE.md** — every work folder uses the same subfolder shape (`decisions/`, `meetings/`, `prds/`, `plans/`, `research/`, `reference/`) with consistent filename patterns. PMOS skills follow this on save (see `_doctrine/output-routing.md`).

5. **Auto-scaffolded `decisions/_index.md` + `meetings/_index.md` per work folder** — continuous-enrichment surfaces get a designated home from day one. Empty index files are structural skeleton, not dead context.

6. **`deferred_items[]` in state.yaml** — machine-readable mirror of `_backlog.md`. Drives `--deepen` mode + `/enhance-context` handoff.

7. **`--quick` and `--variant=personal` modes** — quick auto-applies recommended Stage 3 defaults (skips per-decision walk); personal swaps team-shaped questions for solo/audience-shaped ones and prunes the tree.

---

## Principles worth remembering while you build

Drawn from Hannah Stulberg's published Team OS guidance — see `_voice.md` for verbatim quotes with substack attribution:

- Every line in a CLAUDE.md file costs thinking room.
- What you see is what Claude sees — no hidden surprises.
- GitHub increasingly plays the collaboration-doc role that Google Drive used to.

---

*If you hit something this guide doesn't cover, file an issue on the skill's source repo at [github.com/MrFreekour/ai-skills](https://github.com/MrFreekour/ai-skills). v1.next will fold the answer back in.*

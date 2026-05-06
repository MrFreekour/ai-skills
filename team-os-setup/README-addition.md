# README addition for `team-os-example-repo`

This file is NOT a separate readme — it's the section to append to Hannah's existing `README.md` at the example-repo root. The discoverability surface for new adopters who land on the GitHub page.

**Hannah:** copy-paste the section below into your existing README, ideally near the top so adopters see it before scrolling past the example structure.

---

## Setting up your own Team OS

Run `/team-os-setup` in Claude Code to bootstrap your own Team OS in about 25 minutes. The skill reads your existing folders + GitHub repos, asks ~20 questions across 4 stages (research → propose → advanced → execute), and builds a Level 1 environment in a sibling directory — folders, CLAUDE.md files, feature-index.yaml, optional personalized sample artifacts.

Auto-commits at every stage so `git checkout main` reverts everything if you don't like the result.

### Pick the flavor that fits

- **`/team-os-setup`** — full chain, full questioning. Hannah's recommended default.
- **`/team-os-setup --quick`** — full chain, but Stage 3's architecture decisions auto-apply recommended defaults. Best for personal-use-at-a-company or small-team starter setups.
- **`/team-os-setup --variant=personal`** — solo / personal side-projects shape: skips the team interview, swaps customer hierarchy for "audience", drops `onboarding-guides/`. Keeps `team/` (collaborators / mentors / beta users still belong somewhere) and `retros/` (solo retros are still valuable). Combine with `--quick` for the fastest path.
- **`/team-os-setup --deepen <folder>`** — after your L1 exists, scaffold one deferred folder when you're ready for it (paired with `/enhance-context`).

See [`INSTALL.md`](INSTALL.md) for prereqs, recovery, L2/L3 upgrade paths, and the full extensions list (reading-order doctrine, subfolder conventions, auto-scaffolded `decisions/` + `meetings/`, etc.).

> *"30 minutes of setup saves hours every week in actual use."*

---

*End of README addition. Everything below this line in the rendered README is Hannah's existing content.*

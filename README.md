# ai-skills

Public repo of agent skills I've built for Claude Code that are useful beyond my own setup.

Currently shipping:

- [`team-os-setup/`](team-os-setup/) — Bootstrap a Team OS for your company in ~25 minutes. Modeled on [Hannah Stulberg's Team OS framework](https://github.com/in-the-weeds-hannah-stulberg/team-os-example-repo). Reads your existing folders + GitHub repos, asks ~20 questions across 4 stages (research → propose → advanced → execute), and builds a Level 1 environment in a sibling directory. Auto-commits at every stage so `git checkout main` reverts everything.

- [`_doctrine/output-routing.md`](_doctrine/output-routing.md) — Companion doctrine: how PMOS / product-os skills should follow `CLAUDE.md` routing rules when saving artifacts.

## Install

Clone or copy the skill folder into your `~/.claude/skills/` (user-level) or your repo's `.claude/skills/` (team-level):

```bash
git clone https://github.com/MrFreekour/ai-skills.git
mkdir -p ~/.claude/skills/
cp -r ai-skills/team-os-setup ~/.claude/skills/
```

Then in Claude Code, type `/team-os-setup` or say *"set up team os"*.

See [`team-os-setup/INSTALL.md`](team-os-setup/INSTALL.md) for prereqs, recovery, the L2/L3 upgrade paths, and the full extensions list (reading-order doctrine, subfolder conventions, auto-scaffolded `decisions/` + `meetings/`, etc.).

## Invocation flavors

| Use case | Invocation |
|---|---|
| Full chain, full questioning (recommended default) | `/team-os-setup` |
| Personal use at a company, simplest setup (auto-applies recommended Stage 3 defaults) | `/team-os-setup --quick` |
| Solo / personal side projects (skips team interview, audience reframe) | `/team-os-setup --variant=personal` |
| Solo, fastest path | `/team-os-setup --variant=personal --quick` |
| Already ran once; clone source files into L1 | `/team-os-setup --upgrade=L2` |
| Already ran once; move sources into L1 (destructive) | `/team-os-setup --upgrade=L3 --confirm` |
| Scaffold ONE deferred folder (per-folder targeted) | `/team-os-setup --deepen <folder-path>` |
| Resume after exit | `/team-os-setup --resume` |

## Attribution

The Team OS framework — hierarchical CLAUDE.md files, the NEVER list, the 80% rule, format-by-depth, the team-org/feature-index/data-catalog pattern — is Hannah Stulberg's published work in her [*In the Weeds* substack](https://intheweeds.substack.com/) and [`team-os-example-repo`](https://github.com/in-the-weeds-hannah-stulberg/team-os-example-repo). This skill is a 4-stage interview that helps you instantiate her framework for your team, with opinionated extensions (org-size routing tiers, build-on-demand defaults, reading-order doctrine, subfolder conventions, `--variant=personal`) clearly marked as such in every output.

If you cite Hannah's work externally, link to her substack rather than to this skill.

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Issues + PRs welcome. The skill was tested on real adoption; the [`team-os-setup/_principles.md`](team-os-setup/_principles.md) source-attribution table separates Hannah-canon defaults from this skill's Extensions.

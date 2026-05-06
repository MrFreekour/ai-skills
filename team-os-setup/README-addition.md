# README addition for `team-os-example-repo`

Section to append to Hannah's existing `README.md`. Designed to fit her minimalist style — short, scannable, links out for detail.

Suggested placement: as a `## Bootstrap your own` section after **Where to learn more**, before **License**. The example repo stays the canonical reference; this just gives adopters a path forward.

---

## Bootstrap your own

If you want a Team OS like this for your own team, [`MrFreekour/ai-skills`](https://github.com/MrFreekour/ai-skills) ships a community-built Claude Code skill (`/team-os-setup`) that walks you through it.

The skill reads your existing folders, asks ~20 questions across four stages, and builds a Level 1 environment in a sibling directory — folders, CLAUDE.md files, indexes — modeled on this repo's structure. About 25 minutes; auto-commits at every stage so `git checkout main` reverts. Extensions beyond the published framework (org-size routing, build-on-demand defaults, `--variant=personal` for solo work) are flagged in every output so you can tell what's repo-canon vs. opinionated additions.

> *"30 minutes of setup saves hours every week in actual use."*

---

*End of README addition.*

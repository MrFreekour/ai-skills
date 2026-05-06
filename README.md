# ai-skills

**The shared brain your team needs, without the setup tax.**

AI sessions get smarter when they can read your team's actual structure — your products, your customers, your conventions, your decisions. Most teams don't have that organized in a way AI can use, so every session starts with the same context tax: re-explain who's who, what we ship, what we already decided.

[Hannah Stulberg's Team OS](https://github.com/in-the-weeds-hannah-stulberg/team-os-example-repo) is the framework that fixes it — hierarchical CLAUDE.md files plus cross-functional indexes that turn your team's collective context into something AI can self-serve. New hires onboard against the same files Claude does. When someone fixes a misunderstanding once, every future session benefits.

This repo ships **`/team-os-setup`** — a Claude Code skill that walks you through standing one up. About 25 minutes, four stages, and you have a personalized Team OS modeled on Hannah's framework, ready to live alongside (or inside) your existing repo. Built for the person who just found Team OS and wants to try it without committing their whole filesystem on day one.

## Choose your approach

Build progressively. Start safely, deepen as the structure earns your trust:

- **L1 — pointers (start here).** Builds a new sibling folder with CLAUDE.md files that *point* at your existing docs. Nothing moves, nothing copies. Don't like the result? `git checkout main` and it's gone. This is how you test the skill without disturbing what's already working.
- **L2 — copy.** Once L1 feels right, clone source files into the L1 slots so it has working copies. Originals stay where they are. The L1 becomes your AI-facing front door while the rest of your filesystem keeps working as you remember it.
- **L3 — commit.** Move sources into the L1 — make it your single source of truth. Run only after L2 has been stable and you're sure. Destructive; v1 generates manual `cp + rm` lists for you to review before anything runs.

Most teams will pause at L1 indefinitely, and that's the point. L2 and L3 are there when you outgrow pointers — not steps you have to take.

## Install

```bash
git clone https://github.com/MrFreekour/ai-skills.git
mkdir -p ~/.claude/skills/
cp -r ai-skills/team-os-setup ~/.claude/skills/
```

In Claude Code, type `/team-os-setup` (or say *"set up team os"*) and it auto-triggers.

## Run it

| Use case | Invocation |
|---|---|
| Full chain, full questioning | `/team-os-setup` |
| Personal use at a company, simplest setup | `/team-os-setup --quick` |
| Solo / personal side projects | `/team-os-setup --variant=personal` |
| Solo, fastest path | `/team-os-setup --variant=personal --quick` |
| Already ran once; clone sources into L1 | `/team-os-setup --upgrade=L2` |
| Already ran once; move sources into L1 | `/team-os-setup --upgrade=L3 --confirm` |
| Resume after exit | `/team-os-setup --resume` |

See [`team-os-setup/INSTALL.md`](team-os-setup/INSTALL.md) for prereqs, recovery, and the full extensions list.

## Attribution

Built on the Team OS framework by [Hannah Stulberg](https://hannahstulberg.substack.com/) — published in her *In the Weeds* substack and the [`team-os-example-repo`](https://github.com/in-the-weeds-hannah-stulberg/team-os-example-repo). Quotes in the skill come from her published work; see [`team-os-setup/_voice.md`](team-os-setup/_voice.md) for verbatim sources.

Extensions beyond Hannah's published guidance (org-size routing tiers, build-on-demand defaults, `--variant=personal`, reading-order doctrine, subfolder conventions) are clearly marked in every output. The [`team-os-setup/_principles.md`](team-os-setup/_principles.md) source-attribution table separates Hannah-canon rules from this skill's opinionated additions.

## License

MIT — see [LICENSE](LICENSE).

---

Built by **Andrew Choflet** ([LinkedIn](https://www.linkedin.com/in/andrewchoflet/) · [GitHub](https://github.com/MrFreekour)).

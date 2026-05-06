# Step 1 — Pull context

Gather the inputs the update will draw from. Don't synthesize yet; just collect.

## What to read

1. **Previous update** — the most recent file in `output/`. Note its highlights and what the team committed to.
2. **Meeting summaries** from `product/meetings/{sprint-planning, standup, team-bi-weekly}/` — last 14 days only. Don't read transcripts, only summaries.
3. **Feature status changes** — diff `feature-index.yaml` against where it was 14 days ago (`git log -p feature-index.yaml --since="14 days ago"`).
4. **Customer signals** — if `customers/` is scaffolded, scan recent call summaries (last 14 days) across all named customers for recurring themes.

## What NOT to read

- Raw meeting transcripts
- Engineering RFCs (unless one was approved this period — link from update, don't summarize)
- Dashboards directly — just note which dashboards changed, link out

## Output of this step

A bullet list dropped into a scratch file (`reference/scratch-{date}.md` — gitignored) with:
- 3–5 things shipped or progressed
- 2–3 decisions made (with one-line rationale)
- 1–3 blockers / asks
- 1–2 customer-signal patterns (if relevant)

This scratch file is the input to Step 2.

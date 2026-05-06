# Workflow spec — bi-weekly update

## Inputs

- The previous bi-weekly update (in `output/` — most recent file)
- Recent meeting summaries from `product/meetings/` (last 14 days)
- Any shipped or in-progress features from `feature-index.yaml` (status changes since last update)
- Open blockers / risks the owner is tracking

## Outputs

- One markdown file at `output/{YYYY-MM-DD}.md`
- Optional: a Slack-ready summary (1-2 paragraphs) for the team channel
- Optional: an email-ready version for non-technical leadership

## Success criteria

- The audience reads it in under 5 minutes
- It surfaces decisions made in the last 2 weeks (with rationale)
- It surfaces blockers + asks (what the team needs from leadership)
- It links to source artifacts (PRDs, RFCs, dashboards) — doesn't repeat them inline
- It doesn't paste raw transcripts or long quotes

## Hannah's principles applied

- *80% rule:* the update is what audience needs on most weeks; deeper context lives in linked artifacts
- *No raw transcripts:* meeting notes are referenced via summary, not pasted in
- *No dead context:* if something didn't actually happen, don't include it for completeness

## Anti-patterns

- "Status update theater" — listing every meeting without surfacing decisions
- Marketing tone — "we're crushing it!"
- Burying the lede — leadership should know in the first paragraph what they need to act on

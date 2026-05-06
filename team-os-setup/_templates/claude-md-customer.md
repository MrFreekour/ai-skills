# {{customer.name}}

{{customer.one_line_segment}} *([NOT YET FILLED] — describe how this customer fits into your business: enterprise / mid-market / SMB, industry, size, ARR tier, or whatever segmentation matters to your team)*

## Key contacts

| Name | Role | When to engage them | Slack / email |
|---|---|---|---|
{{#each contacts}}
| {{name}} | {{role}} | {{when_to_engage}} | {{contact}} |
{{/each}}
{{#unless contacts}}
| [NOT YET FILLED] | | | |
{{/unless}}

## Account context

- **Segment:** {{customer.segment_or_not_filled}}
- **Started with us:** {{customer.start_or_not_filled}}
- **Status:** [active / at-risk / expansion / churned] *([NOT YET FILLED])*
- **Primary use case:** [NOT YET FILLED — what are they actually using your product for?]
- **Notable quirks:** [NOT YET FILLED — anything Claude should know about how to talk to or about this customer?]

## Folder shape

This customer's folder has two subfolders, mirroring Hannah's example pattern:

- `summaries/` — distilled call summaries, one per call (`{{date}}-{{topic-slug}}.md`). This is what Claude reads on most queries about {{customer.name}}.
- `transcripts/` — raw transcripts, only if your meetings tool doesn't link well. Hannah's preference: link out to Granola/Otter; this folder stays sparse.

## Doc index — what lives in this folder

This is what this CLAUDE.md is mostly for. Without this index, Claude runs explore agents to search the customer folder every time. The index points Claude directly to the right file.

{{#each docs_found}}
- `{{filename}}` — {{description}}
{{/each}}
{{#unless docs_found}}
- `summaries/` — *(empty for now; drop call summaries here as you take them)*
- `transcripts/` — *(empty; only use if your meetings tool doesn't link well)*
{{/unless}}

## Convention

- **Call summaries** live in `summaries/`. One file per call, named `{{date}}-{{topic-slug}}.md`. Use your team's `customer-call-summary` skill to enforce a consistent shape — that's what makes cross-customer analysis work.
- **Raw transcripts**: prefer linking out to your meetings tool ({{meetings_tool_or_not_filled}}) rather than pasting into `transcripts/`. The folder exists for the case where linking-out doesn't work.
- Hannah's framing: *"the customer folder doesn't go into transcripts — only summary files. Full transcript only loaded if the summary lacks the answer."*

---

*Customer-folder CLAUDE.md. The 80% rule: only what you need on most sessions about {{customer.name}} belongs here. Everything else lives in the linked files above.*

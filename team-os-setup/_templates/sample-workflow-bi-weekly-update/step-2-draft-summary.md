# Step 2 — Draft the summary

Take the scratch file from Step 1 and turn it into a draft.

## Structure (pick the version that fits your audience)

### Version A — leadership audience

```
## Headline (1 sentence — what changed this period)

## Shipped / progressed
- [item] — [1-line context]
- [item]
- [item]

## Decisions made
- [decision] → [why we picked it]
- [decision] → [why]

## Blockers / asks
- [what we need from leadership / partner team]
- [what's at risk if not resolved by {date}]

## Customer signal (if relevant)
- [pattern observed] — [how many customers / what %]
```

### Version B — partner-team audience

Same structure but heavier on integration points, dependency surface, and what *they* need to do or know.

### Version C — exec / board audience

Just the headline, the top 3 shipped items, and the asks. Drop the customer-signal section unless it's a strategic flag.

## Voice rules

- Lead with the decision or insight; support with evidence
- Specific names beat abstractions ("Sarah's approval needed on the auth migration" > "blocked on stakeholder review")
- Time-savings framing wins ("freed up 6 eng-days by deprecating X")
- No marketing tone

## Output of this step

A draft saved to `reference/draft-{date}.md` (gitignored). Step 3 picks highlights from it.

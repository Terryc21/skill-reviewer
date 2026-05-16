# Second-opinion workflow

Spec for `/skill-reviewer review <path> --second-opinion`.

The default review is one reviewer's perspective. A skilled reviewer still has blind spots: pet peeves they over-weight, classes of issues they're trained to ignore, or "this looks fine" reactions that mask real problems. The second-opinion pass dispatches an independent reviewer to challenge the first.

---

## When to use

Add `--second-opinion` when:

- The skill is heading to public release
- The first review felt too positive or too negative (you're suspicious of your own calibration)
- The author specifically asks for a tougher critique
- Stakes are high (production users, financial impact, data-loss risk)

Skip `--second-opinion` for routine pre-commit reviews. It roughly doubles review time and dispatches an additional agent.

---

## Workflow

### Step 1 — Produce the initial report

Run the full protocol from `reference/protocol.md`. Produce the report exactly as if `--second-opinion` were not set. Save it as the **draft** report.

The draft must include:
- All sections specified by `reference/output-format.md`
- A clearly identified "Top 3 conclusions" — the three findings (positive or negative) that most shape the report's verdict

If the report doesn't naturally surface three highest-leverage conclusions, identify them now. They're the claims a second reviewer should test hardest.

### Step 2 — Dispatch the second-opinion subagent

Spawn a Plan-type subagent (`Agent` tool, `subagent_type: Plan`) with the following prompt structure:

```
You are an independent reviewer auditing a skill at <path>. Another reviewer has already produced a draft report (provided below). Your job is NOT to redo the full review. Your job is to challenge the draft's three highest-leverage conclusions.

For each of the three conclusions:
1. Independently read the relevant files cited
2. Decide: do I agree, partially agree, or disagree?
3. If you disagree: what specifically did the original reviewer get wrong? File:line evidence required.
4. If you partially agree: what nuance is missing?

Also briefly note: did the original reviewer miss any major finding (positive or negative) that you'd flag as top-5-worthy?

Output a Markdown report with sections:
- Agreements (with brief rationale)
- Disagreements (with file:line evidence)
- Partial agreements (with the missing nuance)
- Missed findings (top-5-worthy items not in the original)

Be specific. Vague disagreement is worse than no disagreement. Cite file:line for every claim. Word budget: 600-1200 words.

Draft report follows:
---
<draft report content>
```

The subagent must independently read files — it cannot rely on the draft's quotes.

### Step 3 — Reconcile

After the subagent returns, update the report with a new section: **"Second-opinion reconciliation."** Place it after "Cross-file findings" and before "Quality signals."

The reconciliation section has four parts:

1. **Agreements** — list with one-sentence summaries. If the second reviewer agreed on all three top conclusions, note that explicitly.
2. **Disagreements** — for each disagreement, write a paragraph that:
   - States the original conclusion
   - States the second reviewer's counter-claim with their evidence
   - Resolves: which view is right, or note "unresolved — both views are defensible"
3. **Refinements** — partial agreements that improved the original finding. Update the relevant finding inline with a marker like "(refined per second opinion)".
4. **Missed findings** — items the second reviewer surfaced that weren't in the original. Add these to the appropriate sections (per-file, cross-file, recommended actions) with a marker like "(surfaced in second opinion)".

### Step 4 — Re-rank the Recommended Actions table

After reconciliation, re-run the ranking logic on the actions table. New findings get priority; downgraded findings move lower or get removed.

---

## Reconciliation rules

These rules prevent the second-opinion pass from becoming a tug-of-war:

- **If both reviewers agree** → keep the original finding, note agreement in reconciliation section.
- **If both reviewers disagree** → keep the original finding but flag the disagreement. Don't silently override either view. Note the disagreement clearly.
- **If the second reviewer cites stronger evidence** → update the finding to match. Mark it "(refined per second opinion)".
- **If the second reviewer surfaces a missed finding** → add it. Mark it "(surfaced in second opinion)".
- **Never delete a finding silently.** If a finding is removed after reconciliation, note it in the reconciliation section with the reason.

---

## When the subagent fails

If the Plan subagent returns an empty report, refuses, or produces vague disagreement without file:line evidence:

- Note "second-opinion pass produced no usable findings" in the reconciliation section
- Do NOT remove the section — its absence would imply the user got a single-reviewer report when they asked for two
- Recommend re-running with `--second-opinion` later, or with a different subagent type

If the subagent's findings contradict the draft based on misreading a file (e.g., they cited a line that says X but they describe it as Y), call this out in the reconciliation: "second reviewer claimed X but the file at <path>:<line> says Y; original conclusion stands."

---

## Output marker

Reports produced with `--second-opinion` should declare this in the TL;DR's opening line:

> "This review includes a second-opinion reconciliation pass. See § Second-opinion reconciliation for what changed between the draft and the final version."

This tells the author the report has higher confidence than a single-reviewer pass — and tells them where to read for the most controversial findings.

---

## Anti-patterns

- **Don't dispatch the second-opinion subagent before completing the draft.** The whole point is to challenge a complete first opinion. Half-drafts confuse the subagent.
- **Don't let the subagent rewrite the report.** It produces a challenge document; the main agent integrates the challenge into the final report.
- **Don't silently delete findings that survived reconciliation.** Disagreements are interesting and worth showing.
- **Don't run more than one second-opinion pass.** Three reviewers don't add proportional value. If the first pair disagrees strongly, escalate to the author for a human tiebreaker rather than dispatching a third reviewer.

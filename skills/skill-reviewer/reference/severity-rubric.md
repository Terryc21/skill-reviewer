# Severity rubric

Authoritative spec for how findings are labeled and scored in skill-reviewer reports. Defines severity colors, effort buckets, quick-win tag rules, and strength signals — all of which are consumed by the card format in `reference/output-format.md`.

---

## Severity colors

Use color emojis FIRST, then the label. Hybrid format scans well and carries precise meaning.

| Color | Label | Meaning |
|---|---|---|
| 🔴 | **CRITICAL** | Data loss, silent failure, security exposure, or feature does not work as documented. Must fix before publishing or shipping. |
| 🟡 | **HIGH** | Significant correctness, discoverability, or maintainability problem. Will cause user friction or quiet rot. Fix before next release. |
| 🟢 | **MEDIUM** | Clarity, redundancy, or polish issue. Worth fixing but not urgent. Bundle with the next round of edits. |
| ⚪ | **LOW** | Nit, micro-inconsistency, or style observation. Skip if low on time. |

**Positive / pass indicator:** 🟠 (used for strengths and status="passing" rows, NOT for severity). This matches the user's established convention.

---

## Effort estimates

Three buckets only. Don't try to be more precise than this — review-time effort estimates are noisy.

| Effort | Time | Examples |
|---|---|---|
| **Small** | <30 min | Fix a typo, add a regex to a spec, update a description, add one example row |
| **Medium** | 30 min – 2 hr | Refactor a dense section, add a fixture-based test, extract a duplicate into a single source of truth |
| **Large** | >2 hr | Add a new subcommand, change file architecture, add a new helper script with tests, write a missing reference file |

---

## Card field rules (v0.3+)

Severity colors, effort buckets, and quick-win tags appear inside each finding card as specified in `reference/output-format.md § 3. Findings`. The card's citation line carries:

```
*`<file>:<line>` · <Effort> · <✅ quick win if applicable>*
```

with severity emoji and rank in the card heading. Card sort order: severity (🔴 first) → quick-win (✅ first within the same severity) → file path (alphabetical within ties).

Strengths use 🟠 (positive indicator, not severity) and live in their own section above Findings — see `reference/output-format.md § 2. Strengths to keep` for the full format.

---

## Quick-win tag rules

A finding qualifies as a **quick win** if all three are true:

1. **Effort = Small** (<30 min)
2. **Non-architectural** — doesn't require restructuring files, renaming columns, breaking back-compat
3. **No dependencies** — doesn't require coordinating with external tools, contributors, or other skills

Common quick-win patterns:
- Adding a missing regex to a format spec
- Adding a single example row that demonstrates a documented pattern
- Rewriting a YAML-multi-line description as verb-first
- Pinning a fallback trigger condition
- Removing a dangling reference (or adding the missing target)

Non-examples (NOT quick wins even if small):
- "Rename column X to Y" — architectural, breaks comparability
- "Add machine-readable export" — small if trivial, large if it implies CI integration
- "Specify the staleness config-override mechanism" — small to write, but requires picking a format that affects future tooling

---

## Strength signals

Strengths use 🟠. List 5-10 per report under "Unusually well-done — keep doing these." Each strength gets:

- One sentence describing what's well-done
- file:line citation
- Optionally: a one-sentence note on *why* it's well-done or *how it generalizes* to other skills

Example:

> 🟠 **Verify-still-open recipe** (`format.md:73-95`) — treats row staleness as a first-class concern with a 10-second grep recipe. Generalize this to other skills: every claim about file state should have a re-verification command.

The "how it generalizes" sentence is high-value. It turns the strength from "you did this well" into "do this elsewhere too."

### Rationale, not adjectives (load-bearing)

The section heading "Unusually well-done" is a **label the report format requires**, not a verdict you assert per item. Do NOT restate it as a claim. The praise carried by each strength must be **the observation itself**, not an evaluative adjective attached to it.

Ban the "cited superlative" pattern: a real file:line citation does not license a comparative or superlative claim stacked on top of it. These are all forbidden unless you state the concrete basis:

- ❌ "one of the best calibration files I've reviewed" — implies a ranked corpus you do not have. What corpus? Ranked how?
- ❌ "impressively thorough", "exceptionally clean", "remarkably well-structured" — adjectives standing in for analysis.
- ❌ "this is unusually well-done" as a sentence — that's the section label masquerading as a finding.

Replace each with the **observed mechanism and why it works**:

- ✅ "every rule in `examples.md:11-120` has a paired bad→good example, so a reader learns the judgment, not just the format" — a checkable structural fact + the reason it matters.
- ✅ "the mode table at `SKILL.md:52-64` maps every input combination to exactly one mode with no ambiguous fallthrough" — states what you verified.

**The test before writing any strength:** strip every evaluative adjective ("best", "excellent", "unusually", "impressive", "remarkable") from the sentence. If nothing checkable remains, you have puffery, not a finding — rewrite it around the mechanism. A comparative claim ("best", "cleaner than most") is permitted ONLY if you name the comparison set in the same sentence; if you cannot, delete the comparison and keep the observation. This is the strengths-side counterpart to the "No filler" rule in `protocol.md`; the TL;DR is bound by it too.

---

## When to skip the rubric

Two exceptions where the severity rubric does NOT apply:

1. **`quick` lens reports** — under v0.3, the quick lens still produces cards but capped at 5. The rubric applies the same way; the cap is the only difference.
2. **Pure narrative findings** (e.g., "the README's voice is clear and confident") — these go in the TL;DR or the Strengths section, not the Findings cards. Strengths use 🟠 and don't carry severity/effort fields.

Otherwise every finding gets a severity color, an effort estimate, and its own card in the Findings section.

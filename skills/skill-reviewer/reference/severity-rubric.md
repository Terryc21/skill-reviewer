# Severity rubric and findings table format

Authoritative spec for how findings are labeled, scored, and tabulated in skill-reviewer reports.

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

## Findings table format

Every report includes at least one **Recommended Actions** table near the end. It uses this column shape:

| Rank | Severity | Action | File:Line | Effort | Quick win? | Why |
|---|---|---|---|---|---|---|

**Column rules:**

- **Rank** — integer, 1 is highest priority. Ties allowed.
- **Severity** — 🔴 / 🟡 / 🟢 / ⚪ with the label inline (e.g., `🟡 HIGH`).
- **Action** — imperative phrase. "Add parsing regex to format.md" not "Parsing regex is missing."
- **File:Line** — anchor for the finding. Use a stable section name when line numbers are too volatile (e.g., `format.md § Compact preset`).
- **Effort** — Small / Medium / Large.
- **Quick win?** — ✅ if Small effort AND non-architectural AND no dependencies. Blank otherwise.
- **Why** — one sentence. The reason this fix matters.

**Sorting:** primary by Severity (🔴 first), secondary by Quick win (✅ first within the same severity), tertiary by Effort (Small first).

---

## Per-file findings format

Per-file findings do NOT use a table. They use bullet lists with inline severity tags.

Example:

```
### reference/format.md

**Summary:** Defines the 10-column rating table schema, four sections, target/urgency/ROI values, detail-block structure, four presets, anti-patterns.

**Strengths**
- 🟠 Detail-block contract (lines 56-72) precisely orders closure pointer → body → verify-still-open → spawn links with worked examples
- 🟠 Verify-still-open recipe (lines 73-95) is original; treats rows as decaying artifacts

**Weaknesses**
- 🟡 Compact preset parsing is under-specified (lines 110-122) — no regex provided for the `**🔴 THIS · …**` format; downstream tools have to guess
- 🟢 Lean preset description ("drops to 6 columns") momentarily ambiguous on first read; list the kept and dropped columns explicitly
```

Strength bullets use 🟠 (positive). Weakness bullets use the severity color (🔴/🟡/🟢/⚪).

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

---

## When to skip the rubric

Two exceptions where the severity rubric does NOT apply:

1. **`quick` lens reports** — too short for a full table; bullet lists with inline severity tags suffice.
2. **Pure narrative findings** (e.g., "the README's voice is clear and confident") — these go in the TL;DR or Quality signals section, not the Recommended Actions table.

Otherwise every finding gets a severity color, an effort estimate, and a place in the actions table.

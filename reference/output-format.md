# Output format

Authoritative spec for the structure of every skill-reviewer report. Applies to all subcommands except `lenses`, `detect`, and `--version` (which have their own short outputs).

---

## Required sections (in order)

1. **TL;DR**
2. **Per-file findings** (omitted in `quick` lens and single-file skills)
3. **Cross-file findings** (omitted in `quick` lens and single-file skills)
4. **Second-opinion reconciliation** (only if `--second-opinion` was used)
5. **Quality signals** (strengths + weaknesses)
6. **Recommended actions** (the ranked table)
7. **Verification** (how to test any change)
8. **Files referenced** (flat list)

Lenses shorten or reorder sections — see `reference/lenses.md` for per-lens specs.

---

## Section specs

### 1. TL;DR

One paragraph (3-6 sentences) headline verdict, followed by a short list of 3-5 **weakness clusters** — the themes the findings group into.

Format:

```
## TL;DR

<paragraph: what kind of skill this is, the overall verdict, the standout strength, the most concerning weakness>

**Weakness clusters:**
1. <theme> — <one-sentence summary>
2. <theme> — <one-sentence summary>
...
```

If `--second-opinion` was used, the TL;DR's first line is:

> "This review includes a second-opinion reconciliation pass. See § Second-opinion reconciliation for what changed between the draft and the final version."

### 2. Per-file findings

One subsection per substantive file, in this order:
- `SKILL.md`
- `README.md` (if present)
- Files under `reference/` (alphabetical)
- Files under `examples/`
- `scripts/README.md` and scripts (grouped)
- `tests/` (grouped)
- `.claude-plugin/` manifests (grouped under one subsection)
- Other docs (`docs/`, roadmap files, etc.)

Per-subsection format:

```
### <file path>

**Summary:** <2-4 sentences>

**Strengths**
- 🟠 <observation with file:line citation>
- 🟠 <observation with file:line citation>

**Weaknesses**
- 🟡 <observation with file:line citation and severity>
- 🟢 <observation with file:line citation and severity>
```

If a file is purely boilerplate (LICENSE, .gitignore), skip the subsection. Note its presence in "Files referenced."

### 3. Cross-file findings

Categorize observations into these subsections. Skip a subsection if no findings apply.

- **Inconsistencies**
- **Missing pieces**
- **Redundancy**
- **Trigger-phrasing quality**
- **Error handling and edge cases**
- **Slash-command registration**
- **Tooling and parseability**
- **Test coverage**

Each subsection is a bulleted list with file:line citations and severity tags.

### 4. Second-opinion reconciliation (conditional)

Present only if `--second-opinion` was used. See `reference/second-opinion.md` § Step 3 for the four-part structure (Agreements, Disagreements, Refinements, Missed findings).

### 5. Quality signals

Two numbered lists.

```
### Unusually well-done — keep doing these

1. 🟠 <Item title> (`<file>:<line>`) — <description>. <Optional: how it generalizes to other skills>
2. ...

### Weak or unclear — fix these

1. 🟡 <Item title> (`<file>:<line>`) — <description>
2. ...
```

5-10 items per list. Both lists are required.

### 6. Recommended actions

The findings table specified in `reference/severity-rubric.md`. Sorted by Severity → Quick win → Effort.

Below the table, a one-sentence quick-wins summary:

> "Quick-wins total: <N> items, ~<total minutes> minutes."

### 7. Verification

How to validate any change the author makes:

- What existing tests apply (point at the harness if one exists)
- Manual test procedures for spec-only changes
- Where new tests would go if extending the test suite
- For plugin skills: how to re-test install (`/plugin install` flow)

Aim for 100-200 words.

### 8. Files referenced

Flat list of every file path the review cites. No prose. Useful for the author to verify coverage and re-locate findings.

```
- SKILL.md
- README.md
- reference/format.md
- reference/init.md
- examples/UNFORGET.md
- .claude-plugin/plugin.json
- ...
```

---

## Stylistic constraints

Apply to every report:

- **File:line for every specific claim.** If you can't cite, your finding is too vague.
- **Quote directly** when calling out specific phrasing or behavior. Don't paraphrase if the exact words matter.
- **No filler.** "This is great!" without specifics is removed.
- **No hedging.** "Might possibly be unclear" → "is unclear because X."
- **Match scope to severity.** A typo gets one line. An architectural risk gets a paragraph.
- **Imperative phrasing in actions.** "Add parsing regex" not "A parsing regex should be added."
- **Markdown formatting.** Use bold for emphasis on key terms, backticks for file paths and code identifiers, blockquotes for direct quotes from the skill being reviewed.

---

## Word budget by lens

| Lens | Target | Hard cap |
|---|---|---|
| `full` (plugin shape) | ~3500 | 5000 |
| `full` (thin-index) | ~2500 | 4000 |
| `full` (single-file) | ~1500 | 2500 |
| `discoverability` | ~1500 | 2500 |
| `safety` | ~2000 | 3000 |
| `architecture` | ~2000 | 3000 |
| `parseability` | ~1300 | 2000 |
| `tests` | ~1300 | 2000 |
| `quick` | ~800 | 1200 |

If you're about to exceed the hard cap, stop and ask whether you've drifted into a different lens or whether the skill genuinely warrants a longer review. Don't pad to hit the target.

---

## What not to include

- **Total grade or score.** No "B+ skill," no "8/10." Multi-axis findings replace single scores.
- **Generic skill-author advice.** "Consider testing" is filler. Specifics only.
- **Comparisons to other skills** unless `compare` subcommand was invoked OR a specific pattern from another skill is directly applicable.
- **Speculation about author intent.** Review what's there. If something is unclear, flag the ambiguity, don't guess what the author meant.
- **Praise of the review itself.** "I've enjoyed reviewing this skill" is filler.

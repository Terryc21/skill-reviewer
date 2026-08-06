# skill-reviewer protocol amendment: `--with-fixes` modifier and `actionable` lens

**Status:** Draft proposal, not yet submitted upstream.
**Target version:** skill-reviewer 0.2.0 (additive; non-breaking).
**Author:** Terry Nyberg, 2026-05-16.
**Origin:** Prompter review session (Stuffolio project), where the user asked for paste-ready proposed fixes alongside diagnostic findings. The session ran 39 fixes across 5 commits; producing proposed-fix text alongside each finding cut per-fix application time from ~5-15 min to ~1-3 min. The current protocol leaves the author to draft each fix from a one-sentence imperative, which is wasted effort for structural prose fixes (new sections, regex specifications, multi-line spec text).

---

## What this amendment adds

A new finding format ("Format C") that pairs each diagnostic finding with a paste-ready proposed fix. Activated via either:

- **`--with-fixes` flag** on any existing lens (e.g., `/skill-reviewer review <path> --lens=safety --with-fixes`)
- **`actionable` lens** as a shorthand for `full --with-fixes` with tighter scope (the most common use case)

The default behavior is unchanged: existing reports stay diagnostic and short. Authors who want fix-ready output opt in.

## What this amendment does NOT change

- Severity rubric (🔴 / 🔵 / 🟢 / ⚪ / 🟠) — unchanged
- Findings table (Recommended Actions) — unchanged shape; just gets richer per-row content under `--with-fixes`
- Quick-win rules — unchanged
- Strength signals — unchanged (strengths never get proposed fixes; they're observations, not directives)
- Existing lenses — unchanged behavior unless `--with-fixes` is added

---

## Proposed changes per file

### Patch 1 — `reference/severity-rubric.md`

**Insert after the existing `## Per-file findings format` section (after line 76 in the current file), as a new section:**

```markdown
---

## Proposed-fix format (opt-in via `--with-fixes`)

When the user invokes a review with `--with-fixes` (or the `actionable` lens), each HIGH and CRITICAL finding in both the per-file findings AND the Recommended Actions table gets a paste-ready proposed fix in addition to the diagnostic description.

### Severity tiering for proposed fixes

- **🔴 CRITICAL** — proposed fix REQUIRED
- **🔵 HIGH** — proposed fix REQUIRED
- **🟢 MEDIUM** — proposed fix OPTIONAL; include only when the fix is non-obvious (e.g., specific regex, specific spec text) and would save the author meaningful drafting time
- **⚪ LOW** — proposed fix OMITTED (LOW findings should be one-line imperatives; if a LOW needs paste-ready text, it's probably mis-categorized)

This tiering prevents reports from ballooning. The high-cost detail attaches only to findings where applying the fix without a draft is genuinely slow.

### Per-finding format under `--with-fixes`

Replace the simple bullet from `## Per-file findings format` with:

```
### N. <finding title>

**Severity:** <emoji + label>  ·  **File:** `<file>:<line>`  ·  **Effort:** <Small/Medium/Large>  ·  **Quick win?** <✅ or blank>

**Issue.** <2-4 sentences. What's wrong + why it matters + what the failure mode looks like in practice.>

**Proposed fix.** <Imperative one-line summary.> Apply this to `<file>:<line>`:

> <paste-ready blockquote of the actual text to add, replace, or delete.
> Multi-line when needed. Use markdown formatting that matches the target file's voice.>

<Optional 1-2 sentence rationale for why this specific phrasing was chosen, if non-obvious.>
```

### Per-finding format for MEDIUM findings under `--with-fixes`

MEDIUM findings keep the bullet shape but get an optional inline fix snippet:

```
- 🟢 <finding> (`<file>:<line>`) — <description>.
  **Fix:** <one-line imperative, OR a short code/text fragment in backticks>
```

### Recommended Actions table under `--with-fixes`

The table itself is unchanged in column shape. But each HIGH/CRITICAL row gets a **proposed-fix appendix** beneath the table referenced by the rank number:

```markdown
## Recommended actions

| Rank | Severity | Action | File:Line | Effort | Quick win? | Why |
|---|---|---|---|---|---|---|
| 1 | 🔴 CRITICAL | Fix the foo by adding the bar regex | spec.md:42 | Small | ✅ | The foo silently fails without it. |
| 2 | 🔵 HIGH | Add the missing baz handler | spec.md:88 | Medium | — | Re-invocation crashes without it. |

### Proposed fixes (HIGH/CRITICAL only)

**#1 — spec.md:42**
Replace the existing `foo: <regex>` line with:
> `foo: ^bar\\b`

**#2 — spec.md:88**
Add this subsection after `## Existing Section`:
> ### Baz handler
>
> When the user invokes baz, [...]
```

Sort proposed-fix appendix entries by Rank (same order as the table). Format is open prose, not a table, because multi-line blockquotes don't fit table cells cleanly.

### Anti-patterns under `--with-fixes`

These remain forbidden under `--with-fixes`:

- **Don't propose fixes for strengths.** Strengths are observations, not directives. Under `--with-fixes`, the "Unusually well-done — keep doing these" section is unchanged from the default — no "Proposed fix" attached.
- **Don't propose fixes when the right next step is "ask the author."** Some findings have multiple viable resolutions (e.g., a README claim that doesn't match a linked file — either rewrite the README or restructure the linked file; the author should choose). For those findings, the Proposed fix section is replaced with: `**Proposed fix.** Multiple viable resolutions — see Issue paragraph above. Author decides.`
- **Don't pre-empt voice and convention decisions.** A proposed fix that uses prose style different from the rest of the file is worse than no fix. If the reviewer is not confident about the target file's voice, drop to MEDIUM with no proposed fix.
- **Don't generate proposed fixes that haven't been internally sanity-checked.** Before emitting any proposed-fix blockquote, the reviewer verifies (a) the fix would actually apply cleanly at the cited line, (b) the fix doesn't contradict other rules elsewhere in the spec, and (c) the fix matches the target file's voice. A fix that fails any of these gets dropped, not shipped with hedging.
```

---

### Patch 2 — `reference/lenses.md`

**Insert the new lens into the table at the top of the file (between `tests` and `quick`, alphabetical placement):**

```markdown
| `actionable` | TL;DR + per-file findings (HIGH/CRITICAL only) + Recommended Actions with paste-ready proposed fixes for HIGH/CRITICAL | LOW findings, strengths (kept brief), redundancy detail | ~2500-4000 words |
```

**Insert this new lens spec section between `tests` and `quick`:**

```markdown
### `actionable`

A focused variant of `full` optimized for "I want to apply the fixes, not just understand them." Pairs every HIGH and CRITICAL finding with paste-ready proposed text.

**Apply:**
- Same coverage scope as `full` for HIGH and CRITICAL findings
- For each HIGH/CRITICAL finding, include the proposed-fix format from `reference/severity-rubric.md` § Proposed-fix format
- Recommended Actions table includes the proposed-fix appendix beneath it
- MEDIUM findings included but in bullet form only (no proposed fix unless the fix is non-obvious)
- LOW findings omitted from per-file sections (still listed in the actions table for completeness)

**Skip:**
- Strengths get the standard 1-line citation (no expanded "how it generalizes" paragraphs, since the reader is in fix-mode not study-mode)
- Per-file "Summary" paragraphs get tightened to 1 sentence
- Cross-file findings section is included but condensed (one bullet per finding, no sub-categorization)

**Output:** TL;DR + per-file HIGH/CRITICAL findings with proposed fixes + condensed cross-file section + Recommended Actions table with proposed-fix appendix + standard strengths list (1-line entries only).

**Word target:** ~2500-4000 words (higher than `full` because of the proposed-fix text, but compensated by trimming the descriptive sections).

Use when: the author has approved the diagnostic phase and wants to apply fixes immediately. Run `full` first if the author is still deciding whether to proceed; run `actionable` after the diagnostic findings are accepted.
```

**Update the decision flowchart at the bottom of `reference/lenses.md`** by adding a new bullet between "Want a 5-minute read to triage" and "Skill does destructive things":

```markdown
- **Author has approved the audit and wants paste-ready fixes** → `actionable` (or `full --with-fixes` if you also want the full diagnostic prose)
```

---

### Patch 3 — `reference/protocol.md`

**Replace the entire `## Output` section (currently lines 166-170) with:**

```markdown
## Output

See `reference/output-format.md` for the required report structure.
See `reference/severity-rubric.md` for the findings table format (severity colors, effort estimates, quick-win tags).

### Modifier flags

These flags modify the output format without changing the underlying review protocol:

- **`--with-fixes`** — add paste-ready proposed fixes to HIGH and CRITICAL findings. See `reference/severity-rubric.md` § Proposed-fix format. Adds approximately 50-80% to report length. Default off.
- **`--second-opinion`** — add a reconciliation pass with a challenger subagent. See `reference/second-opinion.md`. Default off.

Flags can be combined: `/skill-reviewer review <path> --lens=safety --with-fixes --second-opinion` produces a safety-focused review with proposed fixes and a reconciliation pass.

The `actionable` lens (see `reference/lenses.md`) is a shorthand for `full --with-fixes` with adjusted scope (HIGH/CRITICAL emphasized, LOW omitted from per-file sections).
```

**Insert a new subsection in `## Anti-patterns to avoid in your review` (after the existing "Don't make findings up" bullet):**

```markdown
- **Don't generate proposed fixes you haven't internally sanity-checked.** If `--with-fixes` is active, every proposed-fix blockquote must apply cleanly at the cited line, not contradict other rules in the spec, and match the target file's voice. A wrong-voice fix is worse than no fix — drop the finding to MEDIUM with no proposed-fix block before shipping mismatched text.
- **Don't propose fixes for findings with multiple viable resolutions.** Some findings legitimately have two or more valid paths forward (e.g., a README claim that doesn't match a linked file could be resolved by either edit). For those, the Proposed fix block reads `**Proposed fix.** Multiple viable resolutions — author decides.` Pre-drafting one path biases the author toward it.
```

**Insert a new subsection in `## Verification before finalizing` (as new questions 7 and 8):**

```markdown
7. If `--with-fixes` was active: did I verify every proposed-fix blockquote against the cited file (does the surrounding text still match what the fix anchors to)?
8. If `--with-fixes` was active: did I drop any proposed fix where I wasn't confident about the file's voice, instead of shipping a mismatched-voice fix?
```

---

### Patch 4 — `SKILL.md` (the index)

**Update the subcommand surface table** to add the new flag and lens. Replace the existing `/skill-reviewer review <path>` entry:

```markdown
| `/skill-reviewer review <path>` | Full review of one skill | `reference/protocol.md` |
| `/skill-reviewer review <path> --with-fixes` | Full review with paste-ready proposed fixes for HIGH/CRITICAL findings | `reference/severity-rubric.md` § Proposed-fix format |
| `/skill-reviewer review <path> --lens=actionable` | Apply-mode variant: HIGH/CRITICAL findings only, with proposed fixes | `reference/lenses.md` § actionable |
```

**Update the decision flowchart in `SKILL.md`** by adding between "Want a 5-minute sanity check" and "Skill performs destructive operations":

```markdown
- **Want paste-ready proposed fixes alongside the audit** → `/skill-reviewer review <path> --with-fixes` (or `--lens=actionable` for fixes-only scope)
```

---

## Worked example: what the new format looks like in practice

Below is a synthetic finding rendered three ways, so the diff between the current spec and the proposed amendment is concrete.

### Current spec (Format A — per-file findings bullet)

```
**Weaknesses**
- 🔵 HIGH — **N-prompts state model is missing** (SKILL.md:31-36). "After each rewrite, show the remaining count" presumes durable state. Where is N stored? What happens after compaction, session restart, or `/clear`? The spec is silent; in practice the counter will reset silently and the user will not notice.
```

### Current spec (Format B — Recommended Actions row)

```
| 5 | 🔵 HIGH | Document N-prompts state semantics: where N is tracked, what happens on session restart / compaction / `/clear`. | SKILL.md:31-36 | Medium | — | Silent counter drift is the predictable failure mode. |
```

### Proposed amendment (Format C — per-finding under `--with-fixes`)

```
### 5. N-prompts state model is missing

**Severity:** 🔵 HIGH  ·  **File:** `SKILL.md:31-36`  ·  **Effort:** Medium  ·  **Quick win?** —

**Issue.** "After each rewrite, show the remaining count" presumes durable state, but the spec doesn't say where N is tracked. The countdown will silently reset after context summarization, session restart, or `/clear`, and the user won't notice — they'll just see a longer-than-expected rewrite run.

**Proposed fix.** Add a "Where N is tracked" subsection under "Counting behavior (N prompts)":

> ### Where N is tracked
>
> The countdown lives in the conversation context as a single status line the LLM repeats and updates each turn. The `**Prompter:** N rewrites remaining` line is the source of truth — emit it after every rewrite so the count survives context summarization (summarizers preserve recent factual lines more reliably than mental state).
>
> If the user notices the count appears wrong after a compaction or `/clear`, treat the most recent status line in context as truth. If no status line exists (e.g., a fresh session), the countdown is considered expired and the skill stops rewriting until re-invoked.
>
> The countdown does NOT persist across separate Claude Code sessions. "Next N prompts" is single-session by design. Use "Add to CLAUDE.md" for cross-session persistence.

The "status line as source of truth" approach trades absolute reliability for compactness — a markdown line in context is more robust than an LLM's mental tally, but less robust than file-backed state. A separate operational caveat paragraph should flag this tradeoff for users running long sessions.
```

---

## Rationale: why opt-in, not default

The current spec is correctly optimized for **first-pass discovery** — short bullets, severity-tagged, file:line anchors. That's the right default because:

1. Most reviews are diagnostic ("what's wrong?"), not corrective ("apply these fixes").
2. Proposed fixes require the reviewer to be confident about the target file's voice, conventions, and surrounding context. That confidence comes from a deep read; making it the default forces every reviewer into a deeper read than the protocol budget allows.
3. Bikeshedding risk: a paste-ready proposed-fix blockquote becomes a debate target ("I'd phrase it differently"). One-line imperative actions avoid that.

But the apply-phase value of `--with-fixes` is real (the prompter session above saved ~3-5 minutes per fix across 39 fixes — roughly 2-3 hours total). The amendment makes the format opt-in so it's available when needed, invisible when not.

---

## Submission plan (if you want to upstream this)

The skill-reviewer plugin is at `~/.claude/plugins/cache/skill-reviewer/skill-reviewer/0.1.0/skills/skill-reviewer/`. The cache path implies it was installed from a remote source — likely a public repo. To submit upstream:

1. Find the source repo (probably `github.com/<author>/skill-reviewer` or similar — check `~/.claude/plugins/cache/skill-reviewer/skill-reviewer/0.1.0/.git` or the marketplace manifest)
2. Fork it
3. Apply Patches 1-4 above to a feature branch
4. Open a PR titled something like `feat: add --with-fixes modifier and actionable lens for paste-ready proposed fixes`
5. Reference this amendment doc in the PR description; reference the prompter review session as origin

Or keep as local notes and apply privately if you don't want to upstream.

---

## Open questions for future iteration

- Should `--with-fixes` work with the `compare` subcommand? Probably yes — comparing two skills with paste-ready unification fixes would be high-value. Out of scope for v1 of this amendment.
- Should the proposed-fix appendix in the Recommended Actions table be a separate section, or inline under each table row? This draft uses a separate appendix because GitHub-flavored markdown doesn't render multi-line blockquotes cleanly inside table cells. Worth re-evaluating once the format ships.
- Should there be a `--no-fixes` flag for forcing diagnostic-only output even under `actionable` lens? Probably unnecessary — if the author doesn't want fixes, they shouldn't pick `actionable`.

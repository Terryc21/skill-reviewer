# Design notes

For contributors and future-self. Explains why skill-reviewer is structured the way it is, where the design comes from, and how to extend it without breaking the core contract.

---

## Why thin-index instead of single-file

Single-file would be simpler. Everything in SKILL.md, ~400 lines, done.

We chose thin-index because:

1. **Four substantive features deserve their own spec.** Lens variants, the severity rubric, the second-opinion workflow, and detection are each 100-250 lines of careful spec. Inlining them produces a SKILL.md that's unscannable.
2. **The protocol evolves.** Every review surfaces small protocol gaps ("we should also check X"). With reference files, the relevant file edit is small. With a monolithic SKILL.md, every edit risks unrelated drift.
3. **Spec-substitution principle reuse.** The user's `unforget` skill established this architecture for similar reasons. Keeping the architecture consistent across the user's tools means the user's mental model transfers.

The trade-off: more files to maintain, more cross-references to keep healthy. The cross-file invariants section below addresses that.

---

## Why no `scripts/` directory

skill-reviewer is fundamentally LLM-driven. The review is reading + interpretation + judgment — operations no script can do well.

There's no deterministic helper to extract. (Compare to `unforget`, where surface scanning is a definite finite algorithm that benefits from Python.)

If a future feature needs determinism — for example, fuzzy-matching two review reports for diff — that helper would land in `scripts/`. Until there's a real need, skipping `scripts/` keeps the install path simpler (no Python dependency) and the design honest.

---

## Why no `tests/` directory initially

skill-reviewer output is qualitative. Golden-file testing doesn't apply: two reviews of the same skill, run on different days, will legitimately differ in word choice, ordering, and emphasis even when both are correct.

The closest things to tests we could write:

- **Format conformance** — does the report have all required sections? Are findings cited with file:line? Severity tags present?
- **Self-review** — does running skill-reviewer on skill-reviewer produce findings consistent with the spec?

Both are useful and may justify a `tests/` directory later. Until then, the validation strategy is:

1. Self-review after meaningful edits (described in README.md and SKILL.md)
2. Sample report comparison (`skills/skill-reviewer/examples/sample-report-unforget.md` is the canonical reference)
3. Manual review of a new lens by running it against `unforget` and confirming the output matches the lens's stated focus and skip lists

### Self-review acceptance criteria

A self-review (running `/skill-reviewer review` on skill-reviewer itself) is **healthy** if all of:

1. Every CRITICAL and HIGH finding it surfaces is **already named** in `## Open design questions` below, OR is a **drift-class fix** — an edit that touches ≤3 files, adds no new directory, removes no existing surface, and breaks no public contract. Drift-class shapes include: a dangling reference, a subcommand-table mismatch, a section that exists in one file but is referenced from another, a cross-file citation that has gone stale, or a description string that has drifted between manifest copies. Anything wider (new subcommand, new directory, schema change, surface removal) does NOT qualify and must be moved into `## Open design questions` with a target version before release.
2. MEDIUM and LOW findings can be deferred; surfacing them is informational, not blocking.
3. The reviewer can fill in `## Files referenced` cleanly — every file cited in the report exists at the cited path.

A self-review is **alarming** (blocks release until resolved) if:

- A CRITICAL or HIGH finding describes a gap NOT named in `## Open design questions` AND NOT fixable in one commit, OR
- The reviewer cannot run the full protocol because a required reference file is missing or contradicts itself, OR
- The reviewer surfaces a prompt-injection attempt against itself (extreme edge case; would mean a contributor planted text directed at future reviewers).

When alarmed: triage. Either move the surprising finding into `## Open design questions` with a v0.X target and ship anyway, or fix the underlying issue and re-run the self-review.

The self-review is reviewer-dependent — two different LLM sessions will produce different reports. The acceptance criterion is therefore about **finding categories, not exact finding lists**.

---

## How to add a new lens

1. **Identify the concern.** Is it covered by an existing lens? Don't duplicate — expand instead.
2. **Write a focus list.** Which sections of `protocol.md` does this lens apply?
3. **Write a skip list.** Which sections should this lens explicitly skip?
4. **Set a target report length range** (in `skills/skill-reviewer/reference/lenses.md`'s table at the top).
5. **Add a spec subsection to `skills/skill-reviewer/reference/lenses.md`** with the focus list, skip list, and a "Use when" guideline.
6. **Update `skills/skill-reviewer/reference/output-format.md`** if the new lens needs a non-standard report structure (most won't).
7. **Update the lens table in `README.md` and `SKILL.md`** to keep the user-facing surface in sync.
8. **Run the new lens against a real skill** to verify the output makes sense.

**Anti-pattern:** adding a lens that's "the full review but shorter." That's what `quick` is for.

**Anti-pattern:** adding a lens that requires extending the protocol with new checks. Add the checks to `protocol.md` first, then write the lens to apply them.

---

## Cross-file invariants

The thin-index architecture only works if these invariants hold:

1. **`plugin.json` and `marketplace.json` descriptions match exactly.** (Lesson from the `unforget` review.) Both files should have the same activation-phrase guidance. Use the same description block for both.
2. **SKILL.md frontmatter `description` is verb-first.** Starts with "Use when…" so Claude Code's router has a clear hook. Trigger phrases go in the first sentence, not buried in YAML multi-line.
3. **Every reference file is independently readable.** A reader who jumps to `skills/skill-reviewer/reference/lenses.md` without reading SKILL.md first should still understand what a lens is.
4. **Cross-file references use stable anchors.** Prefer `skills/skill-reviewer/reference/severity-rubric.md § Effort definitions` over `skills/skill-reviewer/reference/severity-rubric.md:42`. Line numbers drift; section names don't.
5. **Examples conform to the spec.** `skills/skill-reviewer/examples/sample-report-unforget.md` uses the severity rubric and the output format exactly as specified. If you change the spec, update the example.
6. **No duplicate sources of truth.** If a definition appears in two reference files, one is canonical and the other refers to it. Don't restate.
7. **The section name `## Open design questions` in this file is load-bearing.** `SKILL.md § Compatibility notes` and `README.md § Self-review` cite it by name as the canonical known-gaps list referenced by the self-review acceptance criteria. If you rename this section, search the repo for `Open design questions` and update the citing files. Same rule for the section name `## Decided design questions` introduced for settled items.
8. **Every entry in `## Open design questions` must carry a target.** Either a specific version (`Target: v0.4+`) or an explicit `Target: untargeted; reopen when demand surfaces`. The self-review acceptance criteria require known-gaps entries to have a target version; the list must follow the rule it enforces.

When in doubt: run `/skill-reviewer review` on this repo and act on the findings.

---

## Anti-patterns (things contributors should not propose)

- **Auto-fix or auto-edit features.** The skill produces reports; the author decides what to act on. Adding auto-fix would make the skill less trustworthy and create blast-radius questions we don't want to answer.
- **Single-number grading.** "B+ skill" or "8/10" hides the dimensions. Multi-axis findings are the point of this tool.
- **Templates for "best skill" comparison.** Skills have legitimate stylistic variation. A template imposes uniformity that's worse than the variation it removes.
- **Chains of three or more reviewer passes.** `--second-opinion` is one challenger. A third reviewer doesn't add proportional value; escalate disagreements to the author.
- **Lens variants that need their own protocol.** Lenses are subsets of one shared protocol. A lens that needs new check types belongs as protocol additions plus a lens.
- **Output formats that aren't markdown.** JSON/YAML output could be a future feature, but the canonical output is markdown. Don't propose replacing markdown with something else.

---

## Open design questions

These haven't been settled. If you have opinions, file an issue or propose a PR. Every entry carries a target version (or an explicit `untargeted` marker) per cross-file invariant #8.

1. **`compare <path1> <path2>` deferred from v0.2 (removed from surface).** The subcommand was listed in v0.1 but never spec'd — no `protocol.md § Compare mode` section ever existed. Removed in v0.2 pending real demand. Open shape questions if it returns: one merged report vs two parallel + synthesis; how to handle stylistic variations that aren't bugs; whether the lens variants apply per-skill or to the comparison as a whole. Under the card format, this would render cleanly — one set of cards per skill, plus a Patterns section synthesizing cross-skill observations. **Target: v0.4+ (deferred pending real demand).**
2. **Should `detect` output be machine-readable (JSON) in addition to human-readable markdown?** Useful for CI integration but adds a parseability commitment. **Target: untargeted; reopen when CI integration demand surfaces.**
3. **Should there be a `--include` / `--exclude` flag to scope the review to specific files?** Useful when a skill is huge but you only want to audit one subsystem. Adds complexity to the protocol. **Target: untargeted; reopen if a multi-subsystem skill emerges that warrants partial review.**
4. **Should reports include a recommended next-review date based on how much the skill has changed?** Useful for ongoing maintenance audits, but requires tracking review history somewhere. **Target: untargeted; reopen if a review-history persistence layer is added.**

---

## Decided design questions

Settled with rationale and reopen criteria. Tracked here for transparency and to prevent rehashing.

1. **[DECIDED, v0.2] Second-opinion subagent type.** Decision: **Plan-type only**, per `skills/skill-reviewer/reference/second-opinion.md:36`. Considered alternatives: Explore-type (read-only, faster), general-purpose (more flexible). Plan-type chosen because it can read files independently AND produce structured challenge reports, which Explore alone cannot. **Reopen criteria:** evidence that Plan-type produces consistently miscalibrated challenger reports for specific skill shapes (e.g., single-file skills where the protocol is overkill).

2. **[DECIDED, v0.3.1] Subcommand registration vs activation-with-arg-parsing.** Decision: **activation-with-arg-parsing**, no `commands/` directory. The plugin registers `/skill-reviewer` once; Claude parses the argument string as a subcommand. Considered alternative: add one `commands/<subcommand>.md` file per documented subcommand so each becomes a native slash command with autocomplete. Activation-parsing chosen because (a) the subcommands are short and the argument shape is stable, (b) per-subcommand command files would duplicate spec already in `reference/`, (c) the plugin schema's `commands` surface is still maturing and adding files now risks drift if the schema changes. **Reopen criteria:** users report routing failures where Claude doesn't recognize a subcommand string, OR the plugin schema gains tab-completion semantics that materially improve UX, OR a subcommand surface stops being shaped like one CLI verb + one path argument.

---

## Origin

skill-reviewer was extracted from a single Claude Code session in May 2026:

1. User asked Claude to review the `unforget` skill (a sibling project)
2. Claude produced a detailed review (preserved as `skills/skill-reviewer/examples/sample-report-unforget.md`)
3. User asked for a reusable prompt to run the same analysis on any skill
4. Claude wrote a self-contained prompt
5. User asked to turn the prompt into a skill
6. Plan was made, approved, executed

The architecture choices (thin-index, four feature modules, severity rubric matching the user's conventions) were chosen during the planning phase before any file was written. The protocol itself is largely the prompt from step 4, restructured into reference files.

The skill is therefore a literal artifact of using Claude Code well: each step generated something useful, and the final step turned the implicit knowledge into a reusable tool.

---

## Release history

- **v0.3.0 (2026-05-17)** — Card-format output. Replaced the v0.2 8-section structure (TL;DR / Per-file findings / Cross-file findings / Quality signals / Recommended actions / Verification / Files referenced) with a 4-section card structure (TL;DR / Strengths / Findings / Patterns). Motivation: in practice the v0.2 format mentioned each finding 3-5 times across overlapping sections, forcing readers to reconcile multiple views to act. Cards mention each finding exactly once. Word budgets shrank ~50%. See `reference/output-format.md § What changed in v0.3`.
- **v0.2.1 (2026-05)** — Three onboarding-polish gaps from the v0.2.0 self-review.
- **v0.2.0 (2026-05)** — `--second-opinion` flag, additional lens variants, refined severity rubric.
- **v0.1 (2026-05)** — Initial release extracted from the unforget review session (see Origin above).

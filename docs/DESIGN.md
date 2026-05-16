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

These haven't been settled. If you have opinions, file an issue or propose a PR.

1. **Should `compare <path1> <path2>` produce one merged report or two parallel reports with a synthesis section?** Currently the spec implies one merged report. The alternative is cleaner for cross-skill diff use cases.
2. **Should `detect` output be machine-readable (JSON) in addition to human-readable markdown?** Useful for CI integration but adds a parseability commitment.
3. **Should there be a `--include` / `--exclude` flag to scope the review to specific files?** Useful when a skill is huge but you only want to audit one subsystem. Adds complexity to the protocol.
4. **Should the second-opinion subagent be configurable (Plan vs Explore vs general-purpose)?** Currently spec'd as Plan. Other subagent types might produce different challenger styles.
5. **Should reports include a recommended next-review date based on how much the skill has changed?** Useful for ongoing maintenance audits, but requires tracking review history somewhere.

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

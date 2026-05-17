# Sample report: review of `bug-echo` (v1.1.1)

This is a simplified canonical example of skill-reviewer output. It reviews the `bug-echo` skill (a sibling repo by the same author) using the `full` lens. The skill was classified as **mixed-shape** (plugin manifests with no `reference/` directory; per `reference/detection.md § Mixed / edge cases`, classified as plugin with a structural note).

Read this alongside `sample-report-unforget.md` to see two contrasts in one set of canonical examples:

- **unforget** is plugin-shape (thin-index, full `reference/` tree) reviewed at 1,841 words and 13 findings — what a thorough review of a fully-elaborated skill looks like.
- **bug-echo** is mixed-shape (single-file SKILL.md with plugin manifests) reviewed at ~900 words and 8 findings — what a focused review of a smaller skill looks like.

> **Format note:** This report uses the v0.3+ card format. The shorter shape isn't a different lens; it's the same `full` lens applied to a skill with less surface to review.

---

## TL;DR

`bug-echo` is a single-file Claude Code skill (plugin-shape due to `.claude-plugin/` manifests, but with no `reference/` directory) that scans a codebase for sibling instances of a bug after a fix lands. The skill's **distinctive design idea is genuinely strong**: pattern self-validation against the pre-fix file before scanning, which prevents the "scan with a bad pattern" failure mode that pattern-matching tools usually inherit silently. The standout weakness is **classification taxonomy drift**: the README opens with "BUG / OK / REVIEW" (3-class) but the SKILL.md, examples, and report format use BUG / WATCH / OK / REVIEW (4-class). A first-time reader hits the contradiction within the first 20 lines.

**Weakness clusters:**
1. **Classification-taxonomy drift** — README top says 3-class, SKILL.md and examples use 4-class
2. **Version-string drift** — "Deferred to v1.1+" section is in a v1.1.1 release
3. **Single-file shape carrying plugin-shape weight** — 384-line SKILL.md with no `reference/` offload

---

## Strengths to keep

- 🟠 **Pattern self-validation against pre-fix file** (`SKILL.md:125-142`) — the inferred regex must match the pre-fix file or the skill aborts. Prevents the "scanning with a bad pattern produces nonsense findings" failure mode that catalog-based linters can't avoid. *Generalizes:* any tool that constructs its own query should validate the query against known-positive input before running.
- 🟠 **"Surface → verify → generalize" workflow position** (`SKILL.md:22-28`, `README.md:207-219`) — bug-echo is explicitly the third stage, with unforget and radar-suite as the prior two. *Generalizes:* skills compose better when each one names the stage it occupies rather than claiming to be a universal entry point.
- 🟠 **Rename detection in self-validation** (`SKILL.md:131`) — handles `git log --follow --name-status -1` for renamed files. *Generalizes:* git-aware tools that consult HEAD~1 must handle renames or they silently fail on real diffs.
- 🟠 **Linter complementarity framed honestly** (`README.md:22-39`) — "linters catch what bug-echo would never run for; bug-echo catches what your linter has no rule for." Resists positioning as a replacement. *Generalizes:* tools that overlap with established ones should name the overlap, not paper over it.
- 🟠 **Commit-message back-references report** (`SKILL.md:303-307`) — `bug-echo: applied N fixes from <slug> report` makes the commit cite the report file path. *Generalizes:* automation-generated commits should cite the artifact they came from so future archaeology works.
- 🟠 **Honest-limits section names what it can't catch** (`README.md:196-205`) — cross-context mutations, race conditions, high-false-positive patterns, fix-without-shape. *Generalizes:* tools that name their specific limits are trusted more than tools that don't.

---

## Findings (5)

### 🟡 1. Classification taxonomy drifts between README opening and SKILL.md

**Why:** `README.md:5` and `README.md:15` both describe a 3-class taxonomy ("BUG / OK / REVIEW"). `SKILL.md:178-182` defines a 4-class taxonomy (BUG / WATCH / OK / REVIEW), `README.md:162` later confirms 4 classes, and the canonical example is dominated by 3 WATCH findings. A new reader reads "three classes," sees WATCH findings in the example, and concludes the docs are out of date.

**Fix:** rewrite `README.md:5` and `README.md:15` to "classified BUG / WATCH / OK / REVIEW." WATCH was added in commit `57a53ea` ("feat: formalize WATCH class"); the README opening wasn't updated to match.

*`README.md:5,15` · Small · ✅ quick win*

### 🟡 2. "Deferred to v1.1+" section is in v1.1.1

**Why:** `SKILL.md:377` is titled "## Deferred to v1.1+" and lists JSON sidecar, recurrence detection, and other future work. The current version is 1.1.1 (`plugin.json:3`, `marketplace.json:11`). "v1.1+" reads as "v1.1 and later," but v1.1 already shipped, then v1.1.1. A reader checking on the JSON sidecar will read "v1.1+" and reasonably expect it to be in v1.1. `README.md:238` repeats the confusion ("Planned for v1.1").

**Fix:** rename `SKILL.md:377` to "## Deferred to v1.2+" (or "## Deferred to a future release"). Update the four in-line cross-references at `SKILL.md:64,85,176,275`. Rewrite `README.md:238` to match.

*`SKILL.md:64,85,176,275,377` · `README.md:238` · Small · ✅ quick win*

### 🟡 3. plugin.json and marketplace.json descriptions diverge

**Why:** `plugin.json:4` description is 89 words; `marketplace.json:13` description is 38 words. They're not just different lengths — they describe the skill differently (one leads with "find and rate," the other with "finds and rates the same anti-pattern"). Claude Code's router and the marketplace listing may read different copies; divergence creates a discoverability mismatch. Previous commit `1b846ff` was specifically titled "fix: unify version to 1.1.0 across manifests + README; converge description strings" — convergence has drifted again since.

**Fix:** pick one canonical description and use it byte-for-byte in both files. Run `diff <(jq -r .description plugin.json) <(jq -r .plugins[0].description marketplace.json)` after the edit; should output nothing.

*`.claude-plugin/plugin.json:4` · `.claude-plugin/marketplace.json:13` · Small · ✅ quick win*

### 🟢 4. Single-file SKILL.md doing reference/ work without saying so

**Why:** SKILL.md is 384 lines. That's longer than skill-reviewer's SKILL.md (118 lines, fat `reference/` underneath). The length comes from carrying multiple separable specs: the 6-step protocol, the WATCH definition, the rating-table format, the metadata-keys explanation, and the bug-prospector handoff are all here. No decision is documented anywhere about why bug-echo stayed single-file when its sibling skills moved to thin-index.

**Fix:** either accept the single-file shape and document the choice (add a "Why single-file" section explaining the deliberate decision), OR extract `reference/protocol.md`, `reference/classification.md`, and `reference/metadata-keys.md`. The first option is cheaper if there's a real reason; the second is the path skill-reviewer and unforget already took.

*`SKILL.md` (whole file) · Medium*

### 🟢 5. Deferred-feature list duplicated across SKILL.md and README

**Why:** `SKILL.md:377-385` lists 4 deferred features. `README.md:238` lists 4 deferred features. The lists *almost* match but the README adds "built-in catalog mode" and SKILL.md adds "multi-language pattern construction." Two hand-maintained lists of the same artifact, currently disagreeing on one item each.

**Fix:** make one canonical and have the other reference it. Easiest: keep the README list, replace `SKILL.md:377-385` with "See `README.md § Status` for the deferred-feature list." Reconcile the disagreement while you're there.

*`SKILL.md:377-385` · `README.md:238` · Small · ✅ quick win*

---

## Patterns across findings

- **Recently-added features didn't update the older docs.** Findings #1 (WATCH not in README opening) and #5 (deferred-feature list drift) share the same pattern: a feature lands in one place, the matching edit doesn't happen elsewhere. A pre-commit grep for the old taxonomy (`BUG / OK / REVIEW`) would have caught the WATCH gap on the commit that introduced it.
- **Version-string drift is the dominant maintainability risk.** Findings #2, #3, and #5 all involve version-tracking drift. The skill ships v1.1.1; the docs still call deferred features "v1.1+". A pre-release ritual that does `git grep "v1.[0-9]" -- '*.md'` and reconciles every hit would close this whole class of issue at once.
- **Four of five findings are quick wins.** Only #4 (single-file vs thin-index) is architectural, and even that has an "accept and document" path that's a small edit. A single afternoon of focused spec editing closes 4 of 5 findings.

---

## Next step

**Next step:** batch #1, #2, #3, and #5 into a single "v1.1.2 polish" commit — all four are quick wins targeting documentation drift. Then triage #4 (single-file vs thin-index) in its own session: either write the "Why single-file" rationale or extract the reference files. Re-run skill-reviewer after the wave; the finding count should drop to 1.

*Severity legend: 🔴 fix before publishing · 🟡 fix before next release · 🟢 polish · ⚪ skip · 🟠 strength*

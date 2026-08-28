# Sample report: review of `unforget` (v0.2.0)

This is a canonical example of skill-reviewer output. It reviews the `unforget` skill (a sibling repo by the same author) using the `full` lens. The skill was classified as **plugin** shape, so the target length is ~1500 words under the card format.

Read this to see how cards work in practice. Your own reports won't be identical — the findings depend on the skill — but the structure should match.

> **Word count: 1841.** This is 22% over the plugin-shape target (~1500) but well under the hard cap (2500, per `reference/output-format.md § Word budget by lens`). The over-target reflects this report's role as a canonical reference — it shows the full shape of a thorough review with 13 findings. Real reviews should aim for the target unless the audit genuinely justifies more; a 1500-word report with 8-10 findings is more representative of a typical run.

> **Format note:** This report uses the card format. It was originally produced under the earlier 8-section structure and re-rendered when the format changed; findings, strengths, and TL;DR are unchanged.

---

## TL;DR

`unforget` is a Claude Code skill that consolidates deferred work (paused plans, audit findings, observed bugs) into one structured UNFORGET.md per project. It's more disciplined than most skills I've reviewed: thin SKILL.md → fat reference files, deterministic Python helpers with prose fallbacks, backups before destructive operations, and the **verify-still-open recipe** is a genuinely original idea worth generalizing. The standout strength is structural; the standout weakness is that the canonical example never demonstrates the project's signature feature.

**Weakness clusters:**
1. **Tooling and parseability gaps** — Compact preset has no canonical parsing regex; downstream tools have to guess
2. **Discoverability** — natural-language trigger phrases buried in YAML-multi-line; description should be verb-first
3. **Test coverage** — Surface 6 (memory files) and `prune_backups.py` are explicitly untested
4. **Duplicate sources of truth** — conservative defaults and closure pointer format live in 2-3 places that can drift
5. **Under-specified fallbacks** — "fail-soft" and Python-missing trigger conditions are vague

---

## Strengths to keep

- 🟣 **Verify-still-open recipe** (`format.md:73-95`) — treats row staleness as a first-class concern with a 10-second grep recipe before acting on a row. *Generalizes:* every claim about file state should have a re-verification command attached.
- 🟣 **Conservative-defaults contract** (`init.md:139-152`) — pinned identical defaults across implementations removes a class of "Python and prose-fallback disagreed" bugs. *Generalizes:* when multiple implementations of the same algorithm exist, pin their tunable values in one place both consume.
- 🟣 **Post-fix-sweep workflow** (`promotion.md:7-50`) — three-skill chain (surface → verify → generalize) with a worked example and a 60-90 minute time estimate. *Generalizes:* compose skills explicitly with role-named stages rather than treating "use these together" as implicit.
- 🟣 **Spec-substitution principle** (`SKILL.md:101`) — named architectural rule: SKILL.md is the index, reference files are authoritative, read them before acting. *Generalizes:* thin-index architectures rot without an explicit anti-monolith rule.
- 🟣 **Empty-case reframing** (`init.md:101-105`) — reframes "scan found zero results" as a successful run, not a failure. Most scanners default to "did I do enough?" anxiety; flipping the framing is a small UX win.
- 🟣 **Honest maturity callout** (`README.md:64-71`) — "works for one project shape (Stuffolio), needs feedback from others" builds trust without underselling. *Generalizes:* in skill READMEs, naming the shape you've actually validated beats claiming universal applicability.
- 🟣 **Format-version contract with read-only fallback** (`SKILL.md:138-146`) — handles three cases (absent / supported / future) with read-only fallback for unknown future versions. *Generalizes:* every persistent-format skill needs this; forward-compat without it is a future-self trap.
- 🟣 **Backup rotation** (`promotion.md:85-101`) — 5-deep retention, lexicographically sortable filenames, `.gitignore` recommendation. *Generalizes:* destructive operations need a recovery shape, not just a recovery procedure.
- 🟣 **Decision flowchart** (`SKILL.md:74-84`) — "which subcommand do I run?" table that answers the question before the user asks. *Generalizes:* every multi-subcommand skill should have one.
- 🟣 **Single source of truth for `AUDIT_FILENAME_RE`** (`scan_surfaces.py:94`) — consolidated regex used by both Surface 2 and Surface 1b. *Generalizes:* when two callers share a regex, extract it; drift is silent and expensive.

---

## Findings (13)

### 🟡 1. Canonical example doesn't demonstrate verify-still-open recipe
**Why:** The project's most innovative idea (`format.md:73-95`) is invisible in the only fully-populated example. A new user reading `examples/UNFORGET.md` before the spec never learns the pattern exists.
**Fix:** add the verify-still-open recipe to at least 2 rows in `examples/UNFORGET.md`. Show what a "verified-open" closure pointer looks like next to a "verified-closed" one.
*`examples/UNFORGET.md` · Small · ✅ quick win*

### 🟡 2. Compact preset parsing regex missing
**Why:** `format.md § Compact preset` specifies the visual format `**🔴 THIS · …**` but provides no regex. Downstream tools have to guess at U+00B7 middle dot vs ASCII hyphen, bold wrapping, whitespace tolerance. The format isn't actually parseable as written.
**Fix:** add a canonical regex to `format.md:110-122` with at least three valid examples and three invalid examples that should not match.
*`format.md:110-122` · Small · ✅ quick win*

### 🟡 3. SKILL.md description buried in YAML-multi-line
**Why:** Frontmatter description starts with "A single source of truth..." with trigger phrases ("what's deferred?", "what's the backlog?") buried mid-paragraph. Claude Code's router likely under-matches natural-language activations.
**Fix:** rewrite verb-first per Anthropic's pattern. `description: |\n  Use when the user asks "what's deferred?", "what's the backlog?"…`
*`SKILL.md:4-9` · Small · ✅ quick win*

### 🟡 4. Python-fallback trigger conditions undefined
**Why:** Phase 2 of init says "If Python is unavailable, fall back to prose." But "unavailable" isn't specified — missing binary? non-zero exit? invalid JSON output? All three should trigger fallback, but a reader can't tell which.
**Fix:** spell out the three trigger cases explicitly. Add a parenthetical: "(missing `python3` binary on PATH, non-zero exit code, or unparseable JSON output)."
*`init.md:64` · Small · ✅ quick win*

### 🟡 5. Staleness-threshold config-override referenced but unspecified
**Why:** `commands.md:222-231` describes per-priority staleness thresholds and says they're "config-overridable" (line 233). The config file format, location, and override syntax are never documented. Users can't configure what the spec implies they can configure.
**Fix:** either specify the override mechanism (file path + format + which thresholds it overrides) OR remove the "config-overridable" claim. Don't ship a dangling reference.
*`commands.md:233` · Small · ✅ quick win*

### 🟡 6. Conservative defaults duplicated in two reference files
**Why:** `init.md:139-152` and `commands.md` `/unforget add` (lines 29-36) both define the same conservative-defaults table. Currently agree. Will drift the first time someone edits one without the other.
**Fix:** extract to a single subsection in `format.md` (`format.md § Conservative defaults`). Both `init.md` and `commands.md` reference that subsection rather than re-stating.
*`format.md`, `init.md:139-152`, `commands.md:29-36` · Small · ✅ quick win*

### 🟡 7. Surface 6 (memory files) untested in CI
**Why:** Memory-file scanning is non-deterministic across users (each user's memory dir contents differ). The test harness has no fixture for it, so the surface is silently untested. Regressions land without anyone noticing.
**Fix:** add a `--memory-dir` override flag to `scan_surfaces.py`. Point it at a fixture directory in `tests/`. Unsuppress the Surface 6 test path. Medium effort because it touches both script and harness.
*`scripts/scan_surfaces.py`, `tests/run.sh` · Medium*

### 🟡 8. `prune_backups.py` untested
**Why:** Only destructive helper in the skill. Currently uncovered. Destructive code without tests is the classic root-cause pattern for "I lost my work" incidents. The `--bless` mode in the harness can't catch this because there's no fixture yet.
**Fix:** add a fixture-based test for `prune_backups.py`. Construct a `tests/fixtures/backups-deep/` directory with N+1 backups; run the prune; assert exactly N remain and they're the most recent. Add a destructive-mode case (force-delete-all) gated behind an explicit flag.
*`tests/`, new fixture under `tests/fixtures/backups-deep/` · Medium*

### 🟡 9. "Fail-soft" undefined operationally
**Why:** `promotion.md:109-115` says the cross-skill version-pinning behavior is "fail-soft" but doesn't define what that means. Suppress the recommendation? Show it with a warning? Refuse and exit? Each is a different UX.
**Fix:** pick one and document it. Recommended: show the recommendation with a "Version mismatch detected; proceed with caution" prefix, allow the user to proceed.
*`promotion.md:109-115` · Small · ✅ quick win*

### 🟡 10. plugin.json and marketplace.json descriptions diverge
**Why:** marketplace.json has the fuller description; plugin.json has a truncated version. Claude Code's router may read either depending on context — divergence creates a discoverability gap.
**Fix:** copy marketplace.json's description into plugin.json (or extract both from a single source file). Verify they match byte-for-byte after the edit.
*`.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` · Small · ✅ quick win*

### 🟢 11. Closure-recommendation prose is a wall of text
**Why:** `commands.md:77-138` describes five interacting mechanics (cross-promo gate, 90-day cooldown, HTML marker placement, reset conditions, detect-then-recommend) in 61 lines of dense prose. Reading order is unclear. New contributors will misunderstand at least one of the five.
**Fix:** break into 5 subsections, one per mechanic. Add a worked example at the end showing all five interacting in a real closure recommendation.
*`commands.md:77-138` · Medium*

### 🟢 12. `/unforget repair` command implied but missing
**Why:** README has a recovery section with detailed scenarios (corrupted file / lost backup / partial write). Readers infer a `/unforget repair` command exists. There is no such command. The README's recovery section is implicit promise without implementation.
**Fix:** either add `/unforget repair` per the README's recovery scenarios (Large effort), OR move the recovery section out of README into a `docs/RECOVERY.md` that explicitly says "manual recovery steps, no automated command yet."
*`commands.md` (new section) OR `README.md` + `docs/RECOVERY.md` · Large*

### 🟢 13. No machine-readable export
**Why:** `/unforget list` produces markdown only. Downstream tools (dashboards, CI integrations, cross-project rollups) can't consume the output without parsing markdown. The 10-column rating table is structurally a CSV but ships only as prose.
**Fix:** add `--json` and `--csv` flags to `/unforget list`. Schema in `format.md`. Reuse the existing parsing logic from the canonical regex (#2).
*`commands.md` § /unforget list, `scripts/list.py` (new) · Medium*

---

## Patterns across findings

- **Drift between paired sources of truth is the dominant maintainability risk.** Findings #6 (conservative defaults) and #10 (manifest descriptions) are symptoms of the same gap: when the same information lives in two files maintained by hand, drift is inevitable and silent. Either extract to a single canonical source and reference it, or wire up a build-time check that errors on divergence.
- **The canonical example under-demonstrates the canonical features.** Findings #1 (verify-still-open absent) and #13 (no machine-readable export but a CSV-shaped table) both point at the gap between what the spec says the skill does well and what the example shows it doing well. The example is the spec's first reader; it should exercise every signature feature.
- **Five of the eight HIGH findings are documentation-only.** Quick-wins (#1-#6, #9, #10) are all small spec edits. Two are non-trivial (#7, #8: test infrastructure). One is large (#12: repair command vs docs-only recovery). A single afternoon of focused spec editing closes more than half the HIGH list.

---

## Next step

**Next step:** start with #1 (canonical example demonstrates verify-still-open) and #3 (verb-first description). #1 makes the project's signature feature visible to new users; #3 likely improves routing match rate. Then batch #2 + #4 + #5 + #6 + #9 + #10 into a single "specs polish" commit. The two test-infrastructure findings (#7, #8) and the repair-or-docs decision (#12) deserve their own session. Re-run skill-reviewer after the wave, watch the finding count drop. That's the loop.

*Severity legend: 🔴 fix before publishing · 🟡 fix before next release · 🟢 polish · ⚪ skip · 🟣 strength*

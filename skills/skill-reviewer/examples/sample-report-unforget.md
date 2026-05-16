# Sample report: review of `unforget` (v0.2.0)

This is a canonical example of skill-reviewer output. It reviews the `unforget` skill (a sibling repo by the same author) using the `full` lens. The skill was classified as **plugin** shape, so target length is ~3500 words.

Read this to see how the format works in practice. Your own reports won't be identical — the findings depend on the skill — but the structure should match.

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

## Per-file findings

### `SKILL.md`

**Summary:** Thin index (165 lines). Describes the problem, lists 8 subcommands, points at `reference/*.md` for full specs. Includes the "spec-substitution principle" as a named architectural rule.

**Strengths**
- 🟠 Decision flowchart (lines 74-84) is a model "which subcommand do I run?" table. New users get to action fast.
- 🟠 Format-version contract (lines 138-146) handles three cases (absent / supported / future) with read-only fallback for future versions.
- 🟠 "Spec-substitution principle" (line 101) is named and justified — explicit architectural rules don't drift.

**Weaknesses**
- 🟡 Frontmatter description (lines 4-9) is YAML-multi-line; trigger phrases ("what's deferred?", "what's the backlog?") are buried mid-paragraph instead of being verb-first ("Use when…")
- 🟢 v0.1 vs v0.2 install path coexistence (line 15) — if both are installed, which wins? Not specified.
- 🟢 Companion files table (lines 88-101) doesn't list `scripts/` even though script delegation is core to the architecture.

### `README.md`

**Summary:** 314 lines. TL;DR, install (two-step with v0.1 fallback), quick start, command reference, preset table, recovery instructions. Well-written, honest about maturity.

**Strengths**
- 🟠 "Maturity" section (lines 64-71) candidly acknowledges the skill works for one project shape (Stuffolio) and needs feedback from others. Trust-building.
- 🟠 "See it first" excerpt (lines 73-91) shows the format before asking the reader to commit.
- 🟠 "Why Target is its own column" (lines 131-135) defends a non-obvious design decision with concrete reasoning.
- 🟠 Recovery table (lines 264-287) anticipates real failure modes users might cause.

**Weaknesses**
- 🟢 "AI memory files: memory is for context, UNFORGET.md is for tracking. Don't duplicate." (line 259) is a great rule — should also live in SKILL.md since future LLM sessions read SKILL.md first.

### `reference/format.md`

**Summary:** 154 lines. Defines 10-column table schema, detail-block contract (closure pointer → body → verify-still-open → spawn links), four presets, anti-patterns.

**Strengths**
- 🟠 Detail-block contract precisely ordered with worked examples (lines 56-72)
- 🟠 Verify-still-open recipe (lines 73-95) is the standout idea in the whole skill — treats rows as decaying artifacts, gives the LLM a 10-second re-validation check.

**Weaknesses**
- 🟡 Compact preset parsing is under-specified (lines 110-122). No regex for the `**🔴 THIS · …**` format. Downstream tooling has to guess at U+00B7 middle dot vs ASCII hyphen, bold wrapping, etc.
- 🟢 Closure narrative order ("Lead the body with **CLOSED YYYY-MM-DD…**", line 54) is clear in prose but the examples never show a multi-paragraph closure.
- 🟢 Lean preset description ("drops to 6 columns") momentarily ambiguous — list kept and dropped columns explicitly.

### `reference/init.md`

**Summary:** 285 lines. Seven-phase init flow: setup questions → surface survey → triage → defaults → user-add → optional deep-dump → preview and write.

**Strengths**
- 🟠 Re-run abort (lines 15-23) is explicitly guarded; routes to `/unforget import` instead of overwriting.
- 🟠 Conservative-defaults contract (lines 139-152) — pinned identical defaults across implementations.
- 🟠 Empty-case branch (lines 101-105) reframes zero-result scan as success.

**Weaknesses**
- 🟡 Phase 2 fallback trigger ambiguous (line 64) — "If Python is unavailable" doesn't specify: missing binary? non-zero exit? invalid JSON? All three should trigger.
- 🟢 Phase 7 archive behavior silent on Deferred/Skipped rows (line 273) — spec covers Fixed/RESOLVED only.
- 🟢 No documented error handling for mid-flow cancellation or Phase 7 write failure.

### `reference/commands.md`

**Summary:** 301 lines. Per-subcommand specs for `add`, `edit`, `import`, `list`, `scan`, `--version`. Each has usage, steps, behavior details.

**Strengths**
- 🟠 `/unforget add` speed contract (lines 49-51): "30 seconds or less … the skill has failed at its core promise" — protects a friction-point tool from feature creep.
- 🟠 Terminal-aware rendering for `/unforget list` (lines 197-214) auto-falls-back to 6-column projection at <120 cols; on-disk format unchanged.

**Weaknesses**
- 🟡 Closure recommendation logic (lines 77-138) is sophisticated but a wall of prose. Five interacting mechanics (cross-promo gate, 90-day cooldown, HTML marker placement, reset conditions, detect-then-recommend) need subsections + worked example.
- 🟡 `/unforget scan` staleness thresholds (lines 222-231) reference a config-override mechanism (line 233) that is **never documented**. Where does the user write it?
- 🟢 No `/unforget repair` command despite README having a recovery section. Either add the command or move recovery firmly to docs.

### `reference/surfaces.md`

**Summary:** 173 lines. Six surfaces + Surface 1b heuristics, per-surface pre-checks, GitHub four-state, memory-dir pinning, dedup, algorithm fallback.

**Strengths**
- 🟠 Memory-dir pinning with three-tier read order + heal-on-contact (lines 59-78).
- 🟠 GitHub four-state design (no-git / git-without-github / github-but-no-`gh` / fully-wired) right thinking for environment heterogeneity.

**Weaknesses**
- 🟡 Audit-tool version pinning silent on forward-compat (line 124) — hardcoded to radar-suite v3 patterns; v4 will silently stop matching.
- 🟡 Algorithm fallback (lines 161-173) is the project's deepest non-determinism source. Spec acknowledges this but doesn't mandate user-facing behavior (warn? refuse? cap to init-only?).
- 🟢 Redirect-pointer pre-check (lines 26-33) can produce false negatives if a 12-line file points at a non-existent target.

### `reference/promotion.md`

**Summary:** 134 lines. Release-time promote ritual, post-fix-sweep workflow chaining three skills, backup rotation with 5-deep retention, recovery.

**Strengths**
- 🟠 Post-fix-sweep workflow (lines 7-50): surface → verify → generalize is a high-leverage three-skill chain with a worked example.
- 🟠 Backup rotation (lines 85-101): 5-deep retention, lexicographically sortable filenames, `.gitignore` recommendation.
- 🟠 Cross-skill version pinning (lines 109-115) — rare and disciplined.

**Weaknesses**
- 🟡 "Fail-soft" undefined operationally (lines 109-115) — suppress the recommendation? Show it with a warning? Refuse and exit?
- 🟢 Promote step 1 ("Verify every 🔴 THIS row has Status = Fixed") doesn't specify the UX when one isn't.
- 🟢 Recovery (lines 113-122) references "the README.md section" — pointer is ambiguous between project README and a recovery.md.

### `examples/UNFORGET.md`

**Summary:** Fully populated example with 19 rows across four sections, comprehensive detail blocks with closure pointers and spawn links.

**Strengths**
- 🟠 Closure pointers exact per spec; bidirectional spawn links (P3 ↔ P6) validate the chain-walkable invariant.

**Weaknesses**
- 🟡 **Not a single row demonstrates the verify-still-open recipe.** The project's most innovative idea (format.md:73-95) is invisible in the canonical example. A new user reading the example before the spec never learns the pattern exists.
- 🟢 Column abbreviations (`Urg`, `RFix`, `RNo`) used in example but full names used in spec; convention isn't declared.

### `.claude-plugin/plugin.json` and `marketplace.json`

**Summary:** plugin.json (25 lines) is minimal; marketplace.json (23 lines) has a fuller description.

**Strengths**
- 🟠 Keywords (line 11-17 of marketplace.json) include `deferred-work` and `backlog`, which are appropriately specific.

**Weaknesses**
- 🟡 Descriptions diverge between files. marketplace.json is more informative; plugin.json should match.
- 🟡 Neither file lists natural-language activation phrases. Whether Claude Code's router reads SKILL.md frontmatter or manifest description is unverified.
- ⚪ "task-management" keyword too generic.

### `scripts/README.md` and `scan_surfaces.py`

**Summary:** Five helpers (encode_project_path, scan_surfaces, dedup_findings, check_format_version, prune_backups). Python 3.9+ stdlib only, JSON in/out.

**Strengths**
- 🟠 Stdlib-only constraint removes `pip install` friction.
- 🟠 Algorithm-fallback discipline: every script must have a prose fallback.
- 🟠 `scan_surfaces.py` consolidates `AUDIT_FILENAME_RE` for both Surface 2 and Surface 1b (single source of truth).

**Weaknesses**
- 🟡 `scripts/README.md` says self-test corpus is "queued for v0.3" — but `tests/` harness already exists with golden files. Update.
- 🟢 200-char truncation in code-comment surface output (Surface 4) sensible but undocumented in surfaces.md.

### `tests/`

**Summary:** Bash harness, golden files, normalizer, `--bless` mode for re-baselining.

**Strengths**
- 🟠 Exit-code design (line 13-15 of run.sh) deliberately not `-e`; pass/fail from diff. Correct.
- 🟠 `--bless` with "review the diff" emphasis — disciplined.

**Weaknesses**
- 🟡 Surface 6 (memory files) untested in CI — non-determinism. Fix: add `--memory-dir` override to `scan_surfaces.py`.
- 🟡 `prune_backups.py` uncovered. Only destructive helper. Untested destructive code is a classic incident vector.

---

## Cross-file findings

### Inconsistencies

- 🟡 Conservative defaults duplicated in `init.md` (lines 139-152) and `commands.md` `/unforget add` (lines 29-36). Currently agree. Extract to a single subsection in `format.md`.
- 🟡 Closure pointer format appears in 3 places (`format.md:54-63`, `examples/UNFORGET.md:54, 95`, tangentially in `promotion.md`). Pick `format.md` as canonical and have others reference.
- 🟡 plugin.json vs marketplace.json description mismatch.
- 🟢 Column-name abbreviation (`Urg` vs `Urgency`) — implicit convention, never declared.
- 🟢 Companion files table in SKILL.md doesn't list `scripts/`.

### Missing pieces

- 🟡 No `/unforget repair` command despite README having a recovery section. Either add it or move recovery firmly to docs.
- 🟡 No documented behavior for malformed detail blocks. What does `/unforget list` do when row P3 has a bad spawn-link?
- 🟡 Activation phrases not in `.claude-plugin/*.json`. Discoverability risk if router reads manifests.
- 🟡 Config-override for staleness thresholds referenced but unspecified.
- 🟢 No machine-readable export (`--json`, `--csv`). Unlocks CI / dashboards.
- 🟢 Pause/resume integration with Claude Code's plan-pause state unclear.

### Redundancy

- See "Inconsistencies" above (closure pointer × 3, conservative defaults × 2).
- 🟢 Anti-patterns appear in SKILL.md (lines 150-159) AND `format.md`. Currently agree; could drift.

### Trigger-phrasing quality

- 🟡 SKILL.md frontmatter has trigger phrases but they're buried in YAML-multi-line. Rewrite verb-first: `description: |\n  Use when the user asks "what's deferred?", "what's the backlog?"…`
- 🟢 Manifests don't include any trigger-phrase hints.

### Error handling and edge cases

- 🟡 Python-fallback trigger undefined (missing binary? non-zero exit? bad JSON?).
- 🟡 UNFORGET.md missing during `add`/`edit`/`list` — spec silent.
- 🟢 Fuzzy dedup threshold in `import` not specified.
- 🟢 Memory-dir pin drift heals silently — could warn on `--version` output.
- 🟢 Plan-file → row conversion: what happens when the plan file is deleted later?

### Slash-command registration

- 🟢 Plugin.json registers `unforget` skill name; subcommand auto-complete not verified.
- 🟢 No-args form (`/unforget` with no args) behavior not specified.

### Tooling and parseability

- 🟡 Compact preset has no canonical regex.
- 🟢 No machine-readable export option.

### Test coverage

- 🟡 Surface 6 (memory) untested.
- 🟡 `prune_backups.py` (only destructive helper) untested.
- 🟢 Surface 5 (GitHub) stubbed.

---

## Quality signals

### Unusually well-done — keep doing these

1. 🟠 **Verify-still-open recipe** (`format.md:73-95`) — treats row staleness as a first-class concern. **Generalize:** every claim about file state should have a re-verification command.
2. 🟠 **Conservative-defaults contract** (`init.md:139-152`) — pinned identical defaults across implementations removes a class of variability bugs.
3. 🟠 **Post-fix-sweep workflow** (`promotion.md:7-50`) — three-skill chain (surface → verify → generalize) with a worked example.
4. 🟠 **Spec-substitution principle** (`SKILL.md:101`) — named architectural rule. Will pay off as the skill grows.
5. 🟠 **Empty-case reframing** (`init.md:101-105`) — reframes "no results" as success, not failure.
6. 🟠 **Honest maturity callout** (`README.md:64-71`) — "works for one project shape, needs feedback from others" builds trust.
7. 🟠 **Format-version contract with read-only fallback** (`SKILL.md:138-146`) — handles future versions gracefully.
8. 🟠 **Backup rotation** (`promotion.md:85-101`) — destructive operation made safe with 5-deep retention.
9. 🟠 **Decision flowchart** (`SKILL.md:74-84`) — answers "which subcommand?" before the user asks.
10. 🟠 **Single source of truth for AUDIT_FILENAME_RE** (`scan_surfaces.py:94`) — consolidated regex used by both Surface 2 and Surface 1b.

### Weak or unclear — fix these

1. 🟡 Compact preset parsing regex missing (`format.md:110-122`)
2. 🟡 Closure-recommendation prose is a wall of text (`commands.md:77-138`)
3. 🟡 Staleness threshold config-override referenced but not specified (`commands.md:233`)
4. 🟡 Surface 6 + `prune_backups.py` untested
5. 🟡 Activation phrases buried in YAML-multi-line (`SKILL.md:4-9`)
6. 🟡 Two duplicate sources of truth (closure pointer × 3, conservative defaults × 2)
7. 🟡 `examples/UNFORGET.md` doesn't demonstrate verify-still-open
8. 🟡 "Fail-soft" undefined in promotion.md cross-skill table
9. 🟡 Audit-tool version pinning silent on forward-compat (`surfaces.md:124`)
10. 🟢 No machine-readable export option

---

## Recommended actions

| Rank | Severity | Action | File:Line | Effort | Quick win? | Why |
|---|---|---|---|---|---|---|
| 1 | 🟡 HIGH | Add verify-still-open recipe to ≥2 rows in canonical example | `examples/UNFORGET.md` | Small | ✅ | Canonical example must demonstrate the project's signature pattern |
| 2 | 🟡 HIGH | Add Compact preset parsing regex with valid/invalid test cases | `format.md § Compact preset` | Small | ✅ | Unblocks downstream tooling; prevents drift |
| 3 | 🟡 HIGH | Rewrite SKILL.md `description:` verb-first ("Use when…") | `SKILL.md:4-9` | Small | ✅ | Likely improves routing match rate |
| 4 | 🟡 HIGH | Pin Python-fallback triggers explicitly | `init.md:64` | Small | ✅ | Removes implementation-defined behavior |
| 5 | 🟡 HIGH | Specify or remove staleness-threshold config-override | `commands.md:233` | Small | ✅ | Either a real feature or remove the dangling reference |
| 6 | 🟡 HIGH | Extract conservative-defaults to `format.md`, reference from init/add | `format.md`, `init.md`, `commands.md` | Small | ✅ | Single source of truth |
| 7 | 🟡 HIGH | Add `--memory-dir` override flag to `scan_surfaces.py`; unsuppress Surface 6 | `scripts/scan_surfaces.py`, `tests/` | Medium | | Closes the biggest test-coverage gap |
| 8 | 🟡 HIGH | Add fixture-based test for `prune_backups.py` | `tests/`, new fixture | Medium | | Destructive code without tests is a footgun |
| 9 | 🟡 HIGH | Specify "fail-soft" operationally | `promotion.md:109-115` | Small | ✅ | Removes implementation-defined behavior |
| 10 | 🟡 HIGH | Match plugin.json and marketplace.json descriptions | `.claude-plugin/*.json` | Small | ✅ | One source of truth for skill identity |
| 11 | 🟢 MEDIUM | Refactor closure-recommendation prose into subsections | `commands.md:77-138` | Medium | | Comprehensibility |
| 12 | 🟢 MEDIUM | Add `/unforget repair` OR move recovery firmly to docs | `commands.md` or `README.md` | Large | | Resolve implicit promise from README's recovery table |
| 13 | 🟢 MEDIUM | Add machine-readable export (`--json`/`--csv` to `list`) | `commands.md`, scripts | Medium | | Unlocks CI integration |

**Quick-wins total:** 7 items, ~3.5 hours.

---

## Verification

Run the existing test harness:

```
cd /Volumes/2 TB Drive/Coding/GitHub/unforget
./tests/run.sh
```

For spec-only changes (regex addition, description rewrites), no automated test exists. Validate via:

1. Run `/unforget list` with a Compact-preset fixture; verify rows render correctly.
2. Hand-construct an UNFORGET.md with each preset variant; run all read commands (`list`, `scan`).
3. For example changes: run `/unforget init` in a throwaway directory; confirm generated file matches the example structure.

For broader changes (`/unforget repair`, `--json` export), extend `tests/run.sh` with new fixtures and golden files before implementing.

---

## Files referenced

- SKILL.md
- README.md
- reference/format.md
- reference/init.md
- reference/commands.md
- reference/surfaces.md
- reference/promotion.md
- examples/UNFORGET.md
- scripts/README.md
- scripts/scan_surfaces.py (skim)
- tests/README.md
- tests/run.sh
- .claude-plugin/plugin.json
- .claude-plugin/marketplace.json

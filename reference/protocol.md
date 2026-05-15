# Review protocol

Authoritative spec for `/skill-reviewer review <path>`. This file defines how the review is conducted: the reviewer's role, what to read, what to evaluate, and what not to do.

When operating under a lens (e.g. `--lens=safety`), apply only the sections of this protocol that `reference/lenses.md` calls out for that lens, and skip the rest.

---

## Your role

You are a senior reviewer giving honest, no-bullshit feedback to a skill author who wants to improve their work. They want real critique, not polite validation. Equally important: call out things that are unusually well-done so they know what to keep doing.

Treat the author as a peer. Assume strong baseline competence and skip elementary explanations. They've already shipped at least one skill; they don't need definitions of "frontmatter" or "slash command."

**Tone calibration:**
- Willing to say "this is fragile" or "this will silently fail" when warranted
- Equally willing to say "this is unusually well-done" — both signals matter
- No filler ("This is a great skill!" without specifics)
- No hedging ("might possibly be" → say what you observed and what the risk is)
- Match scope to severity: a typo gets one line; an architectural risk gets a paragraph

---

## What to read

Read these thoroughly (full content, not excerpts):

1. The top-level `SKILL.md` (or equivalent main spec file)
2. `README.md` if present
3. Every file under `reference/`, `docs/`, or any other spec directory
4. `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` if present
5. Contents of `examples/` directory
6. `scripts/README.md` and skim each script (describe inputs/outputs/approach; don't quote line-by-line)
7. `tests/README.md`, `tests/run.sh`, and any test harness or fixture files
8. Any version-tracking files (`CHANGELOG.md`, `vN-roadmap.md`)

If the skill is large (>10 reference files or >3,000 lines of spec), dispatch parallel Explore subagents in batches. But personally read SKILL.md, README.md, and the manifest files — these set the skill's voice and discoverability and require continuous attention.

If a file is referenced from another file but doesn't exist, note it as a dangling reference rather than skipping silently.

---

## What to evaluate

### Per-file evaluation

For each substantive file:

- **2-4 sentence summary** of what it does
- **3-6 specific observations** with file:line citations
- **Direct quotes** for anything specific you want to flag

Categorize observations as: strengths (what's well-done), weaknesses (what's risky or unclear), or surprises (anything that broke your expectations).

### Cross-file evaluation

After per-file analysis, look across files for the following.

#### Inconsistencies

- Does SKILL.md describe behavior that reference files contradict?
- Do `plugin.json` and `marketplace.json` descriptions match?
- Are status enums, column names, or other formal vocabulary used consistently across files?
- Do examples actually demonstrate what the spec describes? (Common failure: spec describes a feature, example never uses it.)

#### Missing pieces

- Are slash commands or features referenced that have no implementation or spec?
- Does error handling exist for: missing files, malformed input, failed external dependencies, permission errors, disk-full?
- What happens when external dependencies (Python, `gh`, other skills) are missing or broken? Is the fallback trigger explicit?
- Are there dangling references to files, sections, or commands that don't exist?

#### Redundancy

- Same content in multiple places that could drift over time?
- For each duplication: pin a single source of truth and cross-reference, OR note that the duplication is intentional and acceptable.

#### Trigger-phrasing quality

- Will the skill activate when a user says natural-language phrases the description hints at?
- Are activation phrases buried in YAML-multi-line, or directive-first ("Use when...")?
- Does the manifest description match the SKILL.md frontmatter description? If they differ, which one does Claude Code's router actually read?
- Compare against Anthropic-published skill phrasing conventions (verb-first, concrete trigger phrases listed early)

#### Error handling and edge cases

- Destructive operations: backups? confirmations? recovery procedures?
- What happens to existing state when a command is re-run, cancelled mid-flow, or fails partway?
- Forward-compatibility: how does the skill handle future format versions, future versions of external tools it depends on?
- Non-determinism: any LLM-driven fallback paths that will produce different results across sessions? If yes, is the user warned?

#### Slash-command registration

- Do all subcommands described in spec files have corresponding plugin registration?
- What does the no-args form of the primary command do?
- Are help text and auto-complete plausibly working?

#### Tooling and parseability

- Can downstream tools (CI linters, dashboards, external scripts) parse the skill's output formats?
- Are canonical regexes or schemas provided where needed?
- Is there a machine-readable export option, or is everything markdown-only?

#### Test coverage

- What's tested? What's explicitly untested? What's *silently* untested (no acknowledgement)?
- Are destructive helpers covered? (Common gap.)
- Will the test suite catch regressions in the core features, or only in the easy-to-test helpers?

### Quality signals (always)

Two ranked lists at the end:
- **Unusually well-done — keep doing these** (5-10 items, each with file:line)
- **Weak or unclear — fix these** (5-10 items, each with file:line)

Both are required. A review that only lists weaknesses tells the author what to change but not what to preserve.

---

## Output

See `reference/output-format.md` for the required report structure.
See `reference/severity-rubric.md` for the findings table format (severity colors, effort estimates, quick-win tags).

---

## Anti-patterns to avoid in your review

These produce reviews that *look* thorough but aren't useful:

- **Don't suggest features the skill explicitly declined to build.** Check anti-patterns / "what this won't do" sections first. If the skill's README says "won't accept multi-file splits," don't recommend splitting the file.
- **Don't recommend changes that contradict the skill's stated philosophy** without flagging that you're doing so and explaining why the philosophy might be wrong.
- **Don't pad with generic advice.** "Consider adding tests" is filler. "Add a fixture-based test for `prune_backups.py` because it's the only destructive helper and currently uncovered" is useful.
- **Don't comment on code style or formatting** unless it materially affects correctness or parseability. A misaligned markdown table is a real issue; a missing trailing newline is not.
- **Don't propose adding ceremony** (license headers, badge collections, contribution guidelines, code-of-conduct files) unless the skill is missing something the author would actually want.
- **Don't grade with a single number.** "B+ skill" hides the dimensions. Multi-axis findings are the point.
- **Don't make findings up.** If you didn't read the file, don't cite it. If you're inferring from a filename, say so explicitly ("the filename suggests X, though I didn't read the file").

---

## Verification before finalizing

Before you finalize the report, ask yourself:

1. Did I read every file I'm citing, or am I inferring from filenames?
2. Did I check whether each "missing piece" is actually missing, or just in a file I didn't read?
3. Did I verify that "inconsistencies" are actual contradictions, not just different views of the same fact?
4. Am I being precise about what *fails* vs. what is merely *under-specified*? (Under-specified is weaker than broken.)
5. Did I cite file:line for every specific claim?
6. Did I list at least 5 strengths? (Reviews that find only weaknesses are incomplete.)

If any answer is no, fix it before delivering.

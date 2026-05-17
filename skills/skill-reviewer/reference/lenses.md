# Lens variants

A **lens** is a curated subset of `reference/protocol.md` that focuses the review on one concern. Use a lens when you want depth in one area instead of breadth across all areas.

Invoked as: `/skill-reviewer review <path> --lens=<name>`

If no `--lens` flag is provided, the default is `full`.

Lenses produce reports in the card format defined in `reference/output-format.md` (TL;DR → Strengths → Findings → Patterns → Next step). Per-lens "Output:" lines below describe which **sections** the lens emphasizes and which it suppresses, not a separate report structure. Word budgets per lens live in `reference/output-format.md § Word budget by lens`.

---

## Available lenses

| Lens | Focus | Skip |
|---|---|---|
| `full` | Everything in `protocol.md` | — |
| `discoverability` | Trigger phrases, manifest descriptions, README clarity, install UX, activation paths | Test coverage, parseability, internal architecture |
| `safety` | Destructive operations, backups, recovery, error handling, fail-soft, forward-compat | Trigger phrasing, README, parseability |
| `architecture` | File structure, single-source-of-truth, spec-substitution discipline, redundancy, cross-file refs | Trigger phrasing, install UX |
| `parseability` | Output formats, canonical schemas, regex provided where needed, machine-readable export | Discoverability, README narrative |
| `tests` | What's tested, what's silently untested, coverage of destructive helpers, fixture quality | Trigger phrasing, README, parseability |
| `quick` | TL;DR + 3-5 strengths + up to 5 finding cards + Next step | Patterns section, second-opinion banner |

**Word budgets** for each lens (target and hard cap) live in `reference/output-format.md` § Word budget by lens. That table is canonical; if you need length guidance, read it there.

---

## Lens specs

### `full` (default)

Apply every section of `reference/protocol.md`. Produce the full report structure from `reference/output-format.md`. No skips.

Use when: this is the first time reviewing the skill, or the author wants a comprehensive pre-publication audit.

### `discoverability`

**Apply:**
- "Trigger-phrasing quality" section of protocol.md (in full)
- Manifest cross-checks (plugin.json vs marketplace.json descriptions)
- README clarity for first-time readers
- Install UX (number of steps, fallback paths, recovery from misclick)
- SKILL.md frontmatter description phrasing (verb-first? trigger phrases listed?)
- Activation path verification: would a user saying X cause this skill to fire?

**Skip:**
- Test coverage
- Parseability and machine-readable export
- Internal architecture and file structure (unless it affects discoverability)
- Per-file findings for `reference/*.md` (these are author-facing, not user-facing)

**Output:** TL;DR + Strengths + Findings (focused on manifest/README/frontmatter issues) + Patterns + Next step. Skip cards about internal reference docs unless they affect discoverability.

Use when: the skill is built and you're asking "will users find this and trigger it correctly?"

### `safety`

**Apply:**
- Error handling and edge cases (in full)
- Destructive operations review: backups, confirmations, recovery procedures
- Forward-compatibility (future format versions, external tool versions)
- Fail-soft definitions (what does "fail-soft" mean operationally in this skill?)
- Non-determinism warnings (LLM fallback paths)
- Recovery documentation completeness

**Skip:**
- Trigger phrasing
- README narrative quality (unless recovery instructions are in the README)
- Parseability

**Output:** TL;DR + Strengths + Findings (focused on destructive operations, error handling, and recovery) + Patterns + Next step. Findings heavily weighted toward 🔴 CRITICAL and 🟡 HIGH.

Use when: the skill performs destructive operations (file deletion, schema changes, git operations) or is heading to production users.

### `architecture`

**Apply:**
- Inconsistencies across files
- Redundancy and duplicate sources of truth
- Spec-substitution discipline (thin index vs fat reference)
- Cross-file references (do they resolve? are they brittle?)
- File-structure rationale (is the architecture explained somewhere?)
- Companion files table accuracy

**Skip:**
- Trigger phrasing (covered by `discoverability` lens)
- Install UX

**Output:** TL;DR + Strengths + Findings (focused on inconsistencies, redundant sources of truth, and cross-file references) + Patterns (this lens uses Patterns heavily — architectural drift often shows up only when you compare files) + Next step.

Use when: the skill has grown to >5 reference files or is being prepared for contributor onboarding.

### `parseability`

**Apply:**
- Tooling and parseability section (in full)
- Output format precision (regex, schemas, examples)
- Machine-readable export options (`--json`, `--csv`)
- Downstream tool compatibility (CI, dashboards)
- Format-version contract review

**Skip:**
- Trigger phrasing
- README narrative
- Per-file detail for everything except files defining output formats

**Output:** TL;DR + Strengths + Findings (focused on output-format precision, downstream-tool compatibility, and missing schemas/regexes) + Patterns + Next step. Card Fix lines should include the actual regex or schema when proposing one.

Use when: the skill produces structured output that other tools will consume, or the author wants to build CI integrations.

### `tests`

**Apply:**
- Test coverage section (in full)
- Per-helper coverage analysis (especially destructive helpers)
- Silent-untested detection (is anything untested without acknowledgement?)
- Test harness quality (deterministic? bless-mode discipline?)
- Fixture quality

**Skip:**
- Trigger phrasing
- README
- Architecture (unless it blocks testability)

**Output:** TL;DR + Strengths + Findings (one card per helper or feature with a coverage gap, plus cards for silent-untested patterns) + Patterns + Next step. The Patterns section is a good place to summarize coverage gaps that span multiple helpers.

Use when: the skill has helper scripts/code and the author wants a test-coverage audit before adding more features.

### `quick`

**Apply:**
- TL;DR (1 paragraph, no weakness clusters block)
- Strengths (3-5 items, ratchet down from 5-10)
- Findings (5 cards maximum, sorted by severity)
- Next step (1 line)

**Skip:**
- Patterns across findings (the quick lens is too short for cross-cutting synthesis)
- Second-opinion banner unless `--second-opinion` was explicitly passed

**Output:** ~500 words target, 800 hard cap. Cards stay the same shape as the full lens — Why + Fix + citation — just capped at 5.

Use when: the author wants a fast sanity check before deeper work, or to triage which lens to apply next.

---

## Choosing a lens

Decision flowchart:

- **First review of this skill, or comprehensive audit before publishing** → `full`
- **Want a 5-minute read to triage** → `quick`
- **Skill does destructive things** → `safety`
- **Skill has >5 reference files or contributors joining** → `architecture`
- **Skill produces output other tools consume** → `parseability`
- **Skill has scripts/code with tests** → `tests`
- **Asking "will users actually find this?"** → `discoverability`

Multiple lenses can be run sequentially on the same skill. Each produces an independent report. The full lens subsumes the others; running `full` after a single-lens review may surface the same findings.

---

## Adding a new lens

To propose a new lens variant:

1. Identify the concern. Is it covered by an existing lens? If yes, expand that lens instead.
2. Write a focus list (sections of protocol.md to apply) and a skip list.
3. Set a target report length range.
4. Add the lens to the table at the top of this file and a spec subsection below.
5. Update `reference/output-format.md` if the lens needs a non-standard report structure.

Anti-pattern: don't add lenses that are just "the full review but shorter." That's what `quick` is for.

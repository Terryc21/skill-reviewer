# Skill detection and adaptive depth

Spec for `/skill-reviewer detect <path>` and the detection step that runs implicitly at the start of every `review` invocation.

---

## Why detection exists

Skills come in three shapes:

- **Single-file** — one `SKILL.md` and nothing else
- **Thin-index** — `SKILL.md` plus a `reference/` directory of spec files
- **Plugin** — thin-index plus `.claude-plugin/` manifests for marketplace install

A review that ignores the shape produces useless feedback: a 4,000-word audit of a 60-line single-file skill is overkill; a 1,500-word audit of a 12-file plugin skill is too shallow.

The detection step classifies the skill and tells the review protocol how deep to go.

---

## Detection rules

A path is a **skill** if any of the following is true:

1. The path contains `SKILL.md` at the top level
2. The path contains `.claude-plugin/plugin.json`
3. The parent directory is `~/.claude/skills/` (v0.1 manual-install layout)
4. The path itself is a `.md` file named `SKILL.md` (review the parent directory)

If none of these match, the skill-reviewer refuses to run with the message:

> "Not a skill. Path `<path>` does not contain SKILL.md or .claude-plugin/plugin.json. To force a review anyway, re-run with `--force`."

The `--force` flag bypasses detection and runs the protocol on whatever's at the path. Useful for reviewing a draft skill before its file structure stabilizes.

---

## Classification

Once a skill is detected, classify it as one of three shapes.

### Single-file

**Detected when:** only `SKILL.md` is present at the path. No `reference/`, no `.claude-plugin/`, no `examples/`.

**Adaptive depth:**
- Collapse the per-file section to one subsection (no "per-file" findings if there's only one file)
- Skip the "Companion files" cross-file check
- Skip the manifest cross-check
- Target report length: **~1500 words**
- Severity rubric still applies; recommended actions table still required

### Thin-index

**Detected when:** `SKILL.md` is present AND a `reference/` directory exists with ≥1 spec file. May or may not have `examples/`, `scripts/`, `tests/`.

**Adaptive depth:**
- Standard per-file findings for SKILL.md and every reference/*.md
- Apply the full cross-file analysis from `protocol.md`
- Skip manifest cross-check unless `.claude-plugin/` is present
- Target report length: **~2500 words**

### Plugin

**Detected when:** thin-index plus `.claude-plugin/plugin.json`. May or may not have `marketplace.json`.

**Adaptive depth:**
- Standard per-file findings
- Full cross-file analysis
- **Add:** manifest cross-check (plugin.json vs marketplace.json descriptions match? activation phrases present?)
- **Add:** install-path verification (do the install instructions in README.md actually work?)
- **Add:** marketplace-readiness check (LICENSE present? .gitignore? README badges or status indicator?)
- Target report length: **~3500 words**

---

## Mixed / edge cases

### Mixed-shape (unusual)

If a path has `.claude-plugin/` but no `SKILL.md`, OR has `SKILL.md` but `.claude-plugin/` is empty, classify as **plugin** but note the structural anomaly in the report's TL;DR.

### Multi-skill repos

If a path contains multiple skills (e.g., a monorepo of plugins, each in its own subdirectory), refuse to run with:

> "Multi-skill repo detected. Run `/skill-reviewer review <path>/<skill-name>` to review one skill at a time."

Multi-skill repos require per-skill review with optional cross-skill comparison, not a single mashed-together report.

### Skill-under-development (no `SKILL.md` yet)

If the path looks like a skill-in-progress (has `reference/` files but no SKILL.md), treat as single-file with `--force` implied, but note in the TL;DR: "this skill is missing its SKILL.md index — write that first, then re-review."

---

## `/skill-reviewer detect <path>` output

When invoked standalone (not as part of `review`), `detect` outputs:

```
Path: /Volumes/2 TB Drive/Coding/GitHub/unforget
Classification: plugin
Detected via: SKILL.md present, .claude-plugin/plugin.json present
Files inventoried:
  - SKILL.md (165 lines)
  - README.md (314 lines)
  - reference/format.md (154 lines)
  - reference/init.md (285 lines)
  - reference/commands.md (301 lines)
  - reference/surfaces.md (173 lines)
  - reference/promotion.md (134 lines)
  - examples/UNFORGET.md (~140 lines)
  - scripts/README.md, scripts/*.py (6 files)
  - tests/ (harness + golden files, 6 files)
  - .claude-plugin/plugin.json, marketplace.json
  - LICENSE, .gitignore

Recommended review depth: ~3500 words (plugin shape)
Suggested lenses: full (first review), then safety (destructive ops in promote), then discoverability (pre-publication)
```

This output is one diagnostic; no actions taken. Useful for previewing what `review` will do without running the full protocol.

---

## Adaptive-depth budgets

These are guidance, not hard limits. A skill with unusually rich or unusually shallow content may justify going over or under.

| Shape | Files reviewed | Target words | When to override |
|---|---|---|---|
| Single-file | 1-2 | ~1500 | Skill has extensive examples → can go up to 2000 |
| Thin-index | 3-6 | ~2500 | Skill has >6 reference files → scale up by ~400/file beyond 6 |
| Plugin | 7-15+ | ~3500 | Skill has >15 files → scale up by ~250/file beyond 15 |

If the review is going to exceed double the target word count, stop and apply a lens instead of producing a sprawling full review.

---

## Refusal patterns

Detection should refuse fast and clearly:

| Condition | Refusal message |
|---|---|
| Path does not exist | "Path `<path>` does not exist." |
| Path is a file but not SKILL.md | "Path `<path>` is a file, not a skill directory. Pass the parent directory or pass a SKILL.md." |
| Path has no skill markers (use `--force` to override) | "Not a skill. Path `<path>` does not contain SKILL.md or .claude-plugin/plugin.json. To force a review anyway, re-run with `--force`." |
| Path is a multi-skill repo | "Multi-skill repo detected. Run `/skill-reviewer review <path>/<skill-name>` to review one skill at a time." |

Refusal messages should always include the actionable next step.

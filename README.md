# skill-reviewer

> **Candid reviews of Claude Code skills. File:line citations. Ranked actions. No filler.**

A Claude Code skill that reviews other skills. Produces structured reports with severity-rated findings, citations to specific lines, ranked recommended actions, and an optional second-opinion reconciliation pass.

Built because polite-by-default reviews ship skills with quiet flaws.

## TL;DR

- **Problem:** Skill reviews tend toward hedged generic feedback ("consider adding tests," "you might think about discoverability"). Authors can't act on it.
- **Solution:** A protocol that requires file:line citations for every claim, balances strengths and weaknesses, tags quick wins, and supports lens variants for focused audits.
- **Install:** two `/plugin` commands in Claude Code (below). Skill becomes available as `/skill-reviewer` in any session.
- **Use:** `/skill-reviewer review <path>` for a full review. `/skill-reviewer summary <path>` for a 5-minute sanity check.
- **Lenses:** focused reviews via `--lens=safety`, `--lens=discoverability`, `--lens=architecture`, `--lens=parseability`, `--lens=tests`, or `--lens=quick`.
- **Second opinion:** `--second-opinion` dispatches an independent reviewer to challenge the draft's top conclusions.
- **Maturity:** v0.1.0. Extracted from a real review session reviewing the `unforget` skill. See `examples/sample-report-unforget.md` for canonical output.

## Install

Run these two commands **one at a time** in Claude Code. Wait for Step 1 to confirm "Successfully added marketplace" before running Step 2.

Step 1 — add the marketplace:

```
/plugin marketplace add Terryc21/skill-reviewer
```

Step 2 — install the plugin:

```
/plugin install skill-reviewer@skill-reviewer
```

The skill is now available. To verify, type `/skill-reviewer --version` in any session.

<details>
<summary><strong>Why two separate commands?</strong></summary>

If you copy both `/plugin` lines at once and paste them into Claude Code, the slash-command dispatcher treats the first `/plugin` as the command and the rest of the paste as its arguments. Run them one at a time.
</details>

<details>
<summary><strong>Manual install (fallback)</strong></summary>

If the plugin path isn't available in your environment:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Terryc21/skill-reviewer.git ~/.claude/skills/skill-reviewer
```

Then invoke as `/skill skill-reviewer` (with the prefix). To update later: `cd ~/.claude/skills/skill-reviewer && git pull`.
</details>

## Quick start

In any Claude Code session, point at a skill directory:

```
/skill-reviewer review /path/to/some-skill
```

A full review takes 2-5 minutes depending on the skill's size. Output is a structured markdown report.

For a 5-minute sanity check instead:

```
/skill-reviewer summary /path/to/some-skill
```

For a focused review of one concern:

```
/skill-reviewer review /path/to/some-skill --lens=safety
```

For a pre-publication audit:

```
/skill-reviewer review /path/to/some-skill --second-opinion
```

## Subcommand reference

| Command | What it does |
|---|---|
| `/skill-reviewer review <path>` | Full review. Default lens is `full`. |
| `/skill-reviewer review <path> --lens=<name>` | Focused review under a specific lens. |
| `/skill-reviewer review <path> --second-opinion` | Full review followed by an independent reviewer pass that challenges the top 3 conclusions. |
| `/skill-reviewer compare <path1> <path2>` | Side-by-side review of two skills. |
| `/skill-reviewer summary <path>` | Equivalent to `review --lens=quick`. ~800 words. |
| `/skill-reviewer lenses` | Print the lens catalog. |
| `/skill-reviewer detect <path>` | Classify the skill (single-file / thin-index / plugin) without reviewing. |
| `/skill-reviewer --version` | Print version and install path. |

## Lens variants

A lens narrows the review to one concern. Use a lens when you want depth in one area instead of breadth across all areas.

| Lens | Focus | Best for |
|---|---|---|
| `full` (default) | Everything | First review or comprehensive pre-publication audit |
| `discoverability` | Trigger phrases, manifest descriptions, README clarity, install UX | "Will users find this and trigger it correctly?" |
| `safety` | Destructive operations, backups, recovery, error handling, forward-compat | Skills that delete data, modify state, or run destructive ops |
| `architecture` | File structure, single source of truth, redundancy, cross-file refs | Skills with >5 reference files or onboarding contributors |
| `parseability` | Output formats, canonical schemas, regex provided where needed, machine-readable export | Skills whose output is consumed by other tools |
| `tests` | What's tested, what's silently untested, destructive-helper coverage | Skills with helper scripts |
| `quick` | TL;DR + top 5 strengths + top 5 weaknesses + top 5 actions | Triage |

Full lens specs in `reference/lenses.md`.

## See it first

Excerpt from [`examples/sample-report-unforget.md`](examples/sample-report-unforget.md):

```markdown
## TL;DR

`unforget` is a Claude Code skill that consolidates deferred work into one
structured UNFORGET.md per project. It's more disciplined than most skills
I've reviewed: thin SKILL.md → fat reference files, deterministic Python
helpers with prose fallbacks, backups before destructive operations, and the
**verify-still-open recipe** is a genuinely original idea worth generalizing.
The standout strength is structural; the standout weakness is that the
canonical example never demonstrates the project's signature feature.

**Weakness clusters:**
1. Tooling and parseability gaps — Compact preset has no canonical regex
2. Discoverability — natural-language trigger phrases buried in YAML
3. Test coverage — Surface 6 and prune_backups.py are explicitly untested
4. Duplicate sources of truth — closure pointer format lives in 3 places
5. Under-specified fallbacks — "fail-soft" and Python-missing triggers vague
```

Notice: file:line citations are required, severity colors are explicit, and the report calls out strengths as concretely as weaknesses. The full report continues with per-file findings, cross-file analysis, a ranked actions table, and verification instructions.

## When to use skill-reviewer (and when not)

**Use skill-reviewer when:**

- You've built a skill and want honest feedback before publishing
- You're reviewing someone else's skill and want a structured approach
- You're auditing a skill's safety, discoverability, or test coverage specifically
- You're comparing two skills to decide which patterns to keep
- You want a second-opinion pass to challenge your own first impressions

**Don't use skill-reviewer when:**

- You want auto-generated edits (skill-reviewer produces reports, not patches)
- You want a single grade or score (multi-axis findings are the point)
- You're reviewing something that isn't a skill (the detection step will refuse with `--force` available as override)

## How it works with other tools

- **`unforget`** — track skill-reviewer findings you defer (e.g., "fix in v0.3") as rows in Section 3 (Audit findings) of an UNFORGET.md.
- **`radar-suite`** — audit findings from radar-suite can feed into a skill-reviewer report's per-file findings if the skill under review uses radar-suite output formats.
- **`bug-echo`** — after fixing one finding, bug-echo can sweep for similar patterns across the skill.

## Self-review

skill-reviewer can review itself:

```
/skill-reviewer review /Volumes/2 TB Drive/Coding/GitHub/skill-reviewer
```

Findings should be specific to this skill — gaps in its own protocol or lens definitions. Use this as a sanity check after any meaningful edit to the reference files.

## Origin

skill-reviewer was extracted from a Claude Code session reviewing the `unforget` skill. The user wanted candid feedback with file:line citations and ranked actions. After producing the review, they asked for a reusable prompt; after using the prompt, they asked to turn it into a skill. That review is preserved in `examples/sample-report-unforget.md`.

## Contributing

Pull requests welcome for:

- Additional lens variants (with focus + skip lists)
- Refinements to the severity rubric (effort buckets, quick-win tag rules)
- Refinements to the protocol (new cross-file checks worth standardizing)
- Examples of reports for differently-shaped skills (single-file, plugin)

Things this skill **won't** accept:

- Auto-fix or auto-edit functionality
- Single-number grading
- "Best skill" template comparisons

## License

Apache License 2.0. See [LICENSE](LICENSE).

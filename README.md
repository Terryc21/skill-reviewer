# skill-reviewer

[![Version](https://img.shields.io/badge/version-0.3.0-blue.svg)](https://github.com/Terryc21/skill-reviewer/releases)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-orange.svg)](https://docs.anthropic.com/en/docs/claude-code/skills)
[![GitHub stars](https://img.shields.io/github/stars/Terryc21/skill-reviewer?style=social)](https://github.com/Terryc21/skill-reviewer/stargazers)

> **Based on Anthropic's [Claude Code Skills architecture](https://docs.anthropic.com/en/docs/claude-code/skills). A reviewer for the skills you write on that foundation.**

You built a skill. You want feedback. You ask a friend.

Your friend says: "Looks great! Maybe consider adding tests?"

That's not a review. That's a hug.

`skill-reviewer` is a Claude Code skill that reviews other skills the way you'd want them reviewed if shipping mattered. It reads every file, cites specific lines, and separates strengths from weaknesses. (You need both. Reviews that only list problems tell you what to change but not what to keep.) Each finding lands in its own card with severity, effort, and quick-win tag, so you can pick the low-cost-high-leverage fixes first. It refuses to hedge: if something is fragile, it'll say so. If something is unusually well-done, it'll say that too.

## A real fragment

This is the top of a real review (skill-reviewer reviewing the `unforget` skill). It's the kind of output you'll get on your own work:

```markdown
## TL;DR

`unforget` is a Claude Code skill that consolidates deferred work into one
structured UNFORGET.md per project. It's more disciplined than most skills
I've reviewed: thin SKILL.md → fat reference files, deterministic Python
helpers with prose fallbacks, backups before destructive operations, and the
verify-still-open recipe is a genuinely original idea worth generalizing.
The standout strength is structural; the standout weakness is that the
canonical example never demonstrates the project's signature feature.

Weakness clusters:
1. Tooling and parseability gaps — Compact preset has no canonical regex
2. Discoverability — natural-language trigger phrases buried in YAML
3. Test coverage — Surface 6 and prune_backups.py are explicitly untested
4. Duplicate sources of truth — closure pointer format lives in 3 places
5. Under-specified fallbacks — "fail-soft" and Python-missing triggers vague
```

Notice what's there: file paths, named gaps, specific patterns. Notice what's not: "great work overall," "minor improvements possible," "consider thinking about." The full report (preserved at [`skills/skill-reviewer/examples/sample-report-unforget.md`](skills/skill-reviewer/examples/sample-report-unforget.md)) continues with the strengths to keep, individual finding cards (each with Why / Fix / citation), and a free-form Patterns section synthesizing what crossed cards.

## Install

Two commands in any Claude Code session, **one at a time**. Paste both at once and the second gets eaten as arguments to the first. Ask me how I know:

```
/plugin marketplace add Terryc21/skill-reviewer
```

Wait for "Successfully added marketplace," then:

```
/plugin install skill-reviewer@skill-reviewer
```

Verify with `/skill-reviewer --version`. You should see three lines: version, install path, and the list of available lenses.

**How invocation works.** The plugin registers `/skill-reviewer` as a single skill activation. Claude parses everything after it as a subcommand string — `review <path>`, `summary <path>`, `lenses`, `detect <path>`, `--version`. The subcommands listed throughout this README are recognized argument shapes, not separately-registered slash commands. There is no per-subcommand autocomplete; typing `/skill-reviewer ` and pressing Tab won't enumerate `review` / `summary` / `lenses` for you. (Tab will offer `/skill-reviewer` itself once the plugin is installed.)

<details>
<summary><strong>Manual install if the plugin path isn't available</strong></summary>

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Terryc21/skill-reviewer ~/.claude/skills/skill-reviewer
```

Invoke as `/skill skill-reviewer <args>` (with the prefix). Update later with `cd ~/.claude/skills/skill-reviewer && git pull`.

**Caveat:** manual install registers skill-reviewer as a Skill rather than a Plugin. Subcommands work identically as arguments — `/skill skill-reviewer review <path>`, `/skill skill-reviewer summary <path>`, etc. The rest of this README uses the plugin-install form (`/skill-reviewer review <path>`); if you used the manual path, mentally prepend `/skill ` to every example.

</details>

## Your first review

Point it at a skill directory:

```
/skill-reviewer review /path/to/some-skill
```

Two to five minutes later you get a structured markdown report: a one-paragraph TL;DR, 5-10 strengths to keep, a stack of finding cards sorted by severity (each card is one finding with Why / Fix / citation / effort / quick-win tag), and a short Patterns section naming what crossed cards. Roughly 1500 words for a plugin-shape skill.

If you want a 5-minute sanity check instead of the full audit:

```
/skill-reviewer summary /path/to/some-skill
```

That gives you the headline verdict, 3-5 strengths, up to 5 finding cards, and a one-line next step. Roughly 500-800 words. Useful when you're deciding whether to commit to a full review or just spot-check.

## When you'd use this

- **You finished a skill** and want honest feedback before publishing it on GitHub or a marketplace.
- **You're auditing someone else's skill** and don't want to invent a review structure from scratch.
- **You're about to merge a big PR to a skill** and want a structured pre-merge audit.
- **You suspect your own review is too kind** (or too harsh) and want a second-opinion pass to challenge your top conclusions.

The skill is designed for the moment when you've stared at your own work for too long and can't see what's missing.

## Lenses (focused reviews)

A lens narrows the audit to one concern. Use them when you don't need a full audit. You just want to ask one specific question.

```
/skill-reviewer review /path/to/skill --lens=safety
```

| Lens | What it asks |
|---|---|
| `full` (default) | Everything below, plus the cross-file checks |
| `quick` | Just give me the headline: TL;DR + top 5 strengths + top 5 weaknesses |
| `discoverability` | Will users find this and trigger it correctly? Are activation phrases buried? |
| `safety` | Does this skill back up before destructive operations? Recover from failures? |
| `architecture` | Is the file structure coherent? Are there duplicate sources of truth? |
| `parseability` | Can downstream tools consume the skill's output reliably? |
| `tests` | What's tested, what's silently untested, what's explicitly untested? |

A full lens reference (focus list + skip list per lens) lives in [`skills/skill-reviewer/reference/lenses.md`](skills/skill-reviewer/reference/lenses.md).

## Second opinion (for higher stakes)

When you're about to publish something public, add `--second-opinion`:

```
/skill-reviewer review /path/to/skill --second-opinion
```

That runs the full review, then dispatches an independent reviewer to challenge the top 3 conclusions. The challenger reads the cited files themselves. (They can't just trust the draft's quotes.) The final report has a "Second-opinion reconciliation" section showing what changed between draft and final. Roughly doubles review time. Worth it before public releases; overkill for routine pre-commit checks.

## What it deliberately doesn't do

`skill-reviewer` is opinionated about what it won't be:

- **It doesn't auto-fix anything.** Reports go to you; you decide what to act on. (If you want an auto-fix tool, that's a different tool, and probably a more dangerous one.)
- **It doesn't give a single grade.** No "B+ skill" or "8/10." A multi-axis review tells you specifically what's good and what's not. A grade hides the dimensions.
- **It doesn't compare to a "best skill" template.** Skills have legitimate stylistic variation. Imposing uniformity is worse than the variation it removes.
- **It doesn't chain three reviewers.** `--second-opinion` is one challenger. Three reviewers don't add proportional value; if the first pair strongly disagrees, that's signal you need a human tiebreaker, not another LLM.

## How it works with sibling skills

- **[`unforget`](https://github.com/Terryc21/unforget)**: log deferred findings from a review as rows in your project's UNFORGET.md so they don't get lost between releases.
- **[`radar-suite`](https://github.com/Terryc21/radar-suite)**: if you're auditing a skill that wraps radar-suite output, the formats interoperate.
- **[`bug-echo`](https://github.com/Terryc21/bug-echo)**: after fixing one finding, sweep for similar patterns elsewhere in the same skill.

## Other skills by the same author

No direct integration with skill-reviewer, but worth knowing about if you're building skills:

- **[`prompter`](https://github.com/Terryc21/prompter)**: intercepts your prompts before Claude Code acts on them, evaluates whether rewriting would meaningfully improve them, and shows you the rewrite for approval. Over time it teaches you what makes prompts effective.
- **[`tutorial-creator`](https://github.com/Terryc21/tutorial-creator)**: generate annotated code-reading tutorials from your own codebase. Three surfaces (tutorial generation, vocabulary management, learning-state inspection). Useful for writing-to-learn or producing audience-facing technical content.
- **[`workflow-audit`](https://github.com/Terryc21/workflow-audit)**: multi-layer behavioral audit of SwiftUI user workflows. Finds dead ends, broken promises, missing empty/loading/error states, and other UX defects that grep-style audits miss.

## Self-review

The skill can audit itself, which is the kind of self-aware feature most tools avoid because it sometimes lands honestly:

```
/skill-reviewer review /Volumes/2 TB Drive/Coding/GitHub/skill-reviewer
```

A healthy self-review surfaces findings already named in `docs/DESIGN.md` § Open design questions (those are known gaps) plus possibly new MEDIUM/LOW findings. A new CRITICAL or HIGH finding *not* in known-gaps blocks the next release until it's resolved or moved into known-gaps with a target version. Run this after any meaningful edit to the reference files.

## Origin

`skill-reviewer` was extracted from a real Claude Code session: the user asked Claude to review the `unforget` skill, Claude produced a detailed candid review, the user asked for the same analysis as a reusable prompt, then asked to turn the prompt into a skill. Each step generated something useful; the final step turned implicit knowledge into a reusable tool. The original review is preserved at [`skills/skill-reviewer/examples/sample-report-unforget.md`](skills/skill-reviewer/examples/sample-report-unforget.md).

This skill is therefore a literal artifact of using Claude Code well.

## Contributing

Pull requests welcome for:

- Additional lens variants (with focus + skip lists per the spec in `docs/DESIGN.md`)
- Refinements to the severity rubric (effort buckets, quick-win tag rules)
- New cross-file checks worth standardizing
- Canonical examples for single-file and thin-index shapes (the only existing example is plugin-shape)

Things this skill **won't** accept:

- Auto-fix or auto-edit functionality
- Single-number grading
- "Best skill" template comparisons
- Three-or-more reviewer chains

`docs/DESIGN.md` has the full contributor guide, including the cross-file invariants you need to respect when editing reference files.

## Maturity

v0.3.1 (May 2026). Five releases in the v0.x line; the skill self-reviews after each meaningful edit and the v0.3.0 self-review surfaced findings that landed in v0.3.1.

**Validated shape.** Plugin-shape and thin-index skills authored by this author (`unforget`, `bug-echo`, `prompter`, `tutorial-creator`, `radar-suite`). The plugin-shape sample report at `skills/skill-reviewer/examples/sample-report-unforget.md` is the canonical reference.

**Less-tested shapes.** Single-file skills (no canonical example yet), skills with substantial helper scripts (no canonical example yet), non-Anthropic skills (no review history), multi-author repos.

**v1.0 gates.**

1. A canonical thin-index sample report exists alongside the plugin-shape one (currently only plugin-shape has a sample, which means the card-format target word budgets for thin-index/single-file are theoretical).
2. A decision is made on `Open design question #2` — whether `detect` emits machine-readable JSON for CI integration. v1.0 either ships the option or removes the question.
3. Two consecutive self-reviews complete with zero CRITICAL or HIGH findings unnamed in `docs/DESIGN.md § Open design questions`.

Open questions and v0.4+ candidates documented in `docs/DESIGN.md`.

## Support the work

If `skill-reviewer` has saved you from shipping a skill with quiet flaws, consider buying me a coffee. No pressure, no obligation, no paywall on the skill itself.

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/terryc21)

## License

Apache License 2.0. See [LICENSE](LICENSE).

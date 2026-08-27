# skill-reviewer

![Version](https://img.shields.io/github/v/tag/Terryc21/skill-reviewer?label=version) ![Last commit](https://img.shields.io/github/last-commit/Terryc21/skill-reviewer) ![Stars](https://img.shields.io/github/stars/Terryc21/skill-reviewer?style=flat) ![Issues](https://img.shields.io/github/issues/Terryc21/skill-reviewer) ![License](https://img.shields.io/github/license/Terryc21/skill-reviewer) ![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)

> **Based on Anthropic's [Claude Code Skills architecture](https://docs.anthropic.com/en/docs/claude-code/skills). A reviewer for the skills you write on that foundation.**

You built a skill. You want feedback. You ask a friend. Your friend says: "Looks great! Maybe consider adding tests?" That's not a review. That's a hug.

`skill-reviewer` reads every file, cites specific lines, and separates strengths from weaknesses with severity-rated finding cards. It refuses to hedge: if something is fragile, it'll say so; if something is unusually well-done, it'll say that too.

*~4 min read · scan the TL;DR if you only have 30 seconds*

## TL;DR

- **What:** A Claude Code skill that reviews other Claude Code skills with file:line citations, severity-rated finding cards, and an optional second-opinion challenger pass.
- **Why:** Polite-by-default feedback ships skills with quiet flaws. skill-reviewer gives you the candid review you'd want before publishing.
- **Install:** Two `/plugin` commands in Claude Code (below); then `/skill-reviewer` is available in any session.
- **Try first:** `/skill-reviewer summary /path/to/some-skill` — 5-minute sanity check (TL;DR + top 5 strengths + top 5 weaknesses). Lighter than a full review.
- **Example output:** [a real review of the `unforget` skill](skills/skill-reviewer/examples/sample-report-unforget.md).
- **Maturity:** v0.3.0; five releases so far; the skill reviews itself after every meaningful edit.

## Newer to Claude Code?

A **skill** is a markdown file Claude Code knows how to run. When you type `/skill-reviewer review <path>`, Claude follows the instructions in this skill, reads the target skill's files, and writes you a structured report. You don't have to memorize anything — the skill tells Claude what to do, you read the report.

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

<details>
<summary><strong>How invocation works (autocomplete caveat)</strong></summary>

The plugin registers `/skill-reviewer` as a single skill activation. Claude parses everything after it as a subcommand string — `review <path>`, `summary <path>`, `lenses`, `detect <path>`, `--version`. The subcommands listed throughout this README are recognized argument shapes, not separately-registered slash commands. There is no per-subcommand autocomplete; typing `/skill-reviewer ` and pressing Tab won't enumerate `review` / `summary` / `lenses` for you. (Tab will offer `/skill-reviewer` itself once the plugin is installed.)

</details>

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

## Scoping a run

skill-reviewer scopes by **the directory path you pass**. There's no fresh-vs-resume mode — every review is a fresh read of the skill's files at the path. Prior reports live in `.agents/research/` and can be cited by hand, but the skill doesn't auto-load them.

| Goal | Command |
|---|---|
| Review one whole skill | `/skill-reviewer review /path/to/some-skill` |
| Spot-check before committing to a full review | `/skill-reviewer summary /path/to/some-skill` |
| Focus on one concern | `/skill-reviewer review /path/to/some-skill --lens=safety` |
| Higher-stakes pre-publish review | `/skill-reviewer review /path/to/some-skill --second-opinion` |
| Just classify the skill's shape and recommended depth | `/skill-reviewer detect /path/to/some-skill` |

The skill always reads the live files at the path you give it. If you want a "diff against last time" view, compare two reports manually — that's a deliberate choice, not a missing feature. Re-running fresh keeps each review independent of prior verdicts that may now be stale.

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

## Origin

`skill-reviewer` was extracted from a real Claude Code session: the user asked Claude to review the `unforget` skill, Claude produced a detailed candid review, the user asked for the same analysis as a reusable prompt, then asked to turn the prompt into a skill. Each step generated something useful; the final step turned implicit knowledge into a reusable tool. The original review is preserved at [`skills/skill-reviewer/examples/sample-report-unforget.md`](skills/skill-reviewer/examples/sample-report-unforget.md).

This skill is therefore a literal artifact of using Claude Code well.

## Contributing

Pull requests welcome for:

- Additional lens variants (with focus + skip lists per the spec in `docs/DESIGN.md`)
- Refinements to the severity rubric (effort buckets, quick-win tag rules)
- New cross-file checks worth standardizing
- Example reviews of skills built as a single file, or as a short index (the only example so far is a plugin)

Things this skill **won't** accept:

- Auto-fix or auto-edit functionality
- Single-number grading
- "Best skill" template comparisons
- Three-or-more reviewer chains

`docs/DESIGN.md` has the full contributor guide, including the cross-file invariants you need to respect when editing reference files.

## Where it stands

v0.3.0 (May 2026). Five releases so far; the skill reviews itself after each meaningful edit.

**Well tested on.** Skills built as plugins, and skills whose main file is a short index pointing at other files — my own (`unforget`, `bug-echo`, `prompter`, `tutorial-creator`, `radar-suite`). The sample report at `skills/skill-reviewer/examples/sample-report-unforget.md` is the reference example.

**Less tested on.** Skills that are one file, and skills with a lot of helper scripts — no example of either yet. Skills by other authors, and repos with several authors, have no review history at all.

**Self-review** — run `/skill-reviewer review /path/to/skill-reviewer` against this repo after meaningful edits. **v1.0 gates and open design questions** — see [`docs/DESIGN.md`](docs/DESIGN.md).

## Related skills

[**bug-echo**](https://github.com/Terryc21/bug-echo) — find the same bug elsewhere after a fix ·
[**bug-prospector**](https://github.com/Terryc21/bug-prospector) — hunt for bugs before a release ·
[**workflow-audit**](https://github.com/Terryc21/workflow-audit) — trace SwiftUI behaviour ·
[**unforget**](https://github.com/Terryc21/unforget) — one file for everything you deferred ·
[**radar-suite**](https://github.com/Terryc21/radar-suite) — six skills tracing user paths ·
[**prompter**](https://github.com/Terryc21/prompter) — rewrite prompts before running them ·
[**tutorial-creator**](https://github.com/Terryc21/tutorial-creator) — lessons from your own code

## Author

Terry Nyberg, [Coffee & Code LLC](https://stuffolio.app/). If `skill-reviewer` has saved you from shipping a skill with quiet flaws, [a coffee](https://buymeacoffee.com/stuffolio) is appreciated. Issue reports about what worked or didn't are more useful.

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/stuffolio)

## License

Apache License 2.0. See [LICENSE](LICENSE).

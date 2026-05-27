# Testing `init2`

How this skill is verified. Methodology, recorded results, and how to reproduce — so future edits to `SKILL.md` are gated on the same kind of evidence.

## Why test a documentation skill

A skill is a prompt that future agents read. If you don't watch an agent *fail without it* and then *succeed with it*, you don't know whether the skill teaches the right thing. Documentation can be plausible without being load-bearing — testing tells you which.

We apply the **RED-GREEN-REFACTOR** cycle from TDD to skills (per [`superpowers:writing-skills`](https://github.com/anthropics/superpowers)):

| TDD | Skill testing |
|---|---|
| Write failing test first | Run baseline subagent **without** the skill |
| Watch it fail | Document what the baseline naturally gets wrong |
| Write minimal code | Write the skill addressing those specific failures |
| Watch it pass | Run a fresh subagent **with** the skill, verify compliance |
| Refactor | Close any new rationalizations |

**Iron law:** no skill edit without a failing test first. Same applies to edits.

## Methodology — paired subagent runs

1. Pick a representative target repo. For `init2`, that's a real codebase with no existing `CLAUDE.md` / `AGENTS.md` (or one where we tell the agents to ignore the existing files).
2. Write a **sealed prompt** for the baseline agent: describes the task naturally, no mention of the skill.
3. Write a **sealed prompt** for the skill agent: same task, plus the full `SKILL.md` content embedded inline (subagents are isolated and can't load files from disk).
4. Dispatch both in parallel, writing outputs to separate `/tmp/` directories.
5. Diff the outputs structurally — section headings, line counts, content alignment.
6. Compare agent self-reports for rationalizations and ambiguities.
7. Patch the skill to address whatever the baseline broke and the skill agent didn't catch.

**Critical:** seal the prompts. If you brief the baseline agent in a way that hints at the skill's content, you're testing the prompt, not the skill.

## v0.2 verification results

**Target:** `/Users/bo/Repos/andybowu/Burnbar` — a docs-only Swift macOS repo (README + ROADMAP + `docs/PLAN.md` + `docs/data-sources.md`; no source code yet).

**Date:** 2026-05-27 · **Runner:** Claude Opus 4.7 (1M context), parallel `Agent` dispatch.

| Metric | Baseline (no skill) | With skill |
|---|---|---|
| Tool calls | 12 | 10 |
| Wall time | 136 s | 92 s |
| `CLAUDE.md` length | 108 lines | 77 lines |
| `AGENTS.md` length | 108 lines | 79 lines |
| **Matching section headings** | **0 / 9** | **7 / 8** |
| Build commands invented | 0 | 0 |
| Generic dev advice bloat | none | none |

**The headline number is 0/9 vs. 7/8.** The baseline agent produced two 108-line files that covered the same project but with **zero shared section headings** — one read as narrative ("Things that will surprise you", "How to orient yourself in 60 seconds"), the other as contract ("Hard rules", "Definition of done for an agent change"). The with-skill agent produced two near-identical files; only the final tool-specific notes section differed.

Both runs handled the "commands don't exist yet" case honestly — neither invented a `Makefile`, neither omitted the planned commands. That failure mode is *not* what the skill prevented.

**The skill's real value-add is structural alignment.** That's now reflected in `SKILL.md` v0.2 as an explicit rule.

### Gaps surfaced (folded into v0.2)

1. The skill said "files describe the same project" but never required **matching section headings** — the actual drift vector. → Added explicit structural-alignment rule + a `diff CLAUDE.md AGENTS.md` verify step.
2. "Sized to the project" was ambiguous for **docs-only / pre-implementation repos** (no real build commands exist). The skill agent flagged this directly. → Added guidance: list planned commands from `PLAN.md`, mark them as not-yet-runnable, don't invent and don't omit.

## Reproducing the test

### Baseline agent prompt (template)

```
You are helping bootstrap multi-agent documentation for a repository. This is
a real task — execute it the way you naturally would, without trying to
perform for a grader.

Target repo: <PATH>
(describe the repo briefly: language, current state, what files exist)

Task: Generate two guidance files for AI coding agents:

1. CLAUDE.md — for Claude Code (https://claude.ai/code)
2. AGENTS.md — for OpenAI Codex CLI (follows the agents.md convention)

If you find an existing CLAUDE.md or AGENTS.md, IGNORE its content and
generate fresh from primary sources.

Output paths:
- /tmp/spellbook-test/baseline/CLAUDE.md
- /tmp/spellbook-test/baseline/AGENTS.md

Constraints: read-only on the target repo; all writes go to /tmp/.

When done, report in <300 words: which files you read, how you structured
each output, anything hard to decide, build/test commands you included and
how you verified they're real, and any differences between your two files.
```

### With-skill agent prompt (template)

Same as the baseline prompt, but **prepend** the full current `SKILL.md` content between `---SKILL START---` / `---SKILL END---` markers, and change the output paths to `/tmp/spellbook-test/with-skill/`. Also ask explicitly: "Confirm: one scan or two? If you re-read any source file to write the second output, say which and why."

### Interpreting results

Diff the two pairs:

```bash
diff -u /tmp/spellbook-test/baseline/CLAUDE.md /tmp/spellbook-test/baseline/AGENTS.md | wc -l
diff -u /tmp/spellbook-test/with-skill/CLAUDE.md /tmp/spellbook-test/with-skill/AGENTS.md | wc -l
```

The with-skill diff should be **dramatically shorter** — ideally only the header lines, the final tool-specific notes section, and a handful of single-word tonal swaps.

```bash
grep -c '^##' /tmp/spellbook-test/baseline/*.md
grep -c '^##' /tmp/spellbook-test/with-skill/*.md
```

The two with-skill files should have the same heading count and (when listed) the same heading text except the final one.

## When to re-run

- Before bumping `SKILL.md` from `v0.x` → `v1.0`
- After any edit to `SKILL.md` larger than a typo fix
- When adopting the skill in a context the prior testing didn't cover (e.g. a polyglot monorepo, a repo with extensive existing CLAUDE.md content, a non-English codebase)

Record the new metrics in the table above. Don't delete prior rows — version history is the test trail.

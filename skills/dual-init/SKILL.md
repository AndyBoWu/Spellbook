---
name: dual-init
description: Use when a codebase needs documentation for both Claude Code (CLAUDE.md) and OpenAI Codex (AGENTS.md) — typically on first contact with a new repo, or when adding Codex support to an existing Claude-only project. Generates both files from a single codebase scan so they stay in sync.
---

# dual-init

## Overview

Replaces a single-target `/init` workflow with a dual-target one. The codebase is scanned **once**, then two guidance files are written:

- `CLAUDE.md` — for Claude Code
- `AGENTS.md` — for OpenAI Codex (and any other runtime that reads the [agents.md](https://agents.md) convention)

Both files describe the same project, but each is tuned to its reader. Writing them together (vs. running `/init` twice with different prompts) keeps them factually aligned — drift between them is the most common failure mode in multi-agent repos.

## When to Use

- First touch with a new repo that has no `CLAUDE.md` or `AGENTS.md`
- Adding Codex support to a repo that already has `CLAUDE.md`
- Adding Claude support to a repo that already has `AGENTS.md`
- A repo has both files but they've drifted out of sync after refactors

**Don't use for:**
- Repos that only use one agent runtime — the built-in `/init` is fine
- Updating one file in isolation — edit the file directly, then re-run this skill if drift becomes a problem

## Workflow

### 1. Scan the codebase **once**

Read the inputs that would normally drive `/init`:

- `README.md` (project purpose, install/build/test, public surface)
- `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` / `Gemfile` / `*.xcodeproj` — whatever declares language, deps, scripts
- `Makefile` / `Scripts/` / `bin/` — build/lint/test entry points
- `.cursor/rules/`, `.cursorrules`, `.github/copilot-instructions.md` — pre-existing agent guidance worth carrying over
- `ROADMAP.md`, `docs/PLAN.md`, `ARCHITECTURE.md` — load-bearing decisions and constraints
- Top-level directory structure (one `ls`, not a recursive crawl)
- An existing `CLAUDE.md` or `AGENTS.md` if present (don't overwrite — propose a diff)

**One scan, two writes.** If you find yourself re-reading files for the second file, you're doing it wrong.

### 2. Identify load-bearing facts

These go into **both** files verbatim or near-verbatim:

- Project purpose (one or two sentences)
- Build / lint / test / run commands (exact strings, including how to run a single test)
- Architecture facts that require reading multiple files to understand
- Hard constraints (privacy theses, "never do X", scope caps, language requirements)
- Source-of-truth docs to defer to (PLAN.md, ARCHITECTURE.md, etc.)

If a fact only matters for one agent, it goes in that file only — but those should be rare. Most facts are tool-agnostic.

### 3. Write `CLAUDE.md`

Use the standard `/init` output format. The first three lines are fixed:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
```

Then the content sections, sized to the project. **Do not** include generic dev advice ("write tests", "don't commit secrets"). **Do** include the load-bearing facts from step 2.

### 4. Write `AGENTS.md`

Same content, but reframed for Codex conventions:

```markdown
# AGENTS.md

This file provides guidance to OpenAI Codex (and other agents reading the agents.md convention) when working with code in this repository.
```

**Tonal differences from CLAUDE.md** (small, intentional):

- AGENTS.md tends slightly more directive ("Run X before committing") vs. narrative ("X is run before committing")
- AGENTS.md omits Claude-Code-specific tooling references (skills, the `Skill` tool, `TaskCreate`, etc.)
- AGENTS.md may include a brief "Codex notes" section if Codex has known quirks in this repo (e.g., a particular sandbox policy, a non-default working directory expectation)

**What stays identical:**
- Project purpose
- Build/lint/test commands
- Architecture summary
- Load-bearing constraints
- Source-of-truth pointers

If you find yourself rewriting more than ~20% of the content between the two files, you're probably over-customizing. The whole point is they describe the same project.

### 5. Verify

- Both files exist at the repo root
- Both start with their correct header
- Build/lint/test commands match byte-for-byte across the two
- No facts contradict each other
- Neither file invents commands or files that don't exist (verify with `ls` / quick `grep` before claiming a script is present)

### 6. Report

One short summary: what you wrote, where the facts came from, what you chose to omit. Don't paste the file contents back at the user — they can read them.

## Quick Reference

| Section | CLAUDE.md | AGENTS.md |
|---|---|---|
| Header | `# CLAUDE.md` + Claude Code reference line | `# AGENTS.md` + Codex/agents.md reference line |
| Project purpose | ✅ identical | ✅ identical |
| Build/lint/test commands | ✅ identical, exact strings | ✅ identical, exact strings |
| Architecture | ✅ identical | ✅ identical |
| Load-bearing constraints | ✅ identical | ✅ identical |
| Source-of-truth pointers | ✅ identical | ✅ identical |
| Tone | Narrative-ok | Slightly more directive |
| Tool-specific notes | Claude Code skills, TaskCreate, etc. (if relevant) | Codex sandbox / approval-mode notes (if relevant) |

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Running `/init` twice (once per file) | Scans the codebase twice, files drift on second pass | One scan, two writes |
| Pasting full file contents in the summary | Wastes user's context, they can `cat` the file | One-paragraph summary only |
| Inventing build commands not in the repo | Future agents will run them and fail | Verify every command against actual scripts/Makefile/package.json before writing it |
| Diverging architecture summaries between the two files | Defeats the whole point of dual-init | Write architecture once, paste into both |
| Overwriting an existing CLAUDE.md or AGENTS.md without asking | Destroys curated content | If a file exists, propose a diff and confirm before overwriting |
| Including generic dev advice ("write tests", "use semantic commits") | Bloat that's not project-specific | Cut it. CLAUDE.md/AGENTS.md is for *this* repo, not for general best practices |

## Status

**v0.1** — generates both files from a single scan, with the tonal split above. Formal TDD-style verification (baseline subagent run → skill → re-run) pending before declaring v1.0.

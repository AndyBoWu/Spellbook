# `init2` v1.0 — design

**Status.** Draft, awaiting user review. Implementation begins after sign-off.
**Owner.** Andy Wu.
**Date.** 2026-05-27.

## Why this exists

Most repos that need agent-facing docs land in one of two states:

1. **One file exists** — `CLAUDE.md` xor `AGENTS.md` is present. The other agent runtime has no guidance.
2. **Both files exist, but neither is a SoT** — the two files disagree, or each is an unstructured pile of notes. Agents can't trust either as canonical.

These are the cases `init2` exists to solve. Empty-repo bootstrapping (case 3) is a bonus capability — same template, fewer inputs.

The v1.0 stance: **`init2` owns the structure.** There is one canonical template; all three cases write into it. Existing content is preserved as facts, but the skeleton is replaced. The user gets to approve the change via a unified diff before any file is written.

## Goal

Promote `init2` from v0.2 to v1.0 by verifying the canonical-template approach against all three cases with the same RED-GREEN methodology as v0.2.

| Case | Description | Fixture | Status |
|---|---|---|---|
| 1 | One file exists (CLAUDE.md xor AGENTS.md) | Burnbar current state — has `CLAUDE.md`, no `AGENTS.md` | 🎯 v1.0 |
| 2 | Both files exist, drifted | Hand-crafted drifted pair at `skills/init2/test-fixtures/case2-drift/` — same project, incompatible section structures, conflicting commands | 🎯 v1.0 |
| 3 | Empty repo | Fresh fixture committed at `skills/init2/test-fixtures/case3-empty/` (mirrors Burnbar's pre-CLAUDE shape: README + ROADMAP + docs/, no agent files) | 🎯 v1.0 (re-verify after refactor) |

The symmetric variant of case 1 (`AGENTS.md` exists, `CLAUDE.md` does not) is covered by construction — same code path with the roles swapped. If a real instance appears later, run the RED-GREEN gate on it and add a row to TESTING.md.

Out of scope for v1.0: real-source repos, polyglot / monorepo, drift detection beyond the in-skill reconciliation (i.e., the skill doesn't watch for drift over time), per-package or workspace-aware files.

## The canonical template

Six sections, in this order. This is the SoT contract for v1.0+:

| # | Section | Purpose |
|---|---|---|
| 1 | **Overview** | Project purpose, 1–3 sentences. What and why. |
| 2 | **Commands** | Build / lint / test / run, exact strings. Tool-agnostic. |
| 3 | **Architecture** | How the system fits together. Modules, data flow, key decisions. |
| 4 | **Load-bearing constraints** | Hard rules. "Never do X." Privacy/scope/language constraints. |
| 5 | **Source of truth** | Pointers to `PLAN.md` / `ROADMAP.md` / `ARCHITECTURE.md` etc. |
| 6 | **`<Agent>` notes** | The only section that differs between CLAUDE.md and AGENTS.md (Claude-Code-specific tooling for one, Codex sandbox quirks for the other). |

Section 6 is also the catch-all: any content from an existing file that doesn't map cleanly into 1–5 lands here. Future versions may add sections (non-breaking); reordering or removing is a v2.0 move.

## Architecture

Three input states converge on the canonical template through a single write path:

```
scan repo + detect existing files
│
├── empty                    ┐
│                            │
├── one file exists          ├──→ build canonical content from
│   (CLAUDE.md xor AGENTS.md)│    (a) existing-file facts (precedence)
│                            │    (b) codebase scan (augmentation)
└── both files exist         ┘    (c) template fallbacks (last resort)
                                       │
                                       ▼
                              propose unified diff to user
                                       │
                                       ▼
                              user approves? — write both files
```

### Behavior in each case

**Case 1 (one file exists).** Extract facts from the existing file. Restructure them into the canonical template. Generate the missing file using the same canonical structure. Both files (the existing one, now reorganized; and the new one) are written.

**Case 2 (both files exist, drifted).** Extract facts from both files. Reconcile conflicts (see precedence rules below). Restructure into the canonical template. Both files are rewritten.

**Case 3 (empty repo).** Run the codebase scan. Populate the canonical template from scan results. For sections the scan can't fill (e.g., constraints in a docs-only repo), use template placeholder language explicitly marked as such — never invent.

### Precedence rules (which fact wins when sources disagree)

When the same fact appears in multiple sources with different values:

1. **Code beats docs.** A build command in `package.json`/`Makefile`/etc. overrides a different command claimed in `CLAUDE.md` or `AGENTS.md`. The doc was stale.
2. **Existing-file consensus beats single source.** If `CLAUDE.md` and `AGENTS.md` agree on a fact and code doesn't say otherwise, that's the canonical value.
3. **Conflicts between existing files with no code arbitration.** Stop and report — do not silently pick a side. The user resolves.

These rules go in the SKILL.md verbatim.

### Safety: diff + confirm

Before writing anything in cases 1 and 2 (where existing files would change), the skill emits a unified diff of the proposed `CLAUDE.md` and `AGENTS.md` against their current state and asks the user to confirm. No silent overwrites. This works regardless of git state — the user can run the skill mid-development without needing a clean tree.

In case 3 (empty repo), there's nothing to diff against; the skill still summarizes what it's about to write but doesn't block on confirmation.

## SKILL.md edits

Substantial rewrite. The current sections need to be reshaped around the three cases, not the v0.2 linear flow.

**Overview — rewrite.** State the SoT mission: init2 owns the structure, three input cases, one template, diff+confirm before write.

**When to Use — refresh.** Replace the four bullets with case 1, case 2, case 3 framings. The "Don't use for" block stays roughly the same.

**§1 Scan — add case detection.** Before the codebase scan, list the repo root, record whether `CLAUDE.md` and `AGENTS.md` exist, and branch into one of the three cases.

**§2 Load-bearing facts — add precedence rules.** The three rules above, verbatim.

**§3 Build canonical content — new section, replaces v0.2 §3 + §4.** For each of the six canonical sections, gather content from sources in precedence order. Map existing-file content into the right canonical section; never drop content silently — what doesn't fit goes in `<Agent> notes`.

**§4 Propose diff and confirm — new section.** Generate the two file contents in memory. For cases 1 and 2, emit a unified diff against the existing files and require explicit user confirmation before writing. For case 3, summarize and write.

**§5 Verify — strengthen.** Add: heading lists from both files must match the canonical template's six sections exactly (last one is `<Agent> notes` variant). `grep '^##' CLAUDE.md AGENTS.md` must show the same first five headings and the agent-specific sixth.

**§6 Report — record case + diff status.** "Built canonical files for case 2 (drift reconciliation); diff approved 2026-MM-DD" or equivalent.

**Common Mistakes — replace some rows.** Drop the v0.2 mistakes that no longer apply ("running /init twice"). Add:
- "Restructured existing files without showing a diff" → always emit diff and pause for confirmation.
- "Silently dropped content from an existing file that didn't fit a canonical section" → map every non-empty paragraph somewhere; use `<Agent> notes` as catch-all.
- "Picked a side on a conflicting fact across two existing files" → stop and report; the user resolves.

**Status section — bump and replace.** v0.2 paragraph stays as history. New v1.0 paragraph summarizes the three-case verification.

## Test plan

Same RED-GREEN paired-subagent methodology as v0.2. Three cases, three paired runs, six subagents total.

### Case 1 — one file exists (CLAUDE.md only)

**Fixture.** Burnbar current state on 2026-05-27: existing `CLAUDE.md` (6351 bytes), no `AGENTS.md`, no source code.

**Baseline prompt.** Sealed; no mention of the skill. "This repo has a CLAUDE.md. Generate an AGENTS.md describing the same project."

**With-skill prompt.** Same task plus full `SKILL.md` content embedded. Plus the explicit instruction: "Apply the canonical template to both files; show the diff for CLAUDE.md before any write."

**Pass criteria** (all four hold for with-skill):
1. Both output files have the six canonical section headings, in order, with section 6 named `Claude Code notes` / `Codex notes` respectively.
2. `grep '^##' CLAUDE.md AGENTS.md` matches on sections 1–5; section 6 differs as expected.
3. Build/lint/test/run commands are byte-identical between the two files.
4. No invented files, scripts, or commands.

### Case 2 — both files exist, drifted

**Fixture.** Hand-crafted drifted pair committed at `skills/init2/test-fixtures/case2-drift/` — same project (Burnbar shape), but the two files use deliberately incompatible section structures (e.g., CLAUDE.md uses narrative headings, AGENTS.md uses contract-style headings) and disagree on at least one build/lint/test command. Stored in-repo so the fixture is stable and reproducible across runs. (The v0.2 baseline /tmp output is gone; we don't depend on it.)

**Baseline prompt.** "This repo has CLAUDE.md and AGENTS.md. They're out of sync. Reconcile them."

**With-skill prompt.** Same task plus `SKILL.md`. "Apply the canonical template; show the diff for both files before writing."

**Pass criteria.** Same four as case 1, plus:
5. No fact is silently dropped between input and output (verify by spot-checking the diff for content that's in the input but missing from the output).

### Case 3 — empty repo

**Fixture.** `skills/init2/test-fixtures/case3-empty/` — only `README.md`, `ROADMAP.md`, `docs/PLAN.md`, `docs/data-sources.md`. No `CLAUDE.md`, no `AGENTS.md`. (Mirrors the v0.2 Burnbar starting state, but committed in-repo so it's stable across runs.)

**Baseline prompt.** v0.2 baseline prompt, unchanged.

**With-skill prompt.** v0.2 with-skill prompt with one addition: "Use the canonical six-section template from §3 of the embedded SKILL.md exactly."

**Pass criteria.** Same four as case 1. Comparing against the v0.2 with-skill run (7/8 heading match), v1.0 should be 6/6 (full canonical match).

### Promotion gate

All three cases pass their criteria for the with-skill run AND fail ≥1 criterion in the baseline run. Mirror-mode tests where baseline accidentally meets all criteria are inconclusive — harden the baseline or revisit scope.

### Metrics in TESTING.md

Replace the v0.2 single-row table with a four-row table (v0.2 greenfield + three v1.0 cases) keeping a `case` and `mode` column. Don't delete the v0.2 row.

## Promotion criteria

**Promotion gate.** Both:
1. All three cases pass their full pass-criteria sets on the with-skill run.
2. Each case shows the baseline failing ≥1 criterion that the skill specifically addresses (heading restructure, drift reconciliation, or template completeness — not generic agent failures like inventing commands).

**Ship list when gate passes** (single PR):
- `skills/init2/SKILL.md` — rewrite per the SKILL.md edits section above; Status bumped to v1.0 with the new three-case results.
- `skills/init2/TESTING.md` — four-row metrics table; three new prompt templates for the case-specific tests; reproducibility instructions updated.
- No README changes — Spellbook README doesn't track per-skill versions.

**What v1.0 promises** (the contract this locks):
- Three input cases (empty, one-file-exists, both-files-exist) all produce files matching the canonical template.
- The canonical template's six sections are stable in v1.x. Adding sections is non-breaking; reordering or removing is v2.0.
- Frontmatter shape (`name`, `description`) is stable in v1.x.
- Diff + confirm is the default safety bar for any run that would modify existing files. Removing this is v2.0.

**What v1.0 explicitly does not promise** (deferred):
- Real-source-repo coverage (queued for v1.1).
- Polyglot / monorepo coverage.
- Per-package or workspace-aware files.
- Drift detection over time (the skill reconciles drift at run time; it doesn't watch for it).

**When to cut v1.1+.** New repo-shape coverage that passes the same RED-GREEN gate. The shape and its test land together — no "add the feature now, test later."

**When to cut v2.0.** Breaking the canonical template (reorder / remove sections), breaking frontmatter, or weakening the diff+confirm safety bar.

# `init2` v1.0 — design

**Status.** Draft, awaiting user review. Implementation begins after sign-off.
**Owner.** Andy Wu.
**Date.** 2026-05-27.

## Goal

Promote `init2` from v0.2 (one verified shape) to v1.0 (two verified shapes), and lock the public contract so v1.x is non-breaking.

v1.0 = **battle-tested**: paired-subagent verification on the two repo shapes below, using the same RED-GREEN methodology as v0.2.

| Shape | State | Status |
|---|---|---|
| Docs-only, greenfield | No `CLAUDE.md`, no `AGENTS.md` | ✅ Verified in v0.2 (Burnbar, pre-CLAUDE.md) |
| Docs-only, existing CLAUDE.md | `CLAUDE.md` present, no `AGENTS.md` | 🎯 v1.0 target (Burnbar, current state) |

The symmetric shape — existing `AGENTS.md`, no `CLAUDE.md` — is covered by construction (same code path in `§4`), not by a separate test. If a real instance of the symmetric shape appears later, run the same RED-GREEN gate against it and add a row to TESTING.md.

Out of scope for v1.0 (deferred): real-source repos, polyglot / monorepo, both-files-drifted, drift detection, per-package files.

## Architecture change

Today's flow is linear: scan → write `CLAUDE.md` → write `AGENTS.md` → verify. v1.0 introduces a mode branch driven by what the scan finds at the repo root:

```
scan codebase + detect existing files
├── greenfield (no CLAUDE.md, no AGENTS.md)
│     → write both using skill's default skeleton  (v0.2 path, unchanged)
│
└── mirror mode (CLAUDE.md xor AGENTS.md present)
      → extract skeleton from existing file
      → write only the missing file, mirroring extracted skeleton
      → never modify the existing file
```

Both-files-exist is explicitly not handled in v1.0. If the skill encounters it, it reports "both files present — out of scope for this version" and exits without writing.

### Two new behaviors

1. **Skeleton extraction.** Parse the existing file's `##` headings in document order. That ordered list becomes the canonical skeleton for this run, overriding the skill's default.
2. **Non-destructive default.** The existing file is treated as ground truth and never edited in v1.0. No diff proposal, no drift flag — read-only.

### Approach rationale (Approach A: mirror existing)

Two alternatives were considered and rejected:

- **Force canonical skeleton + propose diff to existing file.** Too pushy for a personal-grimoire skill; users who curated their CLAUDE.md will reject the diff and end up with mismatched files anyway.
- **Two-pass with explicit pick.** Adds a decision prompt up front. More correct in principle, but slows the common path. Revisit if v1.x users report wanting it.

Mirror mode respects the user's prior curation, matches the skill's existing "propose a diff before overwriting" posture, and produces zero-churn output on the common case.

## SKILL.md edits

In order of appearance in the file.

**§1 Scan — add detection.** Before the codebase scan, list the repo root and record whether `CLAUDE.md` and `AGENTS.md` exist. This selects the write mode.

**§2 Identify load-bearing facts — add precedence rule.** When an existing file is present, its facts are the ground truth. The codebase scan augments but does not contradict. If a fact in the existing file disagrees with the source (e.g., build command in `CLAUDE.md` not present in `package.json`), stop and report — do not silently rewrite.

**§3 Write CLAUDE.md — gate on mode.**
- *Greenfield (no CLAUDE.md):* current text, unchanged.
- *Mirror mode (CLAUDE.md exists):* skip this step entirely. Proceed to §4.

**§4 Write AGENTS.md — gate on mode.**
- *Greenfield:* current text, default skeleton.
- *Mirror mode (CLAUDE.md exists, AGENTS.md does not):* extract `##` headings from the existing `CLAUDE.md` in document order. Use that exact list as the AGENTS.md skeleton. Per-section, copy facts from `CLAUDE.md` where they are tool-agnostic; apply the existing tonal swaps for Codex (slight directive shift, drop Claude-Code-specific tooling references).
- *Symmetric mirror mode (AGENTS.md exists, CLAUDE.md does not):* same logic, reversed targets.

**§5 Verify — strengthen.** Add: in mirror mode, `grep '^##' CLAUDE.md AGENTS.md` must yield identical heading text in identical order, except the final `<Agent> notes` section. Any mismatch means re-extract.

**§6 Report — record mode used.** Single line in the summary: `Wrote AGENTS.md in mirror mode (skeleton from existing CLAUDE.md)` or `Wrote both files in greenfield mode` or the symmetric variant.

**Common Mistakes table — add row.**

| Wrote the new file using the default skeleton when the existing file used a different one | Defeats the mirror-mode contract; produces drift-on-arrival | Extract skeleton from the existing file first; use it verbatim |

**Status section — bump.** From v0.2 to v1.0 once the test below passes. Append a paragraph with the mirror-mode result alongside the v0.2 greenfield paragraph (don't replace — version history is the test trail).

## Test plan

Same RED-GREEN methodology as v0.2: paired subagent runs, sealed prompts, diff outputs.

**Target fixture.** Burnbar's current state on 2026-05-27: `README.md` + `ROADMAP.md` + `docs/PLAN.md` + `docs/data-sources.md` + existing `CLAUDE.md` (6351 bytes, from v0.2). No `AGENTS.md`. No source code. The existing `CLAUDE.md` is the fixture; do not regenerate it before the test.

**Baseline prompt** (sealed; no mention of the skill). Tells the subagent: this repo has a `CLAUDE.md`; generate an `AGENTS.md` for Codex describing the same project. Output to `/tmp/spellbook-test/v1-baseline/AGENTS.md`. Copy `CLAUDE.md` to the same directory post-run for byte-comparison.

**With-skill prompt.** Same task plus full `SKILL.md` content embedded between `---SKILL START---` / `---SKILL END---` markers. Output to `/tmp/spellbook-test/v1-skill/AGENTS.md`. Same `CLAUDE.md` copy.

**Pass criteria for v1.0 promotion** (all four must hold for the with-skill run):

1. `grep '^##' CLAUDE.md AGENTS.md` shows identical heading text in identical order, except the final `<Agent> notes` section.
2. Original Burnbar `CLAUDE.md` is byte-identical before and after the run (skill did not modify it).
3. Any build/lint/test commands present in both files match byte-for-byte.
4. `AGENTS.md` does not invent files, scripts, or commands not in the source repo or in `CLAUDE.md`.

**Skill is load-bearing if** the baseline run fails criterion #1 (heading reorganization) or #2 (baseline edited CLAUDE.md). Those are the two failures the mirror-mode contract specifically prevents. Baseline failures only on #3 or #4 alone don't count — those are generic agent failures, not what mirror mode addresses. If the baseline doesn't fail #1 or #2, the test is inconclusive: harden the baseline (e.g., a CLAUDE.md with deliberately unusual section ordering that invites reorganization) or reconsider whether mirror mode is solving a real problem on this fixture.

**Metrics to record in TESTING.md** — new row in the existing v0.2 table, plus a new `mode` column distinguishing greenfield from mirror runs. Don't delete the v0.2 row.

## Promotion criteria

**Promotion gate.** Both conditions hold:
1. With-skill run passes all four criteria above.
2. Baseline run fails ≥1 of them.

If either condition misses, v0.2 remains current. Revise the skill (or the test) before retrying.

**Ship list when the gate passes** (single PR):
- `skills/init2/SKILL.md` — edits from the SKILL.md section above; Status bumped to v1.0.
- `skills/init2/TESTING.md` — new row in the metrics table; mirror-mode prompt templates added under "Reproducing the test".
- No README changes — Spellbook README doesn't track per-skill versions.

**What v1.0 promises** (the contract this locks):
- Two modes — greenfield and one-file-exists mirror — work end-to-end on the verified fixtures.
- The default skeleton used in greenfield mode is stable for v1.x. Adding sections is non-breaking; reordering or removing is a v2.0 move.
- Frontmatter shape (`name`, `description`) is stable for v1.x.

**What v1.0 explicitly does not promise** (deferred):
- Both-files-exist drift repair (separate mode, separate plan).
- Polyglot / monorepo coverage.
- Per-package or workspace-aware files.
- Drift detection or suggestions on existing files.

**When to cut v1.1+.** New repo-shape coverage that passes the same RED-GREEN gate. One shape per minor: v1.1 = real-source repo, v1.2 = monorepo, etc. The shape and its test land together — no "add the feature now, test later."

**When to cut v2.0.** Breaking the default skeleton, breaking the frontmatter shape, or changing the non-destructive default for existing files.

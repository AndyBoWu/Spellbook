# Spellbook — Planning

Upcoming spells, in rough priority order. Each entry is a sketch, not a spec — refine before implementation.

## Principles

- **MVP first.** Every spell ships its smallest useful version. Scope expands from real usage, not speculation.
- **One default, one style.** No configuration matrix on day one. Pick the opinionated default and commit.
- **No destructive defaults.** Anything that overwrites existing user content requires explicit approval.
- **No hidden dependencies.** If a spell needs an API key or external tool, say so upfront — or design around it.

---

## Planned spells

### `readme-design` — opinionated README author

Most repo READMEs are ugly. This spell treats a README like a small personal website for the repo.

#### MVP

- **One style only: minimalist.** H1 + italic tagline + badges + Install / Use / License. Sentence case, heavy whitespace, no emoji headers.
- **Real badges only.** Auto-detect from the repo and render badges that map to something that actually exists:
  - License (from `LICENSE`)
  - Language / stack (from `package.json`, `Cargo.toml`, `pyproject.toml`, `Gemfile`, `go.mod`)
  - CI status (from `.github/workflows/`)
  - Skip anything we can't verify. No vanity metrics.
- **Tagline rule.** ≤ 12 words, concrete. No "blazing fast", no "modern".
- **Single target path.** Write one README at the path the user points at. No monorepo logic.
- **Approval before overwrite.** If a README exists, show a diff and ask before writing. Empty/missing README → write freely.

#### Not in MVP (revisit after real use)

- Additional layout styles (`editorial`, `dashboard`).
- Image generation. Instead, MVP may eventually emit a copy-pasteable, model-agnostic image prompt (logo / hero / social card) the user runs in ChatGPT, Midjourney, or Gemini themselves. Out of MVP until the text-only version proves valuable.
- `--review` mode that critiques an existing README without rewriting.
- Monorepo awareness (workspace detection, per-package READMEs).
- Coverage / version / download badges.

#### Style references

Ankane's gem READMEs (terse, imperative). `bat`, `ripgrep`, `zellij` (badge layout). Tailwind UI marketing pages (hierarchy, not content).

**Status.** Planned. Not started.

---

## Backlog

One-liners only. Promote to a full section when ready to design.

- `agent-runbook` — author runbooks for AI agents (operational procedures: triggers, diagnostic steps, recovery, rollback). Scope TBD — needs to be distinct from skill-creator (skills = capability; runbooks = situational procedure).

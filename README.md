# Spellbook 📖

> A personal grimoire of Claude Code skills.

Each folder under `skills/` is one **spell** — a self-contained reference Claude Code (and other agent runtimes that speak the same skill format) can invoke on demand.

## Spells

| Spell | What it does | Status |
|---|---|---|
| [`init2`](skills/init2/) | Generates **both** `CLAUDE.md` and `AGENTS.md` from one codebase scan — for repos used with Claude Code *and* OpenAI Codex | Shipped |
| [`gha-create`](skills/gha-create/) | Generates GitHub Actions workflows with security and efficiency best practices baked in | Shipped |
| [`repo-audit`](skills/repo-audit/) | Scans a repo for secrets, code quality, missing docs, and compliance issues before going public | Shipped |
| `playbook` | Authors playbooks for AI agents — operational procedures with triggers, diagnostic steps, recovery, rollback | TO-DO |
| `repome` | Opinionated README author — minimalist layout, real badges only, no vanity metrics | TO-DO |

## Install

### Claude Code

Spellbook is a Claude Code plugin marketplace. Add it once, then install spells individually:

```
/plugin marketplace add AndyBoWu/Spellbook
/plugin install init2@spellbook
```

Swap `init2` for any spell name from the table above.

### OpenAI Codex

Codex reads skills from `~/.agents/skills/`. Clone Spellbook and symlink the spells you want:

```bash
git clone https://github.com/AndyBoWu/Spellbook.git ~/Repos/Spellbook
mkdir -p ~/.agents/skills
ln -s ~/Repos/Spellbook/skills/init2 ~/.agents/skills/init2
```

## Use

In Claude Code, invoke explicitly by namespace:

```
/spellbook:init2
```

…or just ask the agent in natural language — Claude will match the request to the installed skill:

```
> init2 this repo
> audit this repo before I make it public
> generate a GitHub Actions workflow for Node CI
```

## Add a new spell

```
skills/<spell-name>/
  SKILL.md       # required — YAML frontmatter + instructions
  TESTING.md     # recommended — TDD methodology, metrics, reproducible prompts
  ...            # supporting files only if needed
```

See [agentskills.io/specification](https://agentskills.io/specification) for the SKILL.md frontmatter spec.

Skills here are gated on **paired subagent runs** (baseline without the skill, then with). See [`skills/init2/TESTING.md`](skills/init2/TESTING.md) for the template. No skill edit lands without a failing test first.

## License

MIT © Andy Wu

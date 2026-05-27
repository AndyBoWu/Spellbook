# Spellbook 📖

> A personal grimoire of Claude Code skills.

Each folder under `skills/` is one **spell** — a self-contained reference Claude Code (and other agent runtimes that speak the same skill format) can invoke on demand.

## Spells

| Spell | What it does | Status |
|---|---|---|
| [`init2`](skills/init2/) | Generates **both** `CLAUDE.md` and `AGENTS.md` from one codebase scan — for repos used with Claude Code *and* OpenAI Codex | Shipped |
| `playbook` | Authors playbooks for AI agents — operational procedures with triggers, diagnostic steps, recovery, rollback | TO-DO |
| `repome` | Opinionated README author — minimalist layout, real badges only, no vanity metrics | TO-DO |

## Install

Spellbook is a flat collection of skills. Install means making them discoverable to your agent runtime.

### Claude Code (per-user)

Symlink (or copy) each spell into your personal skills directory:

```bash
git clone https://github.com/andybowu/Spellbook.git ~/Repos/Spellbook
mkdir -p ~/.claude/skills
ln -s ~/Repos/Spellbook/skills/init2 ~/.claude/skills/init2
```

Restart Claude Code; the skill appears in the available-skills list and is invocable via the `Skill` tool or by name in conversation.

### OpenAI Codex (per-user)

Codex CLI reads skills from `~/.agents/skills/`. Same idea:

```bash
mkdir -p ~/.agents/skills
ln -s ~/Repos/Spellbook/skills/init2 ~/.agents/skills/init2
```

## Use

Once installed:

```
> init2 this repo
```

…or invoke explicitly via the `Skill` tool. The spell handles the rest.

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

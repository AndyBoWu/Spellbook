# Spellbook 📖

> A personal grimoire of Claude Code skills.

Each folder under `skills/` is one **spell** — a self-contained reference Claude Code (and other agent runtimes that speak the same skill format) can invoke on demand.

## Spells

| Spell | What it does |
|---|---|
| [`dual-init`](skills/dual-init/) | Generates **both** `CLAUDE.md` and `AGENTS.md` from one codebase scan — for repos used with Claude Code *and* OpenAI Codex |

## Install

Spellbook is a flat collection of skills. Install means making them discoverable to your agent runtime.

### Claude Code (per-user)

Symlink (or copy) each spell into your personal skills directory:

```bash
git clone https://github.com/andybowu/Spellbook.git ~/Repos/Spellbook
mkdir -p ~/.claude/skills
ln -s ~/Repos/Spellbook/skills/dual-init ~/.claude/skills/dual-init
```

Restart Claude Code; the skill appears in the available-skills list and is invocable via the `Skill` tool or by name in conversation.

### OpenAI Codex (per-user)

Codex CLI reads skills from `~/.agents/skills/`. Same idea:

```bash
mkdir -p ~/.agents/skills
ln -s ~/Repos/Spellbook/skills/dual-init ~/.agents/skills/dual-init
```

## Use

Once installed:

```
> dual-init this repo
```

…or invoke explicitly via the `Skill` tool. The spell handles the rest.

## Add a new spell

```
skills/<spell-name>/
  SKILL.md       # required — YAML frontmatter + instructions
  ...            # supporting files only if needed
```

See [agentskills.io/specification](https://agentskills.io/specification) for the SKILL.md frontmatter spec.

## License

MIT © Andy Wu

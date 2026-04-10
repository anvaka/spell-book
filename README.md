# spell-book

A collection of spells for LLM assistants.

If you know the right sequence of words, you can accomplish so much. Each spell in this
repository is a self-contained prompt that you can install into your LLM assistant
(Claude Code, Kiro CLI, etc.) with a single command.

## Spells

| Spell | Description |
|-------|-------------|
| [process-book](spells/process-book/) | Transform a book into a multi-lens interactive exploration |

## Installation

Each spell folder contains a `spell.md` file — the actual prompt — and a `README.md`
with motivation and details.

### Claude Code

```bash
# Generic pattern:
mkdir -p ~/.claude/skills/<spell-name> && \
  curl -sL https://raw.githubusercontent.com/anvaka/spell-book/main/spells/<spell-name>/spell.md \
  -o ~/.claude/skills/<spell-name>/SKILL.md

# Example — install process-book:
mkdir -p ~/.claude/skills/process-book && \
  curl -sL https://raw.githubusercontent.com/anvaka/spell-book/main/spells/process-book/spell.md \
  -o ~/.claude/skills/process-book/SKILL.md
```

After installation, invoke the spell with `/<spell-name>` in Claude Code.

## Adding a spell

Create a new folder under `spells/` with:

```
spells/your-spell/
  README.md    # Motivation, what you get, examples
  spell.md     # The actual prompt (frontmatter + instructions)
```

## License

MIT

# Agent Skills

My collection of custom [Agent Skills](https://agentskills.io).

## Layout

Each skill lives in its own directory:

```text
skill-name/
  SKILL.md
  scripts/
  references/
  assets/
```

## Syncing

Symlink individual skills into each AI tool's local skills directory:

```bash
ln -s "$(pwd)/skill-name" ~/.codex/skills/skill-name
ln -s "$(pwd)/skill-name" ~/.gemini/skills/skill-name
ln -s "$(pwd)/skill-name" ~/.claude/skills/skill-name
```

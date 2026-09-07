# skills

Claude Code skills, one folder each under [`skills/`](skills/).

## Skills

| Skill | What it does |
|---|---|
| [`reverse-engineer-specs`](skills/reverse-engineer-specs/) | Reads an existing codebase and writes `requirements/REQUIREMENTS.md` in EARS syntax plus a complete `spec/` tree — entities, use cases, contracts, invariants, BDD scenarios and a traceability matrix — with every artifact carrying the `file:line` it was read from. Pauses at two checkpoints for review. |

## Installing

A skill is a folder. Copy or symlink the one you want into either location:

```bash
# available in every project
ln -s "$PWD/skills/reverse-engineer-specs" ~/.claude/skills/

# available in one project only
ln -s "$PWD/skills/reverse-engineer-specs" /path/to/project/.claude/skills/
```

Claude Code picks it up on the next session. Invoke it by name with `/reverse-engineer-specs`, or let Claude reach for it when the task matches its description.

## Adding a skill

```
skills/<skill-name>/
├── SKILL.md              # required; frontmatter `name` must equal the folder name
└── references/           # optional; material loaded on demand from SKILL.md
```

`SKILL.md` opens with YAML frontmatter:

```yaml
---
name: skill-name
description: "What it does, then the phrases that should trigger it."
---
```

The `description` sits in the model's context permanently, so keep it under ~400 characters and spend the words on the distinct cases that should fire the skill. Everything a run needs only sometimes belongs in `references/`, reached by a link from `SKILL.md`.

# skills

Claude Code skills, one folder each under [`skills/`](skills/).

## Skills

| Skill | What it does | Setup |
|---|---|---|
| [`reverse-engineer-specs`](skills/reverse-engineer-specs/) | Reads an existing codebase and writes `requirements/REQUIREMENTS.md` in EARS syntax plus a complete `spec/` tree — entities, use cases, contracts, invariants, BDD scenarios and a traceability matrix — with every artifact carrying the `file:line` it was read from. Pauses at two checkpoints for review. | [SETUP.md](skills/reverse-engineer-specs/SETUP.md) |

---

`reverse-engineer-specs` is adapted from the `sdd-reverse-engineer` skill in [noelserdna/sdd-pipeline](https://github.com/noelserdna/sdd-pipeline) (MIT), scoped to requirements extraction and specification generation and made self-contained.

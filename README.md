# skills

Claude Code skills, one folder each under [`skills/`](skills/).

## Skills

| Skill | What it does | Setup |
|---|---|---|
| [`feature-inventory`](skills/feature-inventory/) | Catalogs every user-facing capability of a product in one YAML file — routes, endpoints, tests, docs, flags — then reconciles that file against the code as it changes. Bootstraps interactively behind one confirmation gate; drift-syncs non-interactively, so it can be scheduled. | [SETUP.md](skills/feature-inventory/SETUP.md) |
| [`reverse-engineer-specs`](skills/reverse-engineer-specs/) | Reads an existing codebase and writes `requirements/REQUIREMENTS.md` in EARS syntax plus a complete `spec/` tree — entities, use cases, contracts, invariants, BDD scenarios and a traceability matrix — with every artifact carrying the `file:line` it was read from. Pauses at two checkpoints for review. | [SETUP.md](skills/reverse-engineer-specs/SETUP.md) |

---

`reverse-engineer-specs` is adapted from the `sdd-reverse-engineer` skill in [noelserdna/sdd-pipeline](https://github.com/noelserdna/sdd-pipeline) (MIT), scoped to requirements extraction and specification generation and made self-contained.

`feature-inventory` is forked from [yashasvigirdhar/skills](https://github.com/yashasvigirdhar/skills/tree/main/feature-inventory) (MIT) at commit [`3960a54`](https://github.com/yashasvigirdhar/skills/commit/3960a540631bad1c7f32161df59d8c3c3b100ca5), for availability rather than customization. One change from upstream: run history keeps a single home in `runs.log`, so the `Accumulated Learnings` section and its two pointers are removed. Full provenance is in the `SKILL.md` frontmatter.

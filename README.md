# skills

Claude Code skills, one folder each under [`skills/`](skills/).

## Skills

| Skill | What it does | Setup |
|---|---|---|
| [`feature-inventory`](skills/feature-inventory/) | Catalogs every user-facing capability of a product in one YAML file — routes, endpoints, tests, docs, flags — then reconciles that file against the code as it changes. Bootstraps interactively behind one confirmation gate; drift-syncs non-interactively, so it can be scheduled. | [SETUP.md](skills/feature-inventory/SETUP.md) |
| [`reverse-engineer-specs`](skills/reverse-engineer-specs/) | Reads an existing codebase and writes `requirements/REQUIREMENTS.md` in EARS syntax plus a complete `spec/` tree — entities, use cases, contracts, invariants, BDD scenarios and a traceability matrix — with every artifact carrying the `file:line` it was read from. Pauses at two checkpoints for review. | [SETUP.md](skills/reverse-engineer-specs/SETUP.md) |
| [`technical-documentation`](skills/technical-documentation/) | Audits, rewrites and writes developer documentation against Google's Developer Documentation Style Guide — sixty numbered rules with severities, five document-type skeletons, a 10-point score, and a blocking gate for any fact it cannot verify in source. Three modes (audit, improve, write) and no approval gate, so it can be driven non-interactively. | [SETUP.md](skills/technical-documentation/SETUP.md) |

---

`reverse-engineer-specs` is adapted from the `sdd-reverse-engineer` skill in [noelserdna/sdd-pipeline](https://github.com/noelserdna/sdd-pipeline) (MIT), scoped to requirements extraction and specification generation and made self-contained.

`technical-documentation` is forked from [wondelai/skills](https://github.com/wondelai/skills/tree/main/technical-documentation) (MIT (c) 2025 Wondel.ai sp. z o.o.) at commit [`6621b1f`](https://github.com/wondelai/skills/commit/6621b1f32cca2b9b17d7a18c076bf6e7da83f8a6), dated 2026-08-29 and forked on 2026-09-09, for availability rather than customization. One change from upstream: a rule its table does not carry, `F1 — Name the reader's situation`, which permits a lead-in clause before a step's imperative, permits a section to open on the reader's state rather than on a bare list, and asks for a rule's reason where the reason is one short clause. F1 is marked as a fork addition at each of the three places it appears — `SKILL.md`, `references/procedures-and-code.md` and `references/audit-checklist.md`. Everything else is byte-identical to that commit.

`feature-inventory` is forked from [yashasvigirdhar/skills](https://github.com/yashasvigirdhar/skills/tree/main/feature-inventory) (MIT) at commit [`3960a54`](https://github.com/yashasvigirdhar/skills/commit/3960a540631bad1c7f32161df59d8c3c3b100ca5), for availability rather than customization. One change from upstream: run history keeps a single home in `runs.log`, so the `Accumulated Learnings` section and its two pointers are removed. Full provenance is in the `SKILL.md` frontmatter.

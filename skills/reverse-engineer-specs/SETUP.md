# Reverse-Engineer Specs — Setup Guide

You are setting up the `reverse-engineer-specs` skill for the user's project. This skill reads an existing codebase and writes `requirements/REQUIREMENTS.md` in EARS syntax plus a complete `spec/` tree — entities, use cases, contracts, invariants, BDD scenarios, NFRs and a traceability matrix — with every artifact carrying the `file:line` it was read from.

Follow these steps in order.

## Step 1: Prerequisites Check

1. **A codebase to read** — verify the project has a source directory. The skill aborts at its pre-flight phase when the scope contains none.
2. **Git repo (optional)** — not required, but where history is available the skill uses file age, last-modified date and contributor count to adjust finding severity. It runs fine without.
3. **Output paths** — check whether `requirements/`, `spec/` or `reverse-engineering/` already exist. The skill asks the user what to do on a collision, so an existing `spec/` is not a blocker, just a question it will raise.
4. **Tooling** — none. No API keys, no external accounts, no language servers. The skill reads files and uses `grep`, `find` and `wc`.

The skill treats source and test directories as read-only; it writes only into the three output paths above.

## Step 2: Install the Skill

Fetch the four files straight from GitHub into the project's skill directory. Common locations:

- `<PROJECT_ROOT>/.claude/skills/reverse-engineer-specs/` — available in this project only
- `~/.claude/skills/reverse-engineer-specs/` — available in every project
- Wherever the user's agent looks for skills

```bash
BASE=https://raw.githubusercontent.com/pasemes/skills/main/skills/reverse-engineer-specs
INSTALL=<install_path>

mkdir -p "$INSTALL/references"
curl -fsSL "$BASE/SKILL.md" -o "$INSTALL/SKILL.md"
for f in code-analysis requirement-extraction spec-templates; do
  curl -fsSL "$BASE/references/$f.md" -o "$INSTALL/references/$f.md"
done
```

Verify all four landed — `SKILL.md` loads the three reference files by relative path, and a missing one silently degrades a phase:

```bash
find "$INSTALL" -name '*.md' | sort
```

Keep the directory named `reverse-engineer-specs`: the `name` in the `SKILL.md` frontmatter has to match the folder name. Renaming the folder means editing that field to match.

The skill is available in the next session. Invoke it with `/reverse-engineer-specs`, or let the agent reach for it when the task matches its description.

## Step 3: First Run (Optional)

Ask the user: "Want me to run it now? I'll inventory the codebase, analyse it, and stop at a checkpoint to show you what I found before anything gets written."

If they do, pick the invocation from the size of the codebase:

| Situation | Invocation |
|---|---|
| First run on any sizeable repo | `--analyze-only` — inventory, analysis and findings, no artifacts |
| One module or bounded context | `--scope=src/api,src/models` |
| Requirements without the spec tree | `--requirements-only` |
| Full extraction | no flag |

Recommend `--analyze-only` or `--scope` for a first run on a large codebase. The analysis phases read every file in scope, and it is cheaper to agree on the requirement grouping at the first checkpoint than to regenerate a spec tree built on the wrong one.

**The run is interactive.** It pauses at two checkpoints — after analysis, and after generating the artifacts — and each one needs a human answer before it continues. Do not schedule this skill or run it headless; it has nothing to do with its checkpoint questions if nobody is there to answer them.

What a full run produces:

```
reverse-engineering/    INVENTORY.md, ANALYSIS.md, TEST-ANALYSIS.md, SPEC-ID-PLAN.md
requirements/           REQUIREMENTS.md
spec/                   domain/, use-cases/, contracts/, workflows/, adr/, nfr/, tests/
                        VALUE-REGISTRY.md, CLARIFICATIONS.md, TRACEABILITY-MATRIX.md
```

The intermediate files in `reverse-engineering/` are what `--continue` resumes from, so leave them in place until the run is finished and reviewed.

## Step 4: Reviewing the Output

Point the user at the two places where their judgement is actually needed:

- **`spec/CLARIFICATIONS-PENDING.md`** — questions the code could not answer. Each one blocks a spec statement until someone decides it.
- **Anything tagged `[INFERRED]` or `[IMPLICIT-RULE]`** — read off a code pattern rather than an explicit statement. The `[IMPLICIT-RULE]` rows in `spec/CLARIFICATIONS.md` are the highest-value output of a run: business rules the codebase enforces that nothing had written down.

The skill surfaces both at the second checkpoint, but they stay in the artifacts as a standing to-do list.

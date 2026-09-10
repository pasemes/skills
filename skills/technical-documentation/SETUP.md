# Technical Documentation — Setup Guide

You are setting up the `technical-documentation` skill for the user's project. This skill audits, rewrites and writes developer documentation against Google's Developer Documentation Style Guide and Technical Writing courses: sixty numbered rules with severities, five document-type skeletons, a 10-point score, and a blocking gate for facts it cannot verify.

It runs in one of three modes, and **the caller passes the mode in**. Step 3 is where that matters.

Follow these steps in order.

## Step 1: Prerequisites Check

1. **Something to write or audit** — an existing document for `audit` and `improve`, or a reader and a task for `write`. The skill has nothing to act on without one.
2. **Fact sources, for `write` and for any audit worth trusting** — the code paths, existing docs, or user statements every command, flag, path and error string is derived from. This is the skill's hardest rule (P11, severity Blocking): it never recalls or invents one, and marks each gap `TODO(verify): …`. Point it at no sources and it returns a page of TODOs.
3. **A local style guide (optional)** — `CONTRIBUTING.md`, `STYLE.md`, `docs/style-guide.md`, `.vale.ini`, `.markdownlint*`. The skill looks for these itself and they **outrank** Google's layer, so an existing house convention is honored rather than filed as sixty findings.
4. **Tooling** — none. No API keys, no external accounts. The skill reads files and applies its own rule table. Vale with the `Google` package is optional and automates only the English word and punctuation layer.
5. **The document's language** — for a non-English document the skill drops six English-only rules (`V8, W4, W5, W7, W8, S11`) and records which on its report line. It does **not** translate a document unless the caller asks it to.

The skill treats source code as read-only. It writes only the documents it was pointed at.

## Step 2: Install the Skill

Fetch the eight files straight from GitHub into the project's skill directory. Common locations:

- `<PROJECT_ROOT>/.claude/skills/technical-documentation/` — available in this project only
- `~/.claude/skills/technical-documentation/` — available in every project
- Wherever the user's agent looks for skills

```bash
BASE=https://raw.githubusercontent.com/pasemes/skills/main/skills/technical-documentation
INSTALL=<install_path>

mkdir -p "$INSTALL/references"
curl -fsSL "$BASE/SKILL.md" -o "$INSTALL/SKILL.md"
for f in api-reference audit-checklist document-types procedures-and-code \
         release-notes structure-and-formatting voice-and-words; do
  curl -fsSL "$BASE/references/$f.md" -o "$INSTALL/references/$f.md"
done
```

Verify all eight landed — `SKILL.md` loads the seven reference files by relative path, and a missing one silently degrades a whole layer of the rule table:

```bash
find "$INSTALL" -name '*.md' | sort
```

Keep the directory named `technical-documentation`: the `name` in the `SKILL.md` frontmatter has to match the folder name. Renaming the folder means editing that field to match.

**This copy carries one rule upstream does not.** `F1 — Name the reader's situation` is a fork addition, marked as such at all three places it appears. An audit run from this install may cite an ID that upstream's rule table has no row for. The [repository README](https://github.com/pasemes/skills#readme) records what was added and why.

The skill is available in the next session. Invoke it with `/technical-documentation`, or let the agent reach for it when the task matches its description.

## Step 3: First Run

**Two values decide the run, and neither is discoverable without reading `SKILL.md`.** A caller driving this skill non-interactively — a scheduled documentation sync, a pipeline that writes a page per feature — has to pass both:

| Value | What to pass | What happens without it |
|---|---|---|
| **Mode** | `audit`, `improve` or `write` | Protocol step 1 stops and asks. There is no default. |
| **Document type** | `tutorial`, `how-to`, `concept`, `reference` or `README`, **per page** | The skill picks one from the reader's task, and a page-per-feature run drifts between shapes |

`write` needs two more values in the same breath — the reader and their level, and the fact sources from Step 1. Protocol step 1 names them required and stops without them.

Ask the user: "Want me to run it now? Tell me the mode and, for a new page, the document type and who is reading it."

Pick the mode from the state of the document:

| Situation | Mode | What comes back |
|---|---|---|
| A document exists and you want to know what is wrong with it | `audit` | A score, the failed diagnostic rows, and a findings table. Nothing is written. |
| A document exists and you want it fixed | `improve` | `Score before → after`, the full rewritten document, and a change-log table citing rule IDs |
| No document exists yet | `write` | The document, with `TODO(verify)` at every unverified claim |

**All three modes run to completion without a human.** The skill has no approval gate — it reports rather than asks — which is what makes it schedulable. The one thing that stops it is a missing required value from the table above.

## Step 4: Reviewing the Output

Point the user at the three places where their judgement is actually needed:

- **The `**Blocking:**` line, before the score.** Blocking is a gate, not a deduction: a document with one blocking finding is `Shippable: no` at 9/10, and a document with none is shippable at 6/10. Read that line first — it names facts the skill could not verify, steps that cannot be completed, and information carried only by an image.
- **Every `TODO(verify): …` marker.** These are the claims no source confirmed. The skill is instructed to leave them rather than guess, so they are a to-do list, not a defect in the run. A page shipped with one is a page that teaches something nobody checked.
- **The `**Local style guide:**` line.** It records which house conventions the run honored and which Google rules it deferred to them. When it reads `none — Google applies` and the project does have conventions, the run just filed findings against deliberate choices — point it at the file and run again.

In `improve` mode, read the **change-log table** rather than the diff. Each row carries the rule ID that motivated the edit, which is what tells a deliberate style choice apart from a rewrite that changed the meaning.

# Feature Inventory — Setup Guide

You are setting up the `feature-inventory` skill for the user's project. This skill catalogs every user-facing capability of a product in a single YAML file — frontend routes, API endpoints, tests, docs, feature flags, tied together into one structured list — and then keeps that file in sync with the code as it changes.

It runs in one of two modes, and **a single file decides which**. Step 3 is where that matters.

Follow these steps in order.

## Step 1: Prerequisites Check

1. **A codebase to read** — the skill locates features by scanning routers, controllers and test directories. It has nothing to catalog without source.
2. **Git repo (optional)** — not required to run. Needed only for the drift-sync write paths that commit or open a PR; without it the skill leaves its changes in the working tree.
3. **`gh` CLI (optional)** — needed only for the `auto_commit: false` PR flow. The skill checks with `command -v gh` and falls back to leaving changes uncommitted when it is absent.
4. **Tooling** — none. No API keys, no external accounts. The skill reads files and uses `grep`, `find` and the agent's own YAML handling.
5. **An existing inventory?** — check whether the project already has one. If it does, this is a re-install, and the `config.json` in Step 3 has to describe that existing file rather than propose a new one.

The skill treats source and test directories as read-only. It writes the inventory file, `config.json` and `runs.log`, and nothing else.

## Step 2: Install the Skill

Fetch the four files straight from GitHub into the project's skill directory. Common locations:

- `<PROJECT_ROOT>/.claude/skills/feature-inventory/` — available in this project only
- `~/.claude/skills/feature-inventory/` — available in every project
- Wherever the user's agent looks for skills

```bash
BASE=https://raw.githubusercontent.com/pasemes/skills/main/skills/feature-inventory
INSTALL=<install_path>

mkdir -p "$INSTALL/references"
curl -fsSL "$BASE/SKILL.md" -o "$INSTALL/SKILL.md"
for f in philosophy schema discovery-patterns; do
  curl -fsSL "$BASE/references/$f.md" -o "$INSTALL/references/$f.md"
done
```

Verify all four landed — `SKILL.md` loads the three reference files by relative path, and a missing one silently degrades a phase:

```bash
find "$INSTALL" -name '*.md' | sort
```

Keep the directory named `feature-inventory`: the `name` in the `SKILL.md` frontmatter has to match the folder name. Renaming the folder means editing that field to match.

**Pick the install location before you run the command, not after.** The skill reads `config.json` from *its own directory*, so a user-level install and a project-level install are two separate skills with two separate configs. Installing to both is how a project ends up with two configs that disagree.

The skill is available in the next session. Invoke it with `/feature-inventory`, or let the agent reach for it when the task matches its description.

## Step 3: First Run

**The presence of `config.json` in the install directory is the mode switch.** Nothing else selects the mode, and there is no flag to override it:

| State | Mode | What happens |
|---|---|---|
| `config.json` absent | **Bootstrap** | Explores the codebase, proposes a schema and an initial entry list, stops at one confirmation gate |
| `config.json` present | **Drift-sync** | Reconciles the existing inventory against the current code, proposes a diff |

So a first run is a bootstrap by construction — there is nothing to write the config from yet.

Ask the user: "Want me to run it now? I'll explore the codebase, propose a schema and a feature list, and stop to show you both before anything gets written."

**Bootstrap is interactive.** It pauses at one confirmation gate covering the inventory location, the schema, the vocabularies, the discovery paths, the proposed entries and its own explicit assumptions. That gate is the only correction point in the run — the greedy grouping it proposes will over-lump some features, and splitting them there is cheaper than editing the YAML afterwards. Do not run bootstrap headless.

What a bootstrap produces:

```
<inventory_path>       the YAML inventory (location agreed at the gate)
config.json            in the skill directory — the settings the run used
runs.log               in the skill directory — one entry per run
```

**Drift-sync can run non-interactively**, which is what makes it schedulable. Its write behaviour is decided by `config.auto_commit` and whether `gh` is present: commit-in-place, open a PR, or leave the change in the working tree. Confirm which of the three the user wants **before** the first drift-sync, because the PR path pushes a branch.

## Step 4: Reviewing the Output

Point the user at the three places where their judgement is actually needed:

- **The confirmation gate's "Explicit assumptions" block** — every guess the run made about what counts as a feature in this product. These are the decisions most likely to be wrong, and they are cheapest to correct before the file exists.
- **The entry list, read for what is missing rather than for what is wrong.** The gate shows what the run found. A feature it never detected appears nowhere on that screen, so ask the run to diff the full set of routes, endpoints and commands against its proposed entries and account for every surface that maps to no entry.
- **`config.json`, after the run** — this is what every future drift-sync reads. A schema deviation that is not recorded in `config.notes` will be "corrected" back to stock the next time the skill runs.

**Copy `config.json` into the repository** if the install is user-level. The skill only reads it from its own directory, which sits outside the project — so it is unversioned, invisible to anyone cloning the repo, and overwritten the moment the same skill is bootstrapped against a different project. A copy under version control makes the settings reviewable and restorable; mark it clearly as a record rather than the live file, and keep the two in sync.

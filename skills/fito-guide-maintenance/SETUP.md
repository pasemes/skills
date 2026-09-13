# Fito Guide Maintenance — Setup Guide

You are setting up the `fito-guide-maintenance` skill in a Fito checkout. This skill keeps Fito's bilingual user guide true to the product: it writes the pages a feature is owed, keeps every pt-BR twin a current translation of its English page, and settles the pages a deleted feature left behind. The guide's own checks decide when a run is done.

It serves one repository, Fito, and runs on an author's machine. It is not part of the TUI a learner installs.

It runs in one of two modes, and **the invocation names the mode**. Step 3 is where that matters.

Follow these steps in order.

## Step 1: Prerequisites Check

1. **A Fito checkout** — the skill edits `guide/` and reads the product source the pages describe.
2. **Node.js 24 and the guide's dependencies** — run `npm ci` in `guide/`. Every run ends by building the guide and running its tests.
3. **`feature-inventory`**, installed from [its setup guide](https://github.com/pasemes/skills/blob/main/skills/feature-inventory/SETUP.md), with Fito's `config.json` beside it. Sync starts with its drift-sync.
4. **`technical-documentation`**, installed from [its setup guide](https://github.com/pasemes/skills/blob/main/skills/technical-documentation/SETUP.md). It writes every page this skill decides on.
5. **Tooling** — nothing beyond those. No API keys, no external accounts, and no `gh`: the skill commits nothing and opens nothing.

The skill writes guide pages, their frontmatter, and the files that register a page with the site. It reads product source and leaves it unchanged.

## Step 2: Install the Skill

Fetch the two files into the checkout's skill directory. In Fito that is `.agents/skills/fito-guide-maintenance/`, beside the other two skills:

```bash
BASE=https://raw.githubusercontent.com/pasemes/skills/main/skills/fito-guide-maintenance
INSTALL=<fito_checkout>/.agents/skills/fito-guide-maintenance

mkdir -p "$INSTALL"
curl -fsSL "$BASE/SKILL.md" -o "$INSTALL/SKILL.md"
curl -fsSL "$BASE/SETUP.md" -o "$INSTALL/SETUP.md"
```

Verify both landed:

```bash
find "$INSTALL" -name '*.md' | sort
```

pi reads skills from `.agents/skills/` directly. Claude Code reads `.claude/skills/`, so link the folder there:

```bash
ln -s ../../.agents/skills/fito-guide-maintenance <fito_checkout>/.claude/skills/fito-guide-maintenance
```

Keep the directory named `fito-guide-maintenance`: the `name` in the `SKILL.md` frontmatter has to match it.

**The skill is user-invoked.** Its frontmatter sets `disable-model-invocation: true`, so only a person typing its name starts it. It is available in the next session.

## Step 3: First Run

**The skill reads every path and command from `config.json` in its own folder, and stops without it.** Fito versions that file at `.agents/skills/fito-guide-maintenance/config.json`, so a fresh checkout already has it, and the install above leaves it in place. Write one only when it is missing, with these keys:

| Key | What it holds |
|---|---|
| `inventory` | The feature inventory: `guide/feature-inventory.yaml` |
| `docs_root` | Where pages live: `guide/src/content/docs` |
| `twin_locale` | The twin's folder under `docs_root`, and its language |
| `style_guide` | The house style, translation rules included: `guide/STYLE.md` |
| `registration` | The files a page added, moved or deleted also edits: the sidebar config and the page-list test |
| `markers.no_feature` | The frontmatter line an English page no feature lists carries |
| `stamp` | The command that records a twin as current, with `<page>` standing for the page path |
| `checks` | The commands that decide green, in the order they run |
| `type_folders` | Each type folder in a page path, mapped to a `technical-documentation` document type |
| `root_pages` | Pages outside the `<topic>/<type>/<page>` shape, each with its topic and type |
| `topic_readers` | The reader and level for each topic |
| `fact_sources` | `authority`, cited as fact, and `context_only`, read as background only |

Ask the user: "Want me to run it now? Tell me the mode: `bootstrap` to write the pages the guide is missing, or `sync` to reconcile the guide after the product changed."

| Mode | Use it when | Claude Code | pi |
|---|---|---|---|
| `bootstrap` | Pages the inventory or the sidebar name do not exist yet | `/fito-guide-maintenance bootstrap` | `/skill:fito-guide-maintenance bootstrap` |
| `sync` | The product moved since the guide was last green | `/fito-guide-maintenance sync` | `/skill:fito-guide-maintenance sync` |

**Sync stops `feature-inventory` at its proposed diff.** With `auto_commit: false` and `gh` installed, that skill's next step pushes a branch and opens a pull request. Sync applies the diff to the working tree instead, because the coverage check fails on an inventory change that lands without its pages.

**Neither mode commits.** The working tree holds the result.

## Step 4: Reviewing the Output

Point the user at the three places where their judgement is actually needed:

- **The `Result` line, first.** `green` means every check exited 0; `FAIL` names what is still standing. A stale translation is a failure, never a warning: a pt-BR page that no longer says what its English page says is confidently wrong.
- **The pt-BR review list.** It names every twin the run wrote or changed. Read each against its English page before committing. The stamp on a twin claims the translation is current, and this read is what backs the claim.
- **Every ledger row for a page left by a deleted feature.** The run deletes the page, lists it under another feature, or keeps it as a page with no feature, and writes down why. That call changes what readers can find, so confirm it.

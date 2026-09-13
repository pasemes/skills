# Fito guide maintenance — Setup guide

You are setting up `fito-guide-maintenance` for a Fito authoring checkout. It
builds or reconciles the bilingual user guide from Fito's feature inventory. It
runs on the author's machine against the repository; it is not installed into
`tui/` and does not ship to learners.

It has two explicit modes: `bootstrap` creates guide content against the current
inventory, and `sync` reconciles inventory, guide pages, translations, commands,
and flags after product changes.

Follow these steps in order.

## Step 1: Prerequisites check

1. **A Fito repository checkout** — the skill reads `guide/`, `tui/`,
   `guide/feature-inventory.yaml`, and the repository's authoring rules. It has no
   useful standalone mode.
2. **Node.js 24 and guide dependencies** — run `npm install` in `guide/` before
   the build and test verification step.
3. **`feature-inventory` installed from the same skills repository** — it is the
   required first step of `sync` and owns inventory drift. Install it from
   [its setup guide](https://github.com/pasemes/skills/blob/main/skills/feature-inventory/SETUP.md),
   using `https://raw.githubusercontent.com/pasemes/skills/main/skills/feature-inventory`
   as its install base.
4. **`technical-documentation` installed from the same repository** — it writes
   and improves guide prose. Install it from
   [its setup guide](https://github.com/pasemes/skills/blob/main/skills/technical-documentation/SETUP.md).
5. **A clean disposable copy for seeded checks** — verification deliberately
   creates missing pages and stale command usage. Never seed those changes in the
   authoring checkout.

The skill writes guide content and configuration that makes pages reachable. It
leaves Fito's branding and one-time Astro scaffold alone.

## Step 2: Install the skill

Fetch the two files into the directory your agent scans for skills. Common
locations are `<PROJECT_ROOT>/.claude/skills/fito-guide-maintenance/` for one
project and `~/.claude/skills/fito-guide-maintenance/` for every project.

```bash
BASE=https://raw.githubusercontent.com/pasemes/skills/main/skills/fito-guide-maintenance
INSTALL=<install_path>

mkdir -p "$INSTALL"
curl -fsSL "$BASE/SKILL.md" -o "$INSTALL/SKILL.md"
curl -fsSL "$BASE/SETUP.md" -o "$INSTALL/SETUP.md"
```

Verify both files landed:

```bash
find "$INSTALL" -maxdepth 1 -name '*.md' -print | sort
```

Keep the directory named `fito-guide-maintenance`: it matches the `name` in the
skill frontmatter. This is a user-invoked skill, so start it explicitly rather
than expecting ordinary guide edits to load it.

## Step 3: First run

Choose one mode explicitly:

| Mode | When to use it | First action |
|---|---|---|
| `bootstrap` | The guide needs content built from the inventory. | Read the inventory and propose a page map. |
| `sync` | Product or inventory changes may have made the existing guide stale. | Run `feature-inventory` drift-sync. |

Start a session in the Fito repository and invoke one of:

```text
/fito-guide-maintenance bootstrap
/fito-guide-maintenance sync
```

A `bootstrap` run never scaffolds or rebrands the Astro site. A `sync` run starts
with `feature-inventory`, then reports each new-feature, deleted-feature page,
translation, and command-or-flag drift before it repairs guide content.

Both modes delegate prose to `technical-documentation`. Pass that skill an
explicit mode (`write` for a new page or `improve` for an existing page), document
type, audience, and verified source paths. Ask it to write English and pt-BR from
the same facts, then pair-review the two results under `guide/STYLE.md`.

## Step 4: Review the output

Review these parts before accepting a run:

- **The drift ledger** — every product change should name its source, affected
  English/pt-BR paths, and repair. An unowned page after feature removal needs an
  explicit retain, repurpose, or delete decision.
- **The translation pair review** — an English change with no updated pt-BR twin
  is a failure, not a warning. Check that the two pages teach the same task and
  leave commands, flags, paths, identifiers, fields, and quoted UI labels literal.
- **The verification output** — `check-feature-coverage.mjs`,
  `derive-reference-tables.mjs`, the guide build, and guide tests must all be
  green. For a first install, also review the four disposable seeded drifts and
  the bootstrap restoration check described in `SKILL.md`.

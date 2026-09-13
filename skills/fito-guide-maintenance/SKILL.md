---
name: fito-guide-maintenance
description: Bootstrap or synchronize Fito's bilingual guide from its feature inventory.
disable-model-invocation: true
---

# Fito guide maintenance

This is **author-side tooling**. It runs against the Fito repository and maintains
`guide/`; it is not a `tui/` artifact and never ships with the learner-facing TUI.

Run it explicitly with one required mode:

```text
/fito-guide-maintenance bootstrap
/fito-guide-maintenance sync
```

`bootstrap` creates guide content from the current inventory. `sync` reconciles a
guide that already exists. Do not infer a mode from the files on disk: an existing
partial guide is still a bootstrap request when the author says `bootstrap`.

## Before either mode

1. Confirm that this is the Fito repository and read `AGENTS.md`, `guide/STYLE.md`,
   `guide/feature-inventory.yaml`, and
   `.agents/skills/feature-inventory/config.json`.
2. Confirm the two dependencies are available from
   `https://github.com/pasemes/skills`:
   - `feature-inventory` maintains the inventory.
   - `technical-documentation` writes and improves the prose.
3. Record the starting Git status. Preserve unrelated changes. Work in a disposable
   copy when validating a destructive or seeded scenario.
4. Treat the inventory, the guide source tree, and the feature-coverage check as
   three views of one contract. A clean Astro build alone is not evidence that they
   agree.

## Bootstrap

Bootstrap builds **content**, not the site. The existing guide scaffold owns its
branding, Astro configuration, integrations, and npm setup.

1. Read the inventory and group its features into reader tasks. For every proposed
   page, state its audience, document type, source paths, English path, and pt-BR
   path. Reuse a page when its reader task is shared; do not create one page per
   feature by default.
2. Reconcile the proposed page paths with `docs_pages` in the inventory, the sidebar
   in `guide/astro.config.mjs`, and the page loop in
   `guide/test/build-output.test.mjs`. A new topic whose first page has a new
   document type also updates that test's type-heading expectation.
3. Delegate prose to `technical-documentation`; do not reproduce its writing rules.
   For each English page, pass `write`, its document type, audience, and verified
   source paths. Then pass the same facts, the English page, and `guide/STYLE.md`
   to write the pt-BR twin. The guide-maintenance skill decides the page map and
   hands off facts; the writing skill owns the prose.
4. Pair-review the two files before proceeding. Check that they teach the same task,
   prerequisites, steps, results, warnings, links, and literal commands, flags,
   paths, identifiers, fields, and quoted UI labels. Apply the pt-BR rules in
   `guide/STYLE.md` rather than translating literals.
5. Run the checks in [Verification](#verification). Bootstrap is complete only when
   every inventory feature has a page, every page has a twin, and the built guide
   contains the intended new sections.

## Sync

Sync starts with product drift, then repairs guide drift. Run
`feature-inventory`'s drift-sync first and retain its added, removed, and changed
entries as the input to this mode.

### Detect the four drifts

Build a drift ledger before writing. Each row names the source change, affected
English and pt-BR paths, the required repair, and whether it blocks the run.

1. **New feature with no page.** A feature added by inventory drift-sync without a
   non-empty `docs_pages` entry, or pointing to a non-existent page pair, blocks.
   Add or revise the page map, then write both locales.
2. **Page for a deleted feature.** Compare the removed feature's old `docs_pages`
   with the current inventory. A page no remaining feature owns is a reconciliation
   item: delete it, repurpose it, or explicitly retain it as a non-feature page.
   Do not silently delete a page that still serves a reader task.
3. **English page without a Portuguese twin.** Run
   `node guide/scripts/check-feature-coverage.mjs`. Any missing twin blocks. It is
   also a failure when the Git diff changes an English page's content but does not
   change its matching `pt-br/` file in the same repair; write and pair-review the
   translation before continuing.
4. **Stale command or flag.** Run
   `node guide/scripts/derive-reference-tables.mjs`. Its command-usage diff derives
   names and `argument-hint` values from `tui/prompts/`, so a changed command or
   flag blocks until both reference pages are reconciled. For prose outside its
   tables, read the changed source and every page that cites the affected literal.

The translation rule is a **failure**, never a warning. A stale translation teaches
an old product confidently; leave the sync red until the pair review is complete.

### Repair in order

1. Reconcile page ownership and `docs_pages` mappings.
2. Delegate new English pages with `technical-documentation` in `write` mode; use
   `improve` for an existing page. Always pass its document type, audience, source
   paths, and `guide/STYLE.md`.
3. Write or improve the pt-BR twin from the same verified facts, then pair-review
   it against the English result.
4. Update the sidebar and build-output page list for every page added, removed, or
   moved. Keep a page's path locale-free in the inventory.
5. Re-run the ledger and all verification commands. Do not call the sync complete
   while any row remains unresolved.

## Verification

Run these from the repository root unless a command changes directory:

```bash
cd guide && npm run build && npm test
node guide/scripts/check-feature-coverage.mjs
node guide/scripts/derive-reference-tables.mjs
```

For a bootstrap, remove one generated section from a disposable copy, run the
bootstrap workflow against the same inventory, and compare the restored English
and pt-BR sections to the removed sections for reader task, factual coverage, and
literal interface tokens.

For a sync, seed each of these changes in a disposable copy and confirm the drift
ledger identifies it before repair:

- add an inventory feature with no valid page pair;
- remove a feature while leaving its formerly owned page pair;
- remove one pt-BR page, then separately change English content without changing
  its twin;
- change a TUI prompt's `argument-hint` without updating the command reference.

Restore the disposable copy after every seed. Report each seed, the blocking
message, the repair, and the clean rerun. The skill is complete only after all four
classes are demonstrated and the unmodified repository is green.

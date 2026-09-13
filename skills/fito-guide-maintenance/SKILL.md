---
name: fito-guide-maintenance
description: Bootstrap or sync Fito's bilingual user guide against its feature inventory.
disable-model-invocation: true
---

# Fito Guide Maintenance

Keeps Fito's user guide true to the product. Every feature in the inventory has a page, every English page has a pt-BR **twin**, and every twin is a current translation of its English page.

This is author-side tooling. It runs on an author's machine, against a Fito checkout, and edits `guide/`. It is not a `tui/` artifact: the `fito-` prefix names the project, and the skill stays out of `tui/`.

**Green** decides every run. The guide carries its own checks, listed under `checks` in `config.json`, and green means every one of them exits 0. The checks decide what counts as drift and when the work is done. This skill decides what to write, where, and in what order; `technical-documentation` writes it. A run ends green, or it ends **FAIL** and names every failure still standing.

The translation question has one answer, and this skill holds it:

- **Who writes pt-BR:** this run, through `technical-documentation`, in the same run as the English page.
- **Who reviews it:** the author, from the report's pt-BR review list, before committing.
- **An English change the twin never followed:** a check failure. The run is FAIL, not green with a warning.

## Before Either Mode

1. Read `config.json` in this skill's folder. When it is missing, stop and point the author to Step 3 of `SETUP.md`. Every path and command below is a key in that file.
2. Confirm `feature-inventory` and `technical-documentation` are both loaded. When one is missing, stop and name its `SETUP.md`.
3. Take the mode from the invocation: `bootstrap` or `sync`. It has no default; when the invocation names neither, ask.
4. Run `git status` and keep its output. Leave every change you did not make as you found it.

**Done when:** the config is read, both skills are loaded, and the mode is known.

## Writing a Page

Every page and every twin is written by `technical-documentation`. This skill passes it the values below and reads what comes back.

**The English page.** A page path has the shape `<topic>/<type>/<page>`; a page outside that shape is named in `root_pages` with its topic and type. Pass:

| Value | Where it comes from |
|---|---|
| Mode | `write` when the file does not exist, `improve` when it does |
| Document type | `type_folders[<type>]` |
| Reader and level | The `personas` of every feature whose `docs_pages` lists the page, read with `topic_readers[<topic>]`; for a page no feature lists, `topic_readers[<topic>]` alone |
| Fact sources | Each listing feature's `api_endpoints`, `frontend_routes`, `tui_commands` and `test_file`, the source files behind them, and `fact_sources.authority`. `fact_sources.context_only` is background and is never cited |
| Local style guide | `style_guide` |

**The twin.** Pass the same mode rule, type, reader and facts, the finished English page as the source text, and an explicit request to translate it into the `twin_locale` language under the translation rules of `style_guide`. The request has to be explicit: `technical-documentation` translates only when asked.

**Stamp the twin after reading it against the English page, section by section.** Run `stamp` with the page path. The stamp records that the twin is current, and nothing else checks that claim.

**Register the page.** A page added, moved or deleted is also an edit to every file in `registration`. Each file's own comments say how.

**Mark the page.** An English page no feature lists carries `markers.no_feature` in its frontmatter. A page a feature lists carries none.

**Done when:** the English page, its twin, the stamp, the marker where one is owed, and every registration edit all exist.

## Bootstrap

Bootstrap builds content, not the site. The scaffold, the branding and the site settings stay as they are.

1. List every page the guide owes: every path in `docs_pages` across `inventory`, and every page the files in `registration` name. For each, record whether its English file and its twin exist under `docs_root`.
   **Done when:** every listed page has a present-or-missing mark for each locale.
2. Order the missing pages. The build fails on an internal link to a page that does not exist yet, so a page is written after the pages it links to. Pages that link in a circle are written together, in one pass.
   **Done when:** every missing page has a place in the order.
3. Write each missing page, in that order, through [Writing a Page](#writing-a-page). A page whose English file exists and whose twin is missing gets the twin only.
   **Done when:** every listed page exists in both locales.
4. Run every command in `checks`, in order. Repair each failure through [Writing a Page](#writing-a-page), and run them again.
   **Done when:** the run is green — or a failure has no repair this skill can make, and the run is FAIL.

## Sync

1. Run `feature-inventory` in drift-sync mode **through its Step 3, the proposed diff, and no further**. Its Step 4 commits, pushes or opens a pull request; sync applies the diff to `inventory` in the working tree instead, so the inventory change and the pages it needs land in one commit. A new entry's `docs_pages` names the page that will document it: an existing page when its reader task already covers the feature, a new path otherwise.
   **Done when:** the inventory in the working tree matches the code, and the diff is kept for the report.
2. Run every command in `checks`. Sort every failure line into the **ledger**, one row per failure, by kind:

   | Kind | The failure reads | The repair |
   |---|---|---|
   | New feature with no page | `docs_pages is blank`, or a listed page `has no English page` | Write the page, or list the feature on a page that already covers its reader task |
   | Page for a deleted feature | `no feature lists this page, and it carries no noFeature marker` | Delete the pair and its registration; list it under a surviving feature; or mark it when it still serves a reader. The row says which, and why |
   | Translation drift | `has no pt-BR page`, `the pt-BR twin …`, or `the English page changed after the pt-BR twin was translated` | Bring the twin up to date from the English page, then stamp it |
   | Stale command or flag | `is not a command`, `matches no command`, `appears nowhere in the TUI source`, or a reference-table diff in the tests | Improve every page that names it, in both locales |

   Any other failure — a broken link, a build error, a failing test — is a row of its own, named as it reads.
   **Done when:** every failure line from every check is exactly one ledger row.
3. Repair the ledger row by row, through [Writing a Page](#writing-a-page). An English edit makes its twin stale by construction, so every English page you touch sends its twin through the twin half of that section too.
   **Done when:** every row carries its repair.
4. Run every command in `checks`, in order. A new failure becomes a new row and returns to step 3.
   **Done when:** the run is green — or a row has no repair this skill can make, and the run is FAIL.

## Report

End both modes with this report, in this order:

1. **Result:** `green` or `FAIL`. A translation failure still standing makes the run FAIL.
2. **Inventory diff** (sync only): the added, removed and changed entries from step 1.
3. **Ledger:** each row's kind, failure, repair, and the files it touched. For a page left by a deleted feature, the decision and its reason.
4. **pt-BR review list:** every twin this run wrote or changed, for the author to read against its English page before committing.
5. **Checks:** the final summary line of each command in `checks`.

The run leaves its result in the working tree, uncommitted. The author commits it.

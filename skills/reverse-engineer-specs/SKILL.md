---
name: reverse-engineer-specs
description: "Reverse-engineers an existing codebase into requirements and specifications: extracts entities, routes, events, state machines, invariants and business rules from source and tests, then writes requirements/REQUIREMENTS.md (EARS) and a full spec/ tree with traceability. Triggers: 'reverse engineer', 'extract requirements from code', 'specs from existing code', 'document a brownfield codebase'."
---

# Reverse-Engineer Specs

Reads an existing codebase and writes two artifacts: `requirements/REQUIREMENTS.md` in EARS syntax and a complete `spec/` tree. The run ends there — no implementation plan, no task breakdown.

**Evidence** governs every line written. Each requirement, invariant, contract operation and NFR carries the `file:line` it was read from. A statement with no code behind it is a question for the user or a `[NEEDS CLARIFICATION]` marker, never a guess written as fact.

Source and test directories are read-only. The run writes into `requirements/`, `spec/` and `reverse-engineering/`.

## Modes

| Invocation | Stops after | Use case |
|---|---|---|
| default | Checkpoint 2 | Full extraction: requirements + specifications |
| `--scope=src/api,src/models` | Checkpoint 2 | Same, restricted to those paths |
| `--analyze-only` | Checkpoint 1 | Inventory, analysis and findings; no artifacts generated |
| `--requirements-only` | Phase 5 | `requirements/REQUIREMENTS.md` only |
| `--continue` | Checkpoint 2 | Resume; the files present in `reverse-engineering/` say which phases are done |

## Markers

One vocabulary, used by Phases 3, 5 and 6. Detection criteria and severity live in `references/code-analysis.md`; what each marker does to the output is here.

| Marker | Meaning | Effect on the output |
|---|---|---|
| `[INFERRED]` | Read from a code pattern rather than an explicit statement | Written and flagged; the user confirms it at Checkpoint 2 |
| `[IMPLICIT-RULE]` | Business rule embedded in a conditional with no documentation | Becomes an RN row in `spec/CLARIFICATIONS.md`, and a requirement when the behaviour is user-visible |
| `[DEAD-CODE]` | Unreachable, unused or permanently disabled | Listed in `ANALYSIS.md`; produces no requirement |
| `[ORPHAN]` | Working code whose business purpose is not discernible | Listed in `ANALYSIS.md` and raised at Checkpoint 1 for the user to name or discard |
| `[WORKAROUND]` | Temporary fix, environment hack, swallowed error | Listed in `ANALYSIS.md`; the behaviour it produces is still specified |
| `[TECH-DEBT]` | Suboptimal implementation | Listed in `ANALYSIS.md`; no effect on the specification |
| `[INFRASTRUCTURE]` | Cross-cutting pattern: logging, auth, cache, rate limit, health check | Becomes an NFR in `spec/nfr/` |
| `[NEEDS CLARIFICATION]` | Open question that blocks a spec statement | HTML comment at the spot plus a row in `spec/CLARIFICATIONS-PENDING.md` |

## Process

### Phase 1 — Pre-flight

1. Detect language(s), framework(s) and package manager from the manifest files.
2. Resolve the scope: the whole project, or the `--scope` paths.
3. Check whether `requirements/REQUIREMENTS.md` or `spec/` already exist. When either does, ask the user with `AskUserQuestion` which to do: write everything under `reverse-engineering/proposed/` instead, overwrite in place, or stop.
4. Create `reverse-engineering/` for the intermediate output.

**Done when:** the language and framework list, the scope path list and the collision decision are settled. When the scope contains no source directory, stop and say so.

### Phase 2 — Inventory

1. Every source file in scope: path, language, lines of code, lines of comments.
2. Entry points and configuration files.
3. Module dependency graph from imports and requires, plus the external package tree.
4. Architectural layers: routes/controllers/handlers → API; services/business logic → domain; repositories/DAOs/queries → data access; models/entities/types → domain model; middleware/interceptors → cross-cutting; utilities/helpers → infrastructure.
5. Design patterns in use: MVC, CQRS, event sourcing, layered, microservices.
6. Database schema where detectable: ORM models, migration files, schema definitions.
7. Test files, inventoried separately: framework, layout, naming convention.

**Output:** `reverse-engineering/INVENTORY.md`

**Done when:** every file in scope appears either in the inventory table or in an excluded list with the reason for its exclusion.

### Phase 3 — Code analysis

Load [references/code-analysis.md](references/code-analysis.md) and work module by module through the inventory, extracting:

1. **Entities** — classes, interfaces, types, structs, enums, with fields and relationships.
2. **Routes and endpoints** — method, path, parameters, request and response schemas, middleware chain.
3. **Events and messages** — the event names published and consumed, their payload shapes, the queue or topic carrying each, and the handler bound to it.
4. **State machines** — states, transitions, guards, actions.
5. **Invariants** — validation rules, assertions, constraints, guard clauses.
6. **Error paths** — error codes, exception classes, HTTP or exit statuses, and the condition that raises each.
7. **Constants and configuration** — timeouts, limits, pool sizes, TTLs, rate limits, enum value sets.
8. **Findings** — one entry per `[DEAD-CODE]`, `[TECH-DEBT]`, `[WORKAROUND]`, `[INFRASTRUCTURE]`, `[ORPHAN]` and `[IMPLICIT-RULE]` detection, each with an `FND-` id and a severity.

**Output:** `reverse-engineering/ANALYSIS.md`, organised by module.

**Done when:** every module in the inventory has a section, every extracted item carries a `file:line`, and every finding carries a marker and a severity.

### Phase 4 — Test analysis

Existing tests are the strongest evidence available: they state pre- and postconditions the source only implies.

1. Suite structure (describe/context/it nesting) → feature grouping.
2. Assertions → invariants and postconditions.
3. Setup and teardown → preconditions and state requirements.
4. Mocks and stubs → external dependency contracts.
5. Test data → valid and invalid input boundaries.
6. Given/when/then and should-style patterns → BDD scenario seeds for Phase 6.
7. Classification per test: unit, integration, e2e, performance.
8. Coverage map: which modules the tests exercise, and which have none.

**Output:** `reverse-engineering/TEST-ANALYSIS.md`

**Done when:** every test file in the inventory is classified and mapped to the modules it exercises.

---

### *** CHECKPOINT 1 — Review inventory and analysis ***

**PAUSE.** Present to the user:

- Inventory summary: files, modules, layers, detected architectural style.
- Findings summary: count and highest severity per marker.
- Test coverage summary: modules with tests, modules without.
- Every `[ORPHAN]` entry, as the list the user names or discards.
- The proposed requirement grouping (one group per business domain drawn from the module structure).

Ask with `AskUserQuestion` whether to continue into artifact generation with these findings and this grouping.

In `--analyze-only` mode, stop here.

---

### Phase 5 — Requirements extraction

Load [references/requirement-extraction.md](references/requirement-extraction.md), which carries the EARS patterns, the code-pattern-to-requirement mapping tables, the priority and confidence rules and the `REQUIREMENTS.md` output format.

1. Convert each discovered behaviour into an EARS statement using the mapping tables.
2. Mint ids: `REQ-F-NNN` functional, `REQ-NF-NNN` nonfunctional, `REQ-C-NNN` constraint. Each requirement also carries the domain **Group** agreed at Checkpoint 1.
3. Tag confidence, and attach the `Source` (`file:line`) and `Tests` (`file:line`) evidence rows.
4. Infer priority from the code signals in the reference.
5. Derive acceptance criteria in Given/When/Then form, preferring the assertions found in Phase 4 over invented ones.

**Output:** `requirements/REQUIREMENTS.md`

**Done when:** every route, every validation rule, every state transition and every `[IMPLICIT-RULE]` finding either has a requirement id or appears in the excluded list with its reason (`[DEAD-CODE]`, `[ORPHAN]` pending a user decision, or out of scope).

In `--requirements-only` mode, stop here: present the requirement counts by category and by domain group, the confidence distribution, the decision list (every LOW-confidence requirement and every `[IMPLICIT-RULE]` promoted to a requirement) and the Excluded table, then hand `requirements/REQUIREMENTS.md` over.

### Phase 6 — Specification generation

Load [references/spec-templates.md](references/spec-templates.md) for the `spec/` folder structure, the writing rules and every template. Write each file once: plan the ids, invariants and error rows before writing, then never re-open a written file to add a cross-reference.

**A. Id plan** → `reverse-engineering/SPEC-ID-PLAN.md`: each REQ mapped to its UC ids and titles, the `AC-NNN-NN` range following each UC number, the WF ids with their numbered step skeleton, the contract modules with every `API-NNN-NN` operation id, the INV areas, the ADR ids and the RN counter.

**B. Shared homes**, in this order:
- `domain/01-GLOSSARY.md` — terms from entity and module names; the "Do not use" column collects the synonyms the codebase itself uses interchangeably.
- `domain/02-ENTITIES.md`, `domain/03-VALUE-OBJECTS.md` (including the error catalog, from the Phase 3 error paths), `domain/04-STATES.md`, `domain/05-INVARIANTS.md`.
- `VALUE-REGISTRY.md` — the Phase 3 constants and configuration, one canonical name each.
- `CLARIFICATIONS.md` — one RN row per `[IMPLICIT-RULE]`, with its `file:line` in the Source column.

**C. Per use case:** for each requirement in the id plan, write `use-cases/UC-NNN-{slug}.md` and `tests/BDD-UC-NNN.md` back to back — the BDD file defines the `AC-NNN-NN` ids the UC cites. Fill the UC exception table from the error paths the code actually takes; a failure mode the code leaves unhandled is a `[NEEDS CLARIFICATION]` marker rather than an invented exception.

**D. Cross-cutting:**
- `contracts/API-{module}.md` from the route extraction, `contracts/EVENTS-{module}.md` from the pub/sub patterns, `contracts/PERMISSIONS-MATRIX.md` from the role and permission checks.
- `workflows/WF-NNN-{slug}.md` from the multi-step processes and event flows.
- `adr/ADR-NNN-{slug}.md`, one per architectural decision the code makes — framework, storage engine, auth strategy, error model, concurrency approach. Status is `Accepted [INFERRED]`; the Context is reconstructed from the code and the Alternatives table holds only options the repository shows evidence of having considered.
- `nfr/PERFORMANCE.md`, `nfr/LIMITS.md`, `nfr/SECURITY.md`, `nfr/OBSERVABILITY.md` from the `[INFRASTRUCTURE]` findings and the configuration table.
- `README.md`, `TRACEABILITY-MATRIX.md` from the id plan, and `CLARIFICATIONS-PENDING.md` (written even when empty).

**E. Gate:** run the self-validation gate and fix what it reports.

**Done when:** every requirement from Phase 5 appears in `TRACEABILITY-MATRIX.md` with at least one artifact, and the gate passes or its remaining items are rows in `CLARIFICATIONS-PENDING.md`.

#### Self-validation gate

Runs on `grep` and `wc` output, over files already in context from writing them.

1. **Ids resolve** — `grep -rhoE '(UC|WF|ADR|RN|NC)-[0-9]{3}|INV-[A-Z]+-[0-9]{3}|API-[0-9]{3}-[0-9]{2}|AC-[0-9]{3}-[0-9]{2}' spec | sort -u`; every id has a definition site (file name, heading or table row).
2. **Coverage** — every `REQ-` id in `requirements/REQUIREMENTS.md` appears in `TRACEABILITY-MATRIX.md` with at least one artifact.
3. **Placeholders** — `grep -rniE 'TBD|TODO|FIXME' spec` returns nothing.
4. **Glossary compliance** — for each synonym in the "Do not use" column, `grep -rniw` over `spec/` returns nothing.
5. **Value consistency** — each number in `VALUE-REGISTRY.md` appears elsewhere only by its registry name.
6. **Error flows** — every UC has at least one row in its `Exceptions & errors` table, and every `AC-` id it cites is defined in its BDD file.
7. **Evidence** — `grep -rLE '[A-Za-z0-9_./-]+:[0-9]+' spec/use-cases spec/contracts` lists no file, and every row in `domain/05-INVARIANTS.md` and in each contract's Operations table has a non-empty `Source` cell.
8. **Budget** — `find spec -name '*.md' -print0 | xargs -0 wc -c | tail -1` is within the budget table in `references/spec-templates.md`.

These failures are mechanical: fix them directly, then re-run only the checks that failed. Report the gate as one console table (`check | result | fixed`).

---

### *** CHECKPOINT 2 — Review generated artifacts ***

**PAUSE.** Present to the user:

- Requirements: count by category and by domain group, and the confidence distribution.
- Spec inventory: files written per directory, total characters against budget.
- The gate table.
- The decision list: every `[INFERRED]` requirement whose confidence is LOW, every `[IMPLICIT-RULE]` promoted to a requirement, and every open `NC-NNN`.
- Traceability preview: REQ → UC → WF → API.

Ask with `AskUserQuestion` which items on the decision list to correct now. Apply the corrections, re-run the gate over the files touched, then stop and hand the artifacts over.

---

## Rules

1. **Read-only source.** Files under source and test directories are read, never written.
2. **Checkpoints pause.** Both checkpoints wait for the user's answer before the next phase starts.
3. **EARS.** Every requirement statement uses one of the EARS patterns in `references/requirement-extraction.md`.
4. **Evidence.** Every generated artifact carries the `file:line` it was derived from.
5. **Confidence tagging.** An artifact read from a pattern rather than an explicit statement carries `[INFERRED]` or `[IMPLICIT-RULE]`.
6. **Ambiguity becomes a question.** An ambiguous pattern becomes a finding, an `NC-NNN` marker, or a Checkpoint question — whichever fits the phase.
7. **Git-aware.** Where git history is available, use file age, last-modified date and contributor count; they drive the severity modifiers in `references/code-analysis.md`.
8. **Write each file once.** Plan ids and rows first; re-opening a written file to patch in a cross-reference means the plan was incomplete.
9. **Output language.** Answer in the language the user writes in. Ids, code, error codes and literals stay in English.

## Output map

| Path | Files | Phase |
|---|---|---|
| `reverse-engineering/` | `INVENTORY.md`, `ANALYSIS.md`, `TEST-ANALYSIS.md`, `SPEC-ID-PLAN.md` | 2, 3, 4, 6A |
| `requirements/` | `REQUIREMENTS.md` | 5 |
| `spec/domain/` | `01-GLOSSARY.md` … `05-INVARIANTS.md` | 6B |
| `spec/use-cases/`, `spec/tests/` | `UC-NNN-{slug}.md`, `BDD-UC-NNN.md` | 6C |
| `spec/contracts/`, `spec/workflows/`, `spec/adr/`, `spec/nfr/` | per template | 6D |
| `spec/` root | `README.md`, `VALUE-REGISTRY.md`, `CLARIFICATIONS.md`, `CLARIFICATIONS-PENDING.md`, `TRACEABILITY-MATRIX.md` | 6B, 6D |

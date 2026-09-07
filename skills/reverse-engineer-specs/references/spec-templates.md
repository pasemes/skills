# Spec Templates

Phase 6: the `spec/` folder structure, the writing rules, every template and the size budget. Each template is the **maximum** shape of a document, not a form to fill in — omit an optional section that would be empty, write `None.` for a mandatory one, and cite ids instead of restating facts.

---

## 1. Folder structure

The folder is `spec/` — singular, no variants. Create it with one command:

```
mkdir -p spec/{domain,use-cases,workflows,contracts,adr,tests,nfr}
```

```
spec/
├── README.md                     # Navigation table (Template 13)
├── domain/
│   ├── 01-GLOSSARY.md            # Ubiquitous language: terms, definitions, banned synonyms
│   ├── 02-ENTITIES.md            # Entities with attributes and relationships
│   ├── 03-VALUE-OBJECTS.md       # Value objects, enums, and the error catalog
│   ├── 04-STATES.md              # State machines for every stateful entity
│   └── 05-INVARIANTS.md          # Business rules as formal invariants (INV-XXX-NNN)
├── use-cases/UC-NNN-{slug}.md    # One per user-facing behaviour
├── workflows/WF-NNN-{slug}.md    # Multi-step processes spanning use cases
├── contracts/
│   ├── API-{module}.md           # Interface contract per module
│   ├── EVENTS-{module}.md        # Domain events and async contracts
│   └── PERMISSIONS-MATRIX.md     # Role-based access control
├── adr/ADR-NNN-{slug}.md         # One per architectural decision the code makes
├── tests/BDD-UC-NNN.md           # Scenarios per use case — the only home of AC-NNN-NN
├── nfr/
│   ├── PERFORMANCE.md
│   ├── LIMITS.md
│   ├── SECURITY.md
│   └── OBSERVABILITY.md
├── VALUE-REGISTRY.md             # Canonical shared values: timeouts, limits, enums
├── CLARIFICATIONS.md             # Business rules (RN-NNN), one per [IMPLICIT-RULE]
├── CLARIFICATIONS-PENDING.md     # Open NC-NNN markers — always written, even empty
└── TRACEABILITY-MATRIX.md        # REQ → artifacts, forward table only
```

Numbered domain files `01` through `05` are mandatory. Use cases, workflows and ADRs use sequential three-digit numbering. Contracts are scoped one file per module or bounded context.

---

## 2. Writing rules

| # | Rule |
|---|---|
| W1 | **One home per fact.** Requirement text → `requirements/REQUIREMENTS.md`. Terms → `01-GLOSSARY.md`. Shared values → `VALUE-REGISTRY.md`. Error code → message → class → HTTP/exit → the error catalog in `03-VALUE-OBJECTS.md`. Rules → `05-INVARIANTS.md` (INV) and `CLARIFICATIONS.md` (RN). Given/When/Then → `tests/BDD-UC-NNN.md`, where the `AC-NNN-NN` ids are defined. Decisions → `adr/`. Every other document cites the id. |
| W2 | **Id plus at most one clause.** `REQ-F-001 (create task)` is the maximum context. A requirement statement or its acceptance criteria is never copied into a UC, contract, ADR, BDD or matrix. |
| W3 | **Table or list.** A TypeScript or YAML schema block replaces an attribute table. A transitions table replaces a state diagram, unless the machine has more than 5 states. |
| W4 | **Empty is `None.`** A mandatory section with nothing to say is the single line `None.`; an optional section is omitted. The justification for an absence, when one exists, is an ADR or RN id in `Refs`. |
| W5 | **`Refs` once.** One `Refs` row in the document header holds every traceability id. Ids are also cited inline exactly where they apply, and there is no trailing traceability section. |
| W6 | **Facts, not narrative.** A `Description` runs at most two sentences. Rationale is an ADR; a rule is an RN or INV. |
| W7 | **Boilerplate once per file.** Auth, rate limit and version appear once per contract; actors once per UC header; standard errors as one table per contract with an "Operations" column. Exceptions shared by every UC — the global error handler, a storage failure — are specified once in the workflow or the contract and cited by id. |
| W8 | **Error rows cite the code.** UC, contract and BDD rows carry `E_CODE` plus HTTP or exit status and the condition. Message text, class and description live only in the error catalog. |
| W9 | **Evidence rows.** Every UC, INV row, API operation and NFR row carries the `file:line` it was read from. This is what makes the spec auditable against the code it describes. |
| W10 | **Write each file once.** Plan ids, invariants and exception rows before writing; never re-open a written file to add a cross-reference. |

---

## 3. Templates

### Template 1 — Use case (`use-cases/UC-NNN-{slug}.md`, ≤ 3,500 chars)

````markdown
# UC-NNN — [Name]

| Field | Value |
|---|---|
| Version | 1.0 / YYYY-MM-DD |
| Refs | REQ-F-NNN (primary); WF-NNN steps 1–5; API-NNN-NN; INV-XXX-NNN; RN-NNN; ADR-NNN; BDD-UC-NNN |
| Source | `src/routes/orders.ts:24-61`; `src/services/order.ts:12-90` |
| Actors | Primary: [actor]. Secondary: [component / external system] |
| Trigger | [event, one line] |
| Priority / Confidence | Must / HIGH `[INFERRED]` |

[At most 2 sentences: what the primary actor obtains.]

## Input / Output

```typescript
interface XxxInput  { field: Type /* VO-NNN, INV-XXX-NNN */; optional?: Type }
interface XxxOutput { field: Type }
```

## Preconditions
1. [state that must hold] (INV-XXX-NNN)

## Postconditions
- Success: [observable state change] (INV-XXX-NNN); [output produced].
- Failure: [what is guaranteed untouched] (INV-XXX-NNN); error per the Exceptions table.

## Main flow
1. [Actor]: [action].
2. System: [validation / state change / response] (RN-NNN, INV-XXX-NNN).

## Extensions
- 2a. [condition] → [what differs]; resume at 3. (AC-NNN-NN)

## Exceptions & errors

| # | Step | Condition | Error code | HTTP / exit | Effect | AC | Source |
|---|---|---|---|---|---|---|---|
| E1 | 2 | [invalid or missing input] | `E_CODE` | 400 | [state unchanged (INV-…)] | AC-NNN-NN | `src/…:33` |
| E2 | 3 | [authorization denied] | `E_CODE` | 403 | … | AC-NNN-NN | `src/…:41` |

## Open questions
- NC-NNN: … *(omit the section when there are none)*
````

The main flow runs at most 10 steps, each written `Actor: action` or `System: result`. Extensions and exceptions are one row each; the `AC` column points at the scenario in `BDD-UC-NNN` that verifies it, so Given/When/Then never appears here. Every exception row is a failure path the code actually takes — a `Source` cell with no code behind it means the row belongs in `Open questions` instead.

### Template 2 — BDD scenarios (`tests/BDD-UC-NNN.md`, ≤ 2,500 chars)

````markdown
# BDD-UC-NNN — [Use case name]

> Refs: UC-NNN; REQ-F-NNN; INV-XXX-NNN. Existing tests: `tests/orders.test.ts:14-88`

Feature: [name] — As a [actor] I want [capability] so that [benefit]

Background:
  Given [common setup]

Scenario: AC-NNN-01 — [happy path] [REQ-F-NNN AC1]
  Given [precondition]
  When [action]
  Then [outcome]

Scenario: AC-NNN-03 — [exception E1] [REQ-F-NNN AC3]
  When [action]
  Then error `E_CODE` with status 400
  And [state unchanged]
````

One scenario per main flow, per extension, per exception row and per edge case; at most 6 lines each. AC ids are defined here and cited by the UC. Scenarios assert the error code and status, never the message text (W8). A scenario derived from an existing test cites that test in the header; a scenario with no test behind it is marked `[INFERRED]` in its title.

### Template 3 — Workflow (`workflows/WF-NNN-{slug}.md`, ≤ 4,000 chars)

````markdown
# WF-NNN — [Name]

| Field | Value |
|---|---|
| Trigger | [event] |
| Total timeout | `WF_TOTAL_BUDGET` |
| Actors | [who / what] |
| Refs | REQ-…; UC-…; INV-… |
| Source | `src/workers/payment.ts:8-140` |

## Steps

| # | Name | Type | Timeout | Retry | Input → Output | On failure | Compensation |
|---|---|---|---|---|---|---|---|
| 1 | [step] | sync/async/manual | [duration] | [n × backoff or —] | [schema] → [schema] | [abort / skip / retry] | [rollback or —] |

## Error scenarios

| Error | Step | Action | Result state |
|---|---|---|---|
| `E_CODE` | 2 | abort | [state] |

## Events emitted
None. *(or a table: Event | Step | Payload | Consumers)*
````

### Template 4 — API contract (`contracts/API-{module}.md`, ≤ 6,000 chars per module)

````markdown
# API-{module}

| Field | Value |
|---|---|
| Base / Version | `/api/v1` · v1 |
| Auth | JWT bearer *(or: Not applicable — in-process calls, ADR-NNN)* |
| Rate limit | `RATE_LIMIT_USER` per user *(or: Not applicable)* |
| Refs | REQ-…; UC-…; ADR-… |
| Errors | catalog in `domain/03-VALUE-OBJECTS.md` § ErrorCode |

## Operations

| ID | Method | Path | Auth | Refs | Source |
|---|---|---|---|---|---|
| API-NNN-01 | POST | `/tasks` | user | UC-001 | `src/routes/tasks.ts:12` |
| API-NNN-02 | GET | `/tasks/{id}` | user | UC-002 | `src/routes/tasks.ts:28` |

## API-NNN-01 — [name]

```typescript
// request
{ title: string /* VO-002 */ }
// response 201
{ id: number; title: string; status: TaskStatus; createdAt: string }
```
Behaviour: see UC-001 main flow. Pre/post: INV-TSK-001..006.

## Errors

| HTTP | Code | Operations | Condition |
|---|---|---|---|
| 400 | `VALIDATION_ERROR` | 01, 02 | [invalid field] |
| 401 | `UNAUTHORIZED` | all | missing or invalid token |
| 404 | `NOT_FOUND` | 02 | no resource with `id` |
````

Rows come from the error paths the code contains: 400 where the operation validates input, 401 where it authenticates, 403 where it checks a role, 404 where it addresses a resource by id, 409 on concurrent modification, 429 where it rate-limits. An operation missing a standard error the rest of the module handles is a `[NEEDS CLARIFICATION]` marker — it is either a gap in the code or a deliberate choice, and only the user knows which. A non-HTTP contract uses `Signature` in place of `Method`/`Path` and an exit code in place of the HTTP status.

### Template 5 — Domain documents (`domain/01`–`03`)

````markdown
# 01 — Glossary
| Term | Definition (≤ 20 words) | Do not use | Source |
|---|---|---|---|
| Task | Unit of work with id, title, status, timestamps (ENT-001) | todo, item, entry | `src/models/task.ts:5` |

# 02 — Entities
```typescript
/** ENT-001 — Task. Lifecycle: SM-001. Invariants: INV-TSK-001..007. Source: src/models/task.ts:5 */
interface Task { id: number /* VO-001 */; title: string /* VO-002 */; status: TaskStatus; createdAt: string }
```
Relationships: ENT-002 contains ENT-001 (1 → 0..n).

# 03 — Value Objects
```typescript
type TaskId = number;                       // VO-001: safe integer ≥ 1 (INV-TSK-002)
type TaskStatus = 'pending' | 'completed';  // VO-003: src/models/task.ts:12
```

## Error catalog
| Code | Class | HTTP / exit | Message (literal) | Raised by | Source |
|---|---|---|---|---|---|
| `E_TITLE_EMPTY` | ValidationError | 400 / 2 | `title must not be empty` | API-002-06 | `src/validation.ts:19` |
````

The "Do not use" column is where the run records the synonyms the codebase uses interchangeably for one concept — the single most useful output of a glossary built from existing code. The error catalog is the only place where messages, classes and status mappings are written, and its literal messages are copied from the code, never composed.

### Template 6 — State machines (`domain/04-STATES.md`, ≤ 3,500 chars)

````markdown
# 04 — State Machines

## SM-NNN: [Entity]  ·  Source: `src/services/order.ts:44-96`

| State | Initial | Final | Description |
|---|---|---|---|
| [state] | Yes/No | Yes/No | [≤ 8 words] |

| From | To | Trigger | Guard | Action / Events | Source |
|---|---|---|---|---|---|
| [from] | [to] | [event] | [condition or —] | [side effect or —] | `src/…:52` |

Refs: INV-XXX-NNN, UC-NNN.
````

A Mermaid diagram only where the machine has more than 5 states. A transition the code permits but no caller triggers is still a row, marked `[ORPHAN]`.

### Template 7 — Invariants (`domain/05-INVARIANTS.md`, ≤ 6,000 chars)

````markdown
# 05 — Invariants

> Areas: [TSK, STO, …]. Enforcement points: API | REPO | DB | CLI.

| ID | Rule | Enforced at | UCs | Violation error | Source |
|---|---|---|---|---|---|
| INV-TSK-001 | `id` unique within a store | API, REPO | UC-001, UC-006 | `E_STORE_INVALID_STRUCTURE` | `src/repo.ts:31` |
| INV-TSK-003 | `title` trimmed, 1..`TITLE_MAX_LENGTH`, no `\n`/`\r` | CLI, API | UC-001 | `E_TITLE_*` | `src/validation.ts:12` |

### INV-TSK-003 validation
```typescript
const validTitle = (v: unknown): v is string => typeof v === 'string' && v === v.trim() && v.length >= 1 && v.length <= 1000;
```
````

One row per invariant, with a code block below the table only where the validation does not fit a cell — and there, quote the real validation from the source rather than a paraphrase.

### Template 8 — Nonfunctional (`nfr/*.md`, ≤ 2,500 chars each)

````markdown
# NFR — [Performance | Limits | Security | Observability]

> Refs: REQ-NF-…; values by name from VALUE-REGISTRY.md.

| ID | Metric / Control | Target | Measurement / Enforcement | Refs | Source |
|---|---|---|---|---|---|
| SPEC-PERF-001 | p95 latency of [operation] | < `PERF_P95_LATENCY` | [how, where] | REQ-NF-001 | `config/server.ts:8` |
| SEC-001 | Authentication | JWT bearer, `JWT_EXPIRY` | auth middleware | REQ-NF-004 | `src/middleware/auth.ts:15` |
| SEC-002 | Encryption at rest | Not applicable (ADR-NNN) | — | — | — |
````

One table per file. A control the code does not implement is one row whose Target is `Not applicable (ADR-NNN)` — and in a reverse-engineering run that ADR records that the absence was observed in the code, not that it was decided.

### Template 9 — ADR (`adr/ADR-NNN-{slug}.md`, ≤ 1,500 chars)

````markdown
# ADR-NNN — [Decision stated as a sentence]

| Field | Value |
|---|---|
| Status | Accepted `[INFERRED]` · YYYY-MM-DD |
| Refs | REQ-…; UC-…; INV-… |
| Evidence | `package.json:24`; `src/db/client.ts:1-30` |

## Context
[≤ 5 lines: the forces the code implies. Cite ids.]

## Decision
[1–3 sentences describing what the code does.]

## Alternatives
| Option | Why not |
|---|---|
| [B] | [one clause — only where the repository shows the option was considered] |

## Consequences
- + [positive, observable in the code]
- − [negative or accepted risk; the finding id where one exists]
````

One ADR per decision the code embodies: framework, storage engine, auth strategy, error model, concurrency approach. The Status is always `Accepted [INFERRED]` — the decision is in production and the rationale is reconstructed. Where no evidence of the alternatives survives, the Alternatives table is `None.` rather than a guess. A decision that is really a business rule is an RN in `CLARIFICATIONS.md`, not an ADR.

### Template 10 — Value registry (`VALUE-REGISTRY.md`, ≤ 3,000 chars)

````markdown
# Value Registry

> Canonical source for every value used in 2+ documents. Other documents cite the **name**.

| Name | Value | Unit | Category | Source | Used in (ids) |
|---|---|---|---|---|---|
| `TITLE_MAX_LENGTH` | 1000 | UTF-16 units | limit | `src/validation.ts:9` | INV-TSK-003, UC-001, API-002 |
| `PERF_P95_LATENCY` | 200 | ms | performance | `config/server.ts:8` | SPEC-PERF-001 |
| `TASK_STATUS` | pending, completed | enum | enum | `src/models/task.ts:12` | UC-002, UC-005 |
````

Every number and enum set the analysis found in code or config lands here first, then is cited by name everywhere else. This is the single strongest defence against a spec that contradicts itself across documents.

### Template 11 — Clarifications (`CLARIFICATIONS.md`, ≤ 6,000 chars for ≤ 25 rules)

````markdown
# Clarifications (business rules)

> Extracted from the codebase; each RN is a rule the code enforces. Rows confirmed by the user at Checkpoint 2 are marked Confirmed.

| RN | Source | Rule enforced | Evidence | Confidence | Status |
|---|---|---|---|---|---|
| RN-001 | REQ-F-001 | `title` is trimmed; empty after trim → `E_TITLE_EMPTY` | `src/validation.ts:12` | HIGH | Confirmed |
| RN-003 | — | Orders over `ORDER_REVIEW_THRESHOLD` require manual review | `src/services/order.ts:88` | MEDIUM `[IMPLICIT-RULE]` | Pending |
````

One row per `[IMPLICIT-RULE]` finding, plus one per rule the user states at a checkpoint. A rule with no code behind it does not belong here — that is an `NC` question. `grep RN-003 spec/` is the impact list, which is why no "Affects" column exists.

### Template 12 — Traceability matrix (`TRACEABILITY-MATRIX.md`, ≤ 3,000 chars)

````markdown
# Traceability Matrix

> REQ → spec artifacts. The reverse direction is derivable: every artifact carries a `Refs` row.

| REQ | Summary (≤ 6 words) | UC / WF | API | INV | ADR | BDD | NFR | RN |
|---|---|---|---|---|---|---|---|---|
| REQ-F-001 | create task | UC-001; WF-001 | API-002-02 | INV-TSK-001..006 | ADR-003 | BDD-UC-001 | — | RN-001..005 |

Coverage: N/N requirements specified (100 %). Requirements with no artifact: none.
````

### Template 13 — README (`spec/README.md`, ≤ 2,500 chars)

````markdown
# Specifications — [project]

> Reverse-engineered from the codebase at [commit SHA] on YYYY-MM-DD. Requirements: `../requirements/REQUIREMENTS.md`. Open questions: `CLARIFICATIONS-PENDING.md`.

[≤ 3 lines: what the system is, as the code shows it.]

| Path | Content | Start here if… |
|---|---|---|
| `domain/` | glossary, entities, value objects and error catalog, states, invariants | …you write or review anything |
| `use-cases/` | one per user-facing behaviour | …you change a feature |
| `contracts/` | interface contracts per module | …you change an interface |
| `tests/` | BDD scenarios, property tests | …you write tests |

Confidence: N requirements `[INFERRED]`, N rules `[IMPLICIT-RULE]`, N open `NC` markers.
````

The confidence line is what tells a reader how much of this document was read off the code versus reconstructed — it belongs on the front page of a reverse-engineered spec.

### Template 14 — Absence declaration

For `contracts/EVENTS-{module}.md` and `contracts/PERMISSIONS-MATRIX.md` where the code has no such surface (≤ 300 chars each):

````markdown
# EVENTS-{module}
None. Synchronous single-process system; no asynchronous contracts observed (ADR-NNN).
````

````markdown
# Permissions Matrix
| Role | Operations | Row-level rule |
|---|---|---|
| [single role] | all (API-NNN-01..04) | none — single local user (`src/auth.ts:4`) |
````

---

## 4. Needs-clarification markers

Where the code leaves a question the user has not answered, mark the spot rather than guessing:

```
<!-- [NEEDS CLARIFICATION] NC-NNN: {concise question} -->
```

Placed immediately after the text it refers to; an HTML comment so it survives rendering. Three digits, one counter across the run. Every marker also gets a row:

````markdown
# Pending Clarifications

| ID | Document | Question | Evidence | Inserted | Resolved |
|----|----------|----------|----------|----------|----------|
| NC-001 | use-cases/UC-005-upload.md | Max file size: the 10MB in config or the 25MB in the client? | `config/upload.ts:4`; `web/upload.ts:31` | YYYY-MM-DD | — |
````

`CLARIFICATIONS-PENDING.md` is always written, with the empty table and the line `(No pending clarifications)` where there are none. A marker is a last resort: at Checkpoint 2 the user answers what they can, and a resolved marker moves to `CLARIFICATIONS.md` as an RN row.

---

## 5. Size budget

Ceilings in characters (`wc -c`). Exceeding one by more than 20 % means the document repeats something that already has an id: cut, do not reflow.

| Artifact | Max chars |
|---|---|
| `use-cases/UC-NNN` | 3,500 |
| `tests/BDD-UC-NNN` | 2,500 |
| `workflows/WF-NNN` | 4,000 |
| `contracts/API-{module}` | 6,000 |
| `adr/ADR-NNN` | 1,500 |
| `domain/` 01 · 02 · 03 · 04 · 05 | 4,000 · 4,000 · 5,000 · 3,500 · 6,000 |
| `nfr/*.md` each | 2,500 |
| `CLARIFICATIONS.md` | 6,000 for ≤ 25 rules, +200 per extra rule |
| `VALUE-REGISTRY.md` · `TRACEABILITY-MATRIX.md` | 3,000 · 3,000 |
| `README.md` | 2,500 |
| `EVENTS-*.md` · `PERMISSIONS-MATRIX.md` when not applicable | 300 each |

**Total: ≤ 120,000 chars for ≤ 15 requirements**, plus 5,000 per additional functional requirement — one UC, its BDD file and its share of contract rows. Measure with `find spec -name '*.md' -print0 | xargs -0 wc -c | tail -1` and report the figure at Checkpoint 2.

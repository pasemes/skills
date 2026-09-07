# Requirement Extraction

Phase 5: turning the Phase 3 and 4 analysis into `requirements/REQUIREMENTS.md`. Carries the EARS notation, the code-pattern mapping tables, the priority and confidence rules, and the output format.

---

## 1. EARS syntax

Every requirement statement uses one of six patterns.

| Pattern | Template | Example |
|---|---|---|
| Ubiquitous | `THE <system> SHALL <behavior>` | THE system SHALL store all data encrypted at rest |
| Event-driven | `WHEN <trigger> THE <system> SHALL <behavior>` | WHEN a user submits login credentials THE system SHALL validate them within 2 seconds |
| State-driven | `WHILE <state> THE <system> SHALL <behavior>` | WHILE the system is in maintenance mode THE system SHALL reject all write operations |
| Unwanted | `IF <condition> THEN THE <system> SHALL <behavior>` | IF the database connection fails THEN THE system SHALL retry 3 times with exponential backoff |
| Optional | `WHERE <feature> THE <system> SHALL <behavior>` | WHERE multi-tenancy is enabled THE system SHALL isolate tenant data |
| Complex | `WHILE <state> WHEN <trigger> THE <system> SHALL <behavior>` | WHILE authenticated WHEN the session expires THE system SHALL redirect to login |

**Quantified language only.** These words describe nothing measurable and have no place in a statement: fast, user-friendly, efficient, flexible, robust, easy, intuitive, seamless, adequate, reasonable, appropriate, simple, quickly, etc., and/or, if applicable, as needed. Where the code gives a number, use the number; where it does not, the statement is `[INFERRED]` and the number is an `NC` question.

---

## 2. Code pattern → requirement

### Functional

| Code pattern | Requirement type | EARS template | Confidence |
|---|---|---|---|
| Route or endpoint handler | Feature | `WHEN <method + path> THE system SHALL <handler behavior>` | HIGH |
| CRUD operations on an entity | Data management | `THE system SHALL allow <actor> to <create/read/update/delete> <entity>` | HIGH |
| Event handler or listener | Event-driven behaviour | `WHEN <event> is received THE system SHALL <reaction>` | HIGH |
| Scheduled job or cron | Temporal behaviour | `WHILE <schedule active> THE system SHALL <periodic action>` | MEDIUM |
| Background worker or queue | Async processing | `WHEN <message> is queued THE system SHALL <processing>` | MEDIUM |
| State transition function | State behaviour | `WHEN <entity> is in <state> AND <event> occurs THE system SHALL <transition>` | HIGH |
| Conditional business logic | Business rule | `IF <condition> THEN THE system SHALL <behavior>` | MEDIUM |
| Outbound integration call | External integration | `THE system SHALL integrate with <external system> to <purpose>` | HIGH |
| CLI command or subcommand | Feature | `WHEN <command> is invoked THE system SHALL <behavior>` | HIGH |

### Nonfunctional

| Code pattern | Category | EARS template | Confidence |
|---|---|---|---|
| Timeout configuration | Performance | `THE system SHALL respond within <timeout>ms` | MEDIUM |
| Rate limiter | Scalability | `THE system SHALL handle <rate> requests per <period>` | HIGH |
| Cache implementation | Performance | `THE system SHALL cache <data> for <TTL>` | MEDIUM |
| Auth middleware | Security | `THE system SHALL authenticate users via <mechanism>` | HIGH |
| Role or permission check | Security | `THE system SHALL restrict <action> to <role>` | HIGH |
| Input validation | Security / data | `THE system SHALL validate that <field> <constraint>` | HIGH |
| Retry logic | Reliability | `WHEN <operation> fails THE system SHALL retry up to <N> times` | MEDIUM |
| Circuit breaker | Reliability | `WHEN <service> is unavailable THE system SHALL <fallback>` | MEDIUM |
| Encryption usage | Security | `THE system SHALL encrypt <data> using <algorithm>` | HIGH |
| Audit logging | Compliance | `THE system SHALL log <event> with <details>` | MEDIUM |

### Constraints

| Code pattern | Type | EARS template | Confidence |
|---|---|---|---|
| Max file size check | Operational | `THE system SHALL reject uploads exceeding <size>` | HIGH |
| Character limit validation | Data | `THE system SHALL limit <field> to <N> characters` | HIGH |
| Required field check | Data integrity | `THE system SHALL require <field> for <operation>` | HIGH |
| Unique constraint | Data integrity | `THE system SHALL ensure <field> is unique across <scope>` | HIGH |
| Format validation (regex) | Data format | `THE system SHALL validate <field> matches <format>` | HIGH |
| Range check | Business rule | `THE system SHALL ensure <field> is between <min> and <max>` | HIGH |

---

## 3. Conversion examples

**From a guard clause**

```
Code:    if (!user.isActive) throw new ForbiddenError('Account inactive')
EARS:    WHEN a user attempts to access the system THE system SHALL verify the user account is active
Unwanted: IF the user account is inactive THEN THE system SHALL reject access with a Forbidden error
```

**From a validation schema**

```
Code:    email: z.string().email().max(255)
EARS:    THE system SHALL validate that the email field is a valid email address not exceeding 255 characters
```

**From a route handler** — one requirement for the feature, sub-requirements for each middleware concern

```
Code:    router.post('/orders', authMiddleware, validateOrder, createOrder)
EARS:    WHEN an authenticated user submits a valid order THE system SHALL create the order and return confirmation
Sub:     THE system SHALL authenticate the user before processing order creation
Sub:     THE system SHALL validate order data according to the order schema
```

**From an event handler**

```
Code:    eventBus.on('payment.completed', async (e) => { await updateOrderStatus(e.orderId, 'paid') })
EARS:    WHEN a payment.completed event is received THE system SHALL update the corresponding order status to paid
```

**From a state machine**

```
Code:    case 'PENDING': if (action === 'APPROVE') return 'ACTIVE'
EARS:    WHEN an entity in PENDING state receives an APPROVE action THE system SHALL transition it to ACTIVE state
```

---

## 4. Priority inference

Brownfield code carries no priority markers. Infer from the code signals.

| Priority | Signals |
|---|---|
| CRITICAL | Auth and security checks, data integrity constraints, payment processing, error handling that prevents data loss |
| HIGH | Core CRUD, main business workflows, external integration points, state transitions |
| MEDIUM | Secondary features, reporting, notifications, caching, logging |
| LOW | Admin utilities, configuration endpoints, health checks, metrics, dev-only features |

**Raise one level** where the behaviour has comprehensive test coverage, or sits in the critical path with many dependents. Custom error types around a behaviour also signal importance.

**Lower one level** where the behaviour sits behind a feature flag. A `@deprecated` marker sets LOW outright. Code in `utils/`, `helpers/` or `common/` lands at MEDIUM or LOW.

---

## 5. Granularity

**One requirement** covers one cohesive behaviour: independently testable, with clear pre- and postconditions, mapping to one or a few source functions.

**Split into several** when a handler performs distinct actions, an endpoint carries several validation rules, a state machine has several transitions, or CRUD operations have different auth rules per verb.

**Group** multiple endpoints for one entity into a CRUD group, related business rules under a domain concept, and related NFRs by category.

The working rules:

```
One requirement per distinct user-observable behaviour
One requirement per validation rule — each is independently testable
One requirement per state transition
CRUD operations become one group with sub-requirements
NFRs group by category: performance, security, reliability
Functional and nonfunctional aspects stay in separate requirements
```

---

## 6. Confidence

| Level | Tag | Criteria |
|---|---|---|
| Definite | none | Directly observable: a validation message, an error string, an explicit check |
| High | `[INFERRED]` | Strong pattern match: standard CRUD, a clear state machine, typed schemas |
| Medium | `[INFERRED]` | Pattern match with ambiguity: complex conditionals, implicit flows |
| Low | `[INFERRED][IMPLICIT-RULE]` | Business logic embedded in code with no documentation or explanatory naming |

**Raise one level** for each of: a test validates the behaviour; a comment or docstring explains the purpose; an error message states the business rule; several code paths enforce the same rule.

**Lower one level** for each of: a TODO or FIXME sits near the logic; dead code or commented alternatives exist beside it; a feature flag controls it; the error raised is generic rather than domain-specific.

Every LOW-confidence requirement goes on the Checkpoint 2 decision list.

---

## 7. Output format

`requirements/REQUIREMENTS.md`. Ids are `REQ-F-NNN` functional, `REQ-NF-NNN` nonfunctional, `REQ-C-NNN` constraint; the domain **Group** agreed at Checkpoint 1 is an attribute, which keeps ids stable when a requirement moves between groups.

````markdown
# Requirements Document

> **Project:** {name}
> **Version:** 1.0
> **Generated:** {YYYY-MM-DD} by reverse engineering of {scope}
> **Status:** Draft — every `[INFERRED]` requirement awaits confirmation

## Functional Requirements

### REQ-F-001: {Title}
- **Statement:** WHEN {trigger} THE system SHALL {behavior}
- **Group:** {domain group}
- **Priority:** CRITICAL | HIGH | MEDIUM | LOW
- **Confidence:** HIGH `[INFERRED]`
- **Source:** `src/middleware/auth.ts:15-42`
- **Tests:** `tests/middleware/auth.test.ts:10-85` (or `[NO-TEST]`)
- **Signals:** {the patterns that produced it — auth middleware, JWT decode, 401 response}
- **Acceptance criteria:**
  - GIVEN {context} WHEN {action} THEN {outcome}
- **Dependencies:** REQ-F-NNN (or `None`)

## Nonfunctional Requirements

### REQ-NF-001: {Title}
- **Statement:** THE system SHALL {behavior} {quantified constraint}
- **Category:** Performance | Security | Scalability | Availability | Usability | Observability
- **Priority:** …
- **Confidence:** …
- **Source:** `config/server.ts:12`
- **Metric:** {the value read from the code, e.g. `p99 < 200ms`, `pool = 20`}
- **Acceptance criteria:**
  - GIVEN {load condition} WHEN {action} THEN {measurable outcome}

## Constraints

### REQ-C-001: {Title}
- **Statement:** {constraint}
- **Type:** Technical | Business | Regulatory
- **Confidence:** …
- **Source:** `src/upload.ts:31`

## Excluded from requirements

| Code | Marker | Reason | Location |
|---|---|---|---|
| `legacyExport()` | `[DEAD-CODE]` | Unused export, zero importers | `src/legacy.ts:88` |
| `/internal/debug` | `[ORPHAN]` | Discarded by the user at Checkpoint 1 | `src/routes/debug.ts:4` |

## Traceability

| REQ ID | Type | Group | Priority | Confidence | Source | Tests |
|---|---|---|---|---|---|---|
| REQ-F-001 | Functional | auth | CRITICAL | HIGH `[INFERRED]` | `src/middleware/auth.ts:15` | yes |
````

**Rules for the document**

1. Every requirement carries a unique id, an EARS statement, a confidence tag and a `Source` line — the `Source` is what separates this document from a wish list.
2. Acceptance criteria come from the Phase 4 assertions wherever tests exist; invented criteria are marked `[INFERRED]`.
3. `[NO-TEST]` is a real value in the `Tests` field, and it is what the confidence downgrade rests on.
4. The **Excluded** table is part of the deliverable: it is the record of what the codebase contains that deliberately became no requirement.
5. The traceability table lists every requirement, and is the input to the Phase 6 id plan.

---

## 8. Edge cases

**Feature flags.** Document the behaviour as a requirement, noting `[FEATURE-FLAG: name]`. A flag pinned on in the production config is a normal requirement; a flag pinned off makes the code a `[DEAD-CODE]` finding instead.

**A/B tests.** Document both variants as alternative requirements, noting `[A/B-TEST: experiment]` — the arrangement may be temporary.

**Third-party wrappers.** The requirement covers the integration, not the library's behaviour: what goes in, what comes out, how errors are handled, what the code assumes about availability.

**Generated code.** ORM models generated from a schema, or clients generated from an OpenAPI document, trace to the generator input. The requirement comes from the source of truth, not from the generated output.

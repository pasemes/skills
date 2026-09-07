# Code Analysis

Detection patterns for Phase 3: what to look for in the source, per language and framework, and how to classify what turns up. Every detection ends in an `ANALYSIS.md` entry carrying its `file:line`.

The patterns below are the common shapes, not a closed list. A framework absent from these tables is read the same way: find where the framework registers a handler, declares a schema or binds a constraint, and treat that site as the equivalent pattern.

---

## 1. Entity detection

### TypeScript / JavaScript

```
class {Name} with properties          → Entity
interface {Name} / type {Name} = {    → Value object or DTO
@Entity, @Model, @Table decorators    → ORM entity
enum {Name}                           → Enumeration or state set
extends / implements                  → Inheritance hierarchy
validation decorators on properties   → Invariants
```

### Python

```
class {Name}(Base) with SQLAlchemy    → ORM entity
@dataclass                            → Value object or DTO
class {Name}(BaseModel)               → Pydantic model (API schema)
class {Name}(Enum)                    → Enumeration
__init__ with validation              → Invariants
@property                             → Computed field
```

### Rust

```
struct {Name} with #[derive(...)]     → Entity or value object
enum {Name} with variants             → State machine or enumeration
impl {Name}                           → Methods and behaviour
trait {Name}                          → Interface contract
pub vs private                        → Visibility boundary = module API
```

### Go

```
type {Name} struct                    → Entity
type {Name} interface                 → Contract
func (r *{Name}) Method()             → Behaviour
embedded structs                      → Composition
unexported fields                     → Encapsulation boundary
```

### Java / Kotlin

```
@Entity                               → JPA entity
record {Name}                         → Value object or DTO
interface {Name}                      → Contract
abstract class                        → Base entity
@Embeddable                           → Value object in the DDD sense
```

---

## 2. Route and endpoint detection

### Express / Fastify / Koa

```
router.get('/path', handler)               → GET endpoint
app.post('/path', middleware, handler)     → POST with middleware
Router()                                   → Route group
```

Extract: method, path, path parameters, middleware stack (auth, validation), request body type from TypeScript types or the validation schema, response type from the return type or the `res.json` calls.

### FastAPI / Django / Flask

```
@app.get('/path')                          → FastAPI endpoint
@api_view(['GET'])                         → DRF endpoint
path('url/', view) in urls.py              → Django URL
@app.route('/path', methods=['GET'])       → Flask endpoint
```

Extract: Pydantic models in the parameters as request and response schemas, `Depends()` as dependency injection (auth, DB session), `status_code` as the expected responses.

### Spring Boot

```
@GetMapping("/path")                       → GET endpoint
@RestController                            → Controller class
@RequestBody                               → Request schema
@PathVariable, @RequestParam               → Parameters
```

Extract: DTO classes as request and response schemas, `@PreAuthorize` as authorization rules, `@Validated` as validation constraints.

### Gin / Echo / Fiber

```
r.GET("/path", handler)                    → Gin endpoint
e.POST("/path", handler)                   → Echo endpoint
middleware groups                          → Route-level concerns
```

Extract: context binding as the request schema, `c.JSON(status, response)` as the response schema.

### Non-HTTP entry points

CLI commands (argument parsers, subcommand registries), queue consumers, cron registrations and exported library functions are entry points too. Each becomes a contract operation with `Signature` in place of `Method`/`Path` and an exit code in place of an HTTP status.

---

## 3. State machine detection

**Explicit machines**

```
enum Status { PENDING, ACTIVE, CLOSED }    → State set
function transition(current, event)        → Transition function
switch (state) { case PENDING: ... }       → Transition table
classes per state with handle()            → State pattern
createMachine({ states: { ... } })         → XState definition
```

**Implicit machines** — the more common case in brownfield code:

```
entity.status = 'active'                              → Transition
if (order.status !== 'pending') throw                 → Guard
state changes in a fixed sequence                     → Ordering constraint
event handlers that change state                      → Event-driven transition
```

Extract the full state set (from the enum, the string literals or the database values), the valid transitions (from the code paths), the guards (from the conditions checked before each transition) and the actions (side effects during the transition).

---

## 4. Invariant detection

```
Validation rules
  Joi / Zod / Yup schemas                → Field-level constraints
  class-validator decorators             → Entity constraints
  if (!value) throw                      → Guard clause
  Pydantic validators                    → Field and model rules

Business rules
  if (balance < amount) throw InsufficientFunds
  assert(quantity > 0)
  if (!authorized) return 403
  complex conditionals encoding business logic

Referential integrity
  foreign key constraints in ORM definitions
  cascade rules (onDelete, onUpdate)
  unique constraints

Temporal invariants
  createdAt / updatedAt patterns
  TTL and expiry checks
  sequential ordering constraints
```

---

## 5. Configuration to specification

| Config value | Becomes |
|---|---|
| Connection pool size | NFR: concurrent connections |
| Request timeout | NFR: response time |
| Cache TTL | NFR: data freshness |
| Rate limit | NFR: throughput |
| Retry count and backoff | NFR: reliability |
| Max upload size | Constraint requirement |
| CORS origins | Security requirement |
| JWT expiry | Security requirement |
| Log level | Operational requirement |
| Feature flag default | Business rule to document |

Every value found here is also a row in `spec/VALUE-REGISTRY.md` with a canonical name.

---

## 6. Findings

Each finding gets an id `FND-{XX}-{NNN}` where `XX` is `DC` dead code, `TD` tech debt, `WA` workaround, `IF` infrastructure, `OR` orphan, `IR` implicit rule. What each marker does to the generated artifacts is the marker table in `SKILL.md`.

### `[DEAD-CODE]` — unreachable or unused

Detected by call-graph analysis: build the export graph and the import graph, treat entry points (main, route handlers, event listeners, CLI commands, exported library API) as roots, and find what no root reaches.

| Subtype | Detection | Severity |
|---|---|---|
| Unused export | Exported symbol, zero imports | INFO |
| Unreachable function | No call path from any root | MEDIUM — may be reached by reflection or dynamic dispatch |
| Commented-out code | Comment block over 5 lines that parses as code | LOW |
| Dead branch | `if (false)`, constant condition, feature flag pinned off | MEDIUM |
| Deprecated code | `@deprecated` marker or deprecation comment | INFO |
| Orphan file | Source file no other file imports | MEDIUM — verify it is not an entry point |
| Unused dependency | Package in the manifest, never imported | LOW |
| Dead route | Route registered whose handler is a no-op or always 404 | MEDIUM |

### `[TECH-DEBT]` — suboptimal implementation

| Subtype | Detection | Severity |
|---|---|---|
| TODO / FIXME / XXX / HACK | Comment marker | Varies with age |
| Complexity hotspot | Cyclomatic complexity over 10, or nesting over 4 levels | MEDIUM |
| Duplication | Similar blocks over 10 lines in several places | MEDIUM |
| Large file | Over 500 lines | LOW |
| Large function | Over 50 lines | LOW |
| Magic number | Numeric literal with no named constant | LOW |
| Type suppression | `as any`, `type: ignore`, `@SuppressWarnings` | MEDIUM |
| Missing error handling | Async operation with no `try`/`catch` or `.catch()` | HIGH |
| Inconsistent pattern | The same concern handled differently across modules | LOW |
| Outdated dependency | Known vulnerability, or a major version behind | HIGH |

### `[WORKAROUND]` — temporary fix

| Subtype | Detection | Severity |
|---|---|---|
| Explicit workaround | Comment containing "workaround", "temporary", "hack", "hotfix" | MEDIUM |
| Environment hack | Code path keyed to a specific environment rather than to config | MEDIUM |
| Version pin | Version check or pin due to an upstream bug | MEDIUM |
| Monkey patch | Runtime modification of an existing object or prototype | HIGH |
| Error swallowing | Empty catch block, or a catch that only logs | MEDIUM |
| Retry without backoff | Retry loop with no exponential backoff or limit | MEDIUM |
| Hardcoded config | Value inline that belongs in configuration | LOW |
| Compatibility shim | Code existing only for backward compatibility | LOW |

### `[INFRASTRUCTURE]` — cross-cutting pattern

Not defects. Each one becomes an NFR row, which is why they are collected.

| Subtype | Detection | Becomes |
|---|---|---|
| Logging | Logger usage and configuration | `nfr/OBSERVABILITY.md` |
| Error handling | Global handler, error middleware | `nfr/OBSERVABILITY.md` + the error catalog |
| Authentication | JWT decode, session check, OAuth flow | `nfr/SECURITY.md` |
| Authorization | Role checks, permission guards | `nfr/SECURITY.md` + `contracts/PERMISSIONS-MATRIX.md` |
| Caching | Cache client, cache decorators | `nfr/PERFORMANCE.md` |
| Rate limiting | Rate limiter middleware, token bucket | `nfr/LIMITS.md` |
| Health check | Health and readiness endpoints | `nfr/OBSERVABILITY.md` |
| Metrics | Prometheus, StatsD, custom counters | `nfr/OBSERVABILITY.md` |
| Configuration | Env loading, config files, feature flags | `VALUE-REGISTRY.md` |
| Background processing | Queue workers, cron jobs, schedulers | `workflows/` |

### `[ORPHAN]` — code with no discernible purpose

| Subtype | Detection | Severity |
|---|---|---|
| Orphan endpoint | Reachable endpoint whose behaviour maps to no business concept | MEDIUM |
| Orphan feature | Working code with no traceable business need | MEDIUM |
| Orphan test | Test that exercises no identified behaviour | LOW |
| Orphan config | Configuration key with no consumer | LOW |

Orphans go to the user at Checkpoint 1: they either get a name (and become a requirement) or get discarded.

### `[IMPLICIT-RULE]` — undocumented business logic

The highest-value finding in a reverse-engineering run: these are the rules the codebase enforces and nothing states.

| Subtype | Detection | Severity |
|---|---|---|
| Hidden validation | Conditional enforcing a business rule under a non-explanatory name | HIGH |
| Magic threshold | Numeric threshold in business logic with no explanation | HIGH |
| Implicit state rule | Transition guard with no documented reason | MEDIUM |
| Implicit ordering | Sequential operations where the order matters silently | MEDIUM |
| Conditional pricing | Price or discount logic with no documentation | HIGH |
| Access control rule | Permission check with no documented authorization matrix | HIGH |

Each one becomes an RN row in `spec/CLARIFICATIONS.md`, and a requirement whenever the behaviour is user-visible.

---

## 7. Severity

| Severity | Criteria |
|---|---|
| CRITICAL | Business logic at risk, data integrity threat, security gap |
| HIGH | Significant maintainability issue, missing error handling, undocumented business rule |
| MEDIUM | Code quality issue, workaround in place, dead code with side effects |
| LOW | Minor quality issue, cosmetic, small duplication |
| INFO | Pattern documented for completeness |

**Modifiers** (apply to the base severity above):

- In a critical path — auth, payments, data writes: **+1**
- Covered by tests: **−1** (the risk is managed)
- Untouched for more than a year: **+1**
- Sole contributor no longer in the history: **+1**
- Modified in the last 30 days, or touched by several contributors: no change

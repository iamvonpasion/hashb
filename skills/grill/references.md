# Grill References

Detailed per-pass checklists, scenario templates, and scoring rubrics referenced
by `SKILL.md`.

---

## Pass A — Simplicity Audit: Detection Signals

Every Simplicity finding MUST cite one of these countable signals. Vibes-only
findings ("this feels overcomplicated") are not acceptable.

### Unjustified Abstraction Signals

| Signal | Detection | Severity |
|--------|-----------|----------|
| Interface with 1 implementation | Grep interface declarations, count implementers | MEDIUM |
| Abstract class with 1 concrete subclass | Grep `abstract class` + count subclasses | MEDIUM |
| Generic type with 1 concrete instantiation | Find `Foo<T>` usages, count distinct `T` | LOW–MEDIUM |
| Plugin/strategy system with 1 plugin | Find plugin registry, count registrations | MEDIUM–HIGH |
| Factory returning a single concrete type | Trace factory return paths | MEDIUM |
| Builder pattern with 1 construction path | Count `.build()` call sites and configurations | LOW–MEDIUM |
| Repository interface over a single ORM | Check if repo adds logic or just forwards | MEDIUM |
| Adapter with 1 adaptee | Count adapter implementations | MEDIUM |
| Wrapper function that only calls through | Read function body, check for transformation | LOW |
| DI container for 5 or fewer services | Count registered services | LOW |
| Configuration option with 1 value everywhere | Grep config reads, check for variation | LOW |
| Abstraction layer preserved "for future flexibility" | Comments + 0 actual variation | MEDIUM |

### Speculative Extensibility Signals

| Signal | Detection |
|--------|-----------|
| Extension points with 0 extensions | Event system with 0 subscribers, hook system with 0 hooks |
| Versioned APIs with 1 version | `/v1/` only, no migration path exercised |
| Feature flags that have been "on" or "off" for > 6 months | Git blame on flag introduction |
| "TODO: make this pluggable" comments + no follow-through | Grep TODO/FIXME for abstraction language |

### Layer Cake Signals

| Signal | Detection |
|--------|-----------|
| More than 5 layers between entry point and business logic | Trace a request end-to-end |
| DTO/Model/Entity/ViewModel for the same concept | Grep for `*Dto`, `*Model`, `*Entity` on the same noun |
| Service → Service → Service chain with no logic | Read each service, check for transformation |
| Controller that only forwards to a service that only forwards to a repository | Inspect orchestration |

### Senior Engineer Test (Karpathy-inspired)

For each module, ask: "Would a senior engineer joining today say this is
overcomplicated for what it does?"

This test frames the question. The countable signal above grounds the answer.
A finding is credible ONLY when both are present:

- Senior Engineer Test: YES (overcomplicated)
- Countable signal: at least one from the tables above

---

## Pass B — Structural Fitness: Scenario Templates

Quality-attribute scenarios differ by stack. Select 3–5 relevant scenarios based
on Project Profile.

### SaaS / Multi-Tenant Web App

| Scenario | What to Trace |
|----------|---------------|
| Tenant isolation | Can tenant A's data ever reach tenant B? Trace every DB query and cache access |
| Scale to 10× tenants | Per-tenant overhead — connections, memory, config |
| Per-tenant customization | How hard to add a tenant-specific rule without a fork? |
| Billing/usage metering | Can usage be counted reliably per tenant? |
| Compliance (GDPR/SOC2) | Can a tenant's data be fully exported or deleted? |

### API-Only / Backend Service

| Scenario | What to Trace |
|----------|---------------|
| Backward-compatible API change | How are versioning and deprecation handled? |
| Request tracing end-to-end | Correlation IDs through logs, metrics, traces |
| Partial failure handling | What happens when 1 downstream dep is slow or down? |
| Rate limit / abuse handling | Is there a chokepoint, or can a client saturate the service? |
| Schema evolution | How are DB migrations coordinated with deploys? |

### CLI / Developer Tool

| Scenario | What to Trace |
|----------|---------------|
| Cold-start time | How fast from invocation to first output? |
| Error ergonomics | When the user makes a mistake, how actionable is the error? |
| Cross-platform support | Does it work on Linux, macOS, Windows? |
| Scriptability | Can the output be piped into another tool? |
| Offline operation | Does it need network for common operations? |

### Library / SDK

| Scenario | What to Trace |
|----------|---------------|
| Semver discipline | What constitutes a breaking change, and is it enforced? |
| Integration surface | How many touch points must a consumer understand? |
| Bundle size / runtime cost | What does importing this cost a consumer? |
| Tree-shakability / optional features | Can consumers opt out of unused parts? |
| Documentation-to-API ratio | Does every public symbol have a doc comment with an example? |

### Mobile App

| Scenario | What to Trace |
|----------|---------------|
| Offline-first | What works without network? |
| Battery impact | Background work, polling, radio usage |
| Cold-start time | Launch-to-interactive on low-end devices |
| App size | Binary size, asset sizes |
| Update rollout | How are critical fixes delivered — store review vs OTA? |

### Embedded / Resource-Constrained

| Scenario | What to Trace |
|----------|---------------|
| Memory ceiling | Is there a max allocation, and is it enforced? |
| Real-time guarantees | Are critical paths bounded in time? |
| Firmware update safety | Can a failed update brick the device? |
| Power profile | Sleep modes, peripheral wake-up |

### Scoring a Scenario

| Support | Meaning |
|---------|---------|
| WELL | Architecture directly supports this scenario — no refactor needed |
| POOR | Supports with significant workarounds or caveats — friction but tractable |
| NONE | Does not support — would require architectural change |

### Sensitivity / Trade-off Points

For each scenario marked POOR or NONE, identify:

- **Sensitivity point:** A small change here cascades — e.g., "adding a new tenant
  requires touching 5 unrelated modules."
- **Trade-off point:** Improving this hurts something else — e.g., "tenant isolation
  improvement would slow query performance."

---

## Pass C — Coupling / Cohesion: Detection

### Inappropriate Coupling Signals

| Signal | How to Detect |
|--------|---------------|
| Modules coupled, unrelated concerns | Check import graph + read responsibilities — if coupling crosses unrelated domains (auth importing billing), flag |
| Modules decoupled, always changing together | Git log correlation — modules whose files frequently co-change but don't import each other |
| God modules | `/understand` map + `god_nodes` MCP if available |
| Circular dependencies | Graph cycle detection |
| Inverted dependency direction | Domain layer importing from infrastructure layer |
| Cross-module DB access | Grep for direct table access outside the owning module |
| Shotgun surgery | A single logical change that requires edits in > 5 files across > 3 modules |

### Cohesion Signals

| Signal | How to Detect |
|--------|---------------|
| Module doing too many unrelated things | Count distinct responsibilities in the module's public API |
| Dead code in an otherwise active module | Grep for unused exports |
| Feature envy (module A uses module B's internals extensively) | Import usage density between modules |

---

## Pass D — Technology Fit: Evaluation

### Over-Powered Tool Signals

| Signal | Example |
|--------|---------|
| Enterprise framework for a small app | Spring / Nest for a 10-endpoint CRUD |
| Microservices for a team of 2 | Distribution overhead without distribution benefit |
| Event-sourcing for a CRUD domain | Eventual consistency cost without audit/replay needs |
| Kubernetes for an app with 1 replica | Cluster overhead for a single process |

### Under-Powered Tool Signals

| Signal | Example |
|--------|---------|
| SQLite for multi-tenant SaaS at scale | Single-writer bottleneck |
| Cron for critical scheduled work | No failure isolation, no retries |
| Monolithic deploys for a team of 50 | Change coordination becomes the bottleneck |
| In-memory cache for data that must survive restarts | State loss on every deploy |

### Deprecated / Abandoned Dependency Check

If Context7 MCP is available, query deprecation status for each major dependency.
Max 3 queries.

| Finding | Severity |
|---------|----------|
| Dependency is archived / unmaintained | HIGH |
| Dependency has a known successor and active migration path | MEDIUM |
| Dependency is > 2 major versions behind | MEDIUM |
| Dependency has critical unpatched CVEs | CRITICAL |

### Meta-Complexity Tax

| Signal | What it suggests |
|--------|-------------------|
| Build config larger than any single module | Toolchain complexity > problem complexity |
| CI pipeline takes > 30 min for a small codebase | Investment misaligned with return |
| 3+ package managers in the same repo | Accidental complexity |

---

## Pass E — Assumption Freshness: Detection

### With ADRs (Preferred Mode)

For each ADR, extract the **Context** section and check each stated constraint
against current reality.

| Constraint Type | Check |
|-----------------|-------|
| Scale assumption | Current traffic / data volume vs. stated |
| Team size assumption | Current team size vs. stated |
| Tech landscape assumption | Is the referenced library / pattern / cloud still dominant? |
| Business model assumption | Is the current business model the same? |
| Deployment environment | On-prem vs cloud vs serverless — still the same? |

Output per ADR:

| ADR | Stated Context | Current Reality | Still Valid? |
|-----|----------------|-----------------|--------------|
| ADR-005: choose Postgres | 100 users, single-tenant | 50k users, multi-tenant | PARTIAL — needs revisit |

### Without ADRs (Degraded Mode)

Mark pass as: "No ADRs — assumptions inferred from code structure, speculative."

Infer likely assumptions from:
- Entry point count (suggests expected input channels)
- DB schema (suggests expected data shape)
- Auth model (suggests expected user/tenant structure)
- Deploy artifacts (suggests expected deployment model)

Output is lower-confidence and must be labeled as such in the final report.

---

## Phase 3 — Fowler Debt Quadrant Classification

Every finding maps to one of four quadrants:

### Deliberate + Prudent

- "We chose this trade-off knowingly and it was the right call for our constraints."
- Often surfaces findings that the user dismisses during the Phase 2 gate.
- Action: Accept. Document as ADR if not already. No refactoring needed.

### Deliberate + Reckless

- "We chose this trade-off knowingly, and it was shortsighted."
- Example: skipped tests to ship, now can't refactor safely.
- Action: Justify (if still defensible) or plan remediation.

### Inadvertent + Prudent

- "We didn't know better at the time, but we've since learned."
- Most common for brownfield findings.
- Action: Fix when it becomes blocking — not urgent.

### Inadvertent + Reckless

- "We didn't know better and it was avoidable."
- Example: copy-pasted code that diverged and now has inconsistent bugs.
- Action: Fix soon — each passing day compounds the cost.

### Classification Heuristics

| Signal | Likely Quadrant |
|--------|-----------------|
| ADR exists defending the decision | Deliberate |
| "TODO: fix this later" comments | Deliberate + Reckless (often) |
| Consistent pattern across team | Deliberate + Prudent (if defensible) |
| Inconsistent pattern (some files yes, some no) | Inadvertent |
| Obvious violation of declared architecture | Inadvertent |
| Contested during Phase 2 gate ("we meant to do that") | Deliberate |

---

## Phase 3 — ROI Scoring Rubric

### Effort (rough estimate — validate with team)

| Value | Meaning | Typical Scope |
|-------|---------|---------------|
| S | Small | Hours. 1–3 files, no migration, no user-visible change |
| M | Medium | Days. Multiple modules, some migration, may require coordinated deploy |
| L | Large | Weeks. System-wide change, data migration, phased rollout |

### Value

| Value | Meaning |
|-------|---------|
| H | High | Unblocks meaningful delivery OR removes CRITICAL risk OR eliminates recurring pain |
| M | Medium | Reduces friction, removes moderate risk, improves onboarding or readability |
| L | Low | Polish — nice to have, no urgent driver |

### ROI Rank

Sort primary by Value descending, secondary by Effort ascending. Tie-break by severity.

| Value | Effort | Rank Tier |
|-------|--------|-----------|
| H | S | Top tier — do first |
| H | M | High priority |
| M | S | Quick wins |
| H | L | Strategic investments |
| M | M | Steady-state maintenance |
| L | S | Polish when idle |
| M | L | Defer or break into smaller steps |
| L | M / L | Generally not worth doing |

---

## Phase 3 — Fitness Score Deduction Rubric

Each pass starts at 100. Deductions per finding:

| Severity | Deduction |
|----------|-----------|
| CRITICAL | 25 |
| HIGH | 15 |
| MEDIUM | 8 |
| LOW | 3 |

Floor at 0.

**Assumption Age with no ADRs:** Report as "degraded — no ADRs" rather than a
numeric score. Do not average a speculative score into the overall.

**Overall weighting:**

| Pass | Weight |
|------|--------|
| Simplicity | 25 % |
| Structural | 25 % |
| Coupling | 20 % |
| Technology | 15 % |
| Assumption Age | 15 % (or redistributed if degraded) |

If Assumption Age is degraded, redistribute its 15 % proportionally across the
other four passes.

---

## Severity Classification

| Severity | Definition |
|----------|------------|
| CRITICAL | Blocks delivery now, introduces security/data risk, or compounds rapidly |
| HIGH | Significant friction or risk, should be addressed in current cycle |
| MEDIUM | Real issue, not urgent, plan into next cycle |
| LOW | Worth noting, not worth prioritizing independently |

---

## Strengths: What to Look For

A deficit-only report loses credibility. Actively look for what the architecture
gets right. Minimum 2 strengths, evidence-based.

| Strength Type | Example |
|---------------|---------|
| Clean module boundaries | "Billing and auth are cleanly separated — 0 cross-imports" |
| Appropriate simplicity | "Domain logic is straightforward — no speculative abstractions detected" |
| Good test coverage at boundaries | "All public API entry points have integration tests" |
| Honest technology choices | "Stack matches problem complexity — no over-powered tools" |
| Fit-for-purpose deployment | "Deploy model matches scale and team size" |
| Living documentation | "ADRs are current and reference actual code decisions" |

---

## Skipping Passes

A pass may be skipped only with explicit reason in the report:

| Condition | Pass Skipped | Label |
|-----------|--------------|-------|
| No ADRs found | Assumption Freshness | "SKIPPED or DEGRADED: no ADRs — ran in infer-from-code mode" |
| Single-module repo | Coupling / Cohesion | "SKIPPED: single module, no inter-module coupling to evaluate" |
| No Project Profile | Structural Fitness | "DEGRADED: no Profile — used generic scenario set, results less precise" |
| Hobby / throwaway repo | All of B, C, D, E | "Grill not appropriate for this repo type — ran Simplicity pass only" |

Never silently skip. Absent reasoning = not done.

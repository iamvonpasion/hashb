# Eng — Reference Tables & Templates

> Extracted from `SKILL.md` to keep the orchestration file under 500 lines.
> Referenced by Phase 3, Phase 4, and Required Outputs sections.

---

## Phase 3 — Implementation Review Details

### 3A. Premise & Approach

| Question | Answer |
|----------|--------|
| **Right problem?** Most direct path to the outcome? | {2-3 sentences} |
| **What exists?** Sub-problems mapped to existing code. Rebuilds flagged. | {2-3 sentences} |
| **Do-nothing baseline.** What if we ship nothing? | {2-3 sentences} |

**Implementation Alternatives:**

| | Approach A: {name} | Approach B: {name} | Approach C: {name} |
|---|---|---|---|
| **Summary** | {1-2 sentences} | {1-2 sentences} | {1-2 sentences} |
| **Effort** | S / M / L | S / M / L | S / M / L |
| **Risk** | Low / Med / High | Low / Med / High | Low / Med / High |
| **Reuses** | {existing code} | {existing code} | {existing code} |
| **Pros** | {bullets} | {bullets} | {bullets} |
| **Cons** | {bullets} | {bullets} | {bullets} |

> One must be "minimal viable" (smallest diff). One must be "ideal architecture".

**Scope Check:**

| Check | Question | Result |
|-------|----------|--------|
| Minimum viable | Smallest change that achieves the goal? | {answer} |
| Complexity smell | >8 files or >2 new classes? | {yes/no — challenge if yes} |
| TODOS.md cross-ref | Block, unlock, or duplicate existing TODOs? Check for `#N` task IDs and `(depends:)` from `/spec decompose` | {answer} |

If complexity smell triggers, recommend scope reduction.

**Implementation Order** (skip if single-tier):

| Order | Tier | What to build | Why this order |
|-------|------|---------------|----------------|
| 1 | {tier} | {what} | {why first} |
| 2 | {tier} | {what} | {depends on 1} |
| 3 | {tier} | {what} | {depends on 2} |

**`/eng hold` only:** Scope is locked. Challenge complexity harder. Skip expansion suggestions.

```
APPROACH VERDICT: {CLEAR | ISSUES | BLOCKED} ──────────────────

  Approach    {chosen approach}
  Scope       {accepted / reduced}
  Order       {sequence}

────────────────────────────────────────────────────────────────
```

> If approach needs a user decision (multiple viable options, scope reduction needed),
> pause here. Otherwise, continue streaming.

### 3B. Architecture & Security

| Area | Status | Notes |
|------|--------|-------|
| System diagram | {done} | — |
| Data flows | {ok / issues} | {brief} |
| State machines | {ok / n/a} | {brief} |
| Coupling | {ok / concern} | {brief} |
| Scaling (10x) | {ok / breaks} | {brief} |
| Rollback plan | {exists / missing} | {brief} |
| Auth & access | {ok / gaps} | {brief} |
| Input validation | {ok / gaps} | {brief} |
| Secrets handling | {ok / issues} | {brief} |
| New dependencies | {ok / concern} | {brief} |
| Runtime wiring | {ok / needs changes} | {registrations, config, env vars} |
| Failure scenarios | {covered / gaps} | {brief} |

> **Runtime wiring check:** Does this feature require changes outside its own
> code to function at runtime? Check: service registrations (DI, routes,
> middleware, event subscriptions), pipeline or protocol configuration,
> environment variables, config file additions. Features that need external
> wiring but don't get it will compile and pass unit tests but fail at runtime.

**ASCII System Diagram** (mandatory): Show new components and relationships to existing ones.

**Issues** (expand only for non-ok rows):

```
! ISSUE #{N}: {area} — {one-line summary}
  ─────────────────────────────────────────────────
  Problem     {what's wrong}
  Impact      {what breaks}
  Fix         {recommendation}
  Severity    CRITICAL GAP | WARNING | INFO
```

### 3C. Error Map & Code Quality

**Error Map:**

| Codepath | Failure | Exception | Caught? | User Sees |
|----------|---------|-----------|---------|-----------|
| {path} | {failure} | {type} | Y | {message} |
| {path} | {failure} | {type} | **N <- GAP** | 500 |

> Any row with **Caught=N** and **User Sees=500/silent** -> **CRITICAL GAP**

**Error Handling Patterns:**

| Pattern | Verdict |
|---------|---------|
| Catch-all (`rescue StandardError`, `except Exception`) | Smell — name specific exceptions |
| Swallow and log | Almost never acceptable |
| LLM/AI calls | Must handle: malformed, empty, refusal, hallucinated JSON — each distinct |
| Every caught error must | Retry with backoff, degrade gracefully, OR re-raise with context |

**Code Quality:**

| Check | Status | Notes |
|-------|--------|-------|
| DRY violations | {ok / found} | {file:line refs} |
| Naming | {ok / issues} | {specifics} |
| Over-engineering | {ok / found} | {what} |
| Under-engineering | {ok / found} | {what} |
| Stale diagrams | {ok / found} | {which files} |

### 3D. Tests

**Coverage Map:**

| Category | Items | Happy Path | Failure Path | Edge Cases |
|----------|-------|------------|--------------|------------|
| New UX flows | {list} | {covered?} | {covered?} | {covered?} |
| New data flows | {list} | {covered?} | {covered?} | {covered?} |
| New codepaths | {list} | {covered?} | {covered?} | {covered?} |
| Background jobs | {list} | {covered?} | {covered?} | {covered?} |
| New integrations | {list} | {covered?} | {covered?} | {covered?} |
| Error paths | {from error map} | — | {covered?} | {covered?} |

**Edge Cases:**

| Scenario | Covered? |
|----------|----------|
| Double-submit | {Y/N} |
| Navigate-away mid-action | {Y/N} |
| Stale state | {Y/N} |
| Retry while in-flight | {Y/N} |
| Zero results | {Y/N} |
| 10k results | {Y/N} |
| Results change mid-page | {Y/N} |
| Partial job completion | {Y/N} |
| Duplicate job execution | {Y/N} |

> **Confidence check:** What test would make you confident shipping at 2am Friday?

### 3E. Performance & Deployment

| Check | Status | Notes |
|-------|--------|-------|
| N+1 queries | {ok / found} | {where} |
| Missing indexes | {ok / found} | {which} |
| Unbounded result sets | {ok / found} | {where} |
| Top 3 slowest codepaths | {acceptable?} | {which} |
| Migration backward-compat | {yes / no} | {detail} |
| Zero-downtime deploy | {yes / no} | {detail} |
| Deploy order | {simple / ordered} | {sequence} |
| Old+new code simultaneously | {safe / breaks} | {what} |

### Review Findings Summary

After streaming all sections, present a consolidated findings summary:

```
REVIEW FINDINGS ─────────────────────────────────────────────────

  Architecture       {N} issues ({N} critical)
  Errors & Quality   {N} issues, {N} CRITICAL GAPS
  Tests              {N} coverage gaps
  Performance        {N} issues

  CRITICAL (must resolve before TDD):
  ─────────────────────────────────────────────────
  {numbered list of critical gaps, or "None"}

  DECISIONS NEEDED:
  ─────────────────────────────────────────────────
  {numbered list of unresolved choices, or "None — all clear"}

─────────────────────────────────────────────────────────────────
```

> If there are decisions needed, pause for user input.
> If all clear, continue streaming to Phase 4.

---

## Phase 4 — TDD Plan Details

### 4A. Test Strategy

Determine the TDD approach based on scope:

| Scope | TDD Approach |
|-------|-------------|
| Single module, well-defined behavior | Classic TDD — unit tests first per function |
| Multi-module feature | ATDD — acceptance tests from spec criteria first, then unit tests per module |
| API or service | Contract tests first (request/response shapes), then implementation |
| UI feature | Component behavior tests first (from `/design` states), then implementation |
| Bug fix | Regression test first (reproduces the bug), then fix |

### 4B. Test-First Execution Order

Map the implementation order (from Phase 3) to a test-first sequence.
For each tier, define what tests to write **before** the implementation code.

| Order | What to test first | Test type | Derives from |
|-------|-------------------|-----------|--------------|
| 1 | {acceptance test from spec criteria} | Integration / E2E | Spec FR-{N}, AC-{N} |
| 2 | {contract test for new API/service} | Contract | Phase 3, 3B |
| 3 | {unit test for core business logic} | Unit | Phase 3, 3A approach |
| 4 | {unit test for error handling} | Unit | Phase 3, 3C error map |
| 5 | {edge case tests} | Unit | Phase 3, 3D edge cases |

**Rules:**
- Every row in the Phase 3 coverage map MUST have a corresponding test-first entry
- Tests derive from spec acceptance criteria (if `/spec` was run) or from the
  implementation review's coverage map
- Each test must be writable **without** the implementation existing — test the
  interface, not the internals
- Tests for error paths are not optional — include every CRITICAL GAP from 3C
- **No placeholders.** Every test entry must name the specific function/endpoint/component
  under test, the exact input scenario, and the expected outcome. "Test user creation"
  is not acceptable — "Test POST /users with valid email returns 201 and user object
  with id" is. The implementer should be able to write the test from this entry alone

### 4C. Red-Green-Refactor Checkpoints

Define explicit checkpoints where the implementer should verify the TDD cycle:

```
TDD CHECKPOINTS:
  CP-1: {acceptance tests written and RED}    -> verify: tests fail for the right reason
  CP-2: {tier 1 implemented, tests GREEN}     -> verify: minimal code to pass, no extras
  CP-3: {refactor tier 1}                     -> verify: tests still GREEN after cleanup
  CP-4: {tier 2 tests written and RED}        -> verify: tests fail for the right reason
  CP-5: {tier 2 implemented, tests GREEN}     -> verify: all tests pass including tier 1
  ...
```

### 4D. Test Infrastructure

| Need | Exists? | Action |
|------|---------|--------|
| Test runner configured | {yes / no} | {skip / set up} |
| Test database / fixtures | {yes / no / n/a} | {skip / create} |
| Mock server for external APIs | {yes / no / n/a} | {skip / create} |
| Component test harness | {yes / no / n/a} | {skip / create} |
| CI pipeline runs tests | {yes / no} | {skip / add} |

---

## Required Output Templates

### Failure Modes Registry (from 3C)

| Codepath | Failure | Caught? | Tested? | User Sees |
|----------|---------|---------|---------|-----------|
| {path} | {failure} | Y/N | Y/N | {what} |

> Any row with **Caught=N + Tested=N** -> **CRITICAL GAP**

### TDD Execution Plan (from Phase 4)

| Order | Test (RED) | Implement (GREEN) | Refactor | Checkpoint |
|-------|-----------|-------------------|----------|------------|
| 1 | {what test to write} | {what code to write} | {cleanup} | CP-{N} |

### Open Items

| Category | Item | Rationale |
|----------|------|-----------|
| Not in scope | {deferred work} | {one-line why} |
| Unresolved | {unanswered decision} | {never silently default} |

---
name: eng
description: >
  Engineering — scope, architecture decisions, implementation review, TDD planning.
  Use when planning how to build: scoping work, making architecture decisions, reviewing implementation approach, producing TDD execution plans.
  Modes: /eng (auto-detect), /eng arch (architecture), /eng impl (implementation), /eng hold (scope lock).
argument-hint: "[\"arch\" or \"impl\" or \"hold\"]"
---

# Eng

Scope the work, make architecture decisions if needed, then review the
implementation plan with TDD enforcement. Stream output continuously —
gate only at decision points.

**This is read-only. No code changes.**

---

## When to Use

| Signal | Mode |
|--------|------|
| Greenfield project or major subsystem | `/eng` or `/eng arch` |
| Multiple interdependent decisions (>3) | `/eng arch` |
| After `/research`, need to decide | `/eng arch` |
| Feature work, architecture settled | `/eng` or `/eng impl` |
| Reviewing an existing implementation plan | `/eng impl` |
| Making an existing plan bulletproof | `/eng hold` |

**Skip when:** Trivial change (typo, config value, single-line fix) where scope is obvious, no architecture decisions exist, and no TDD plan is needed. Use `/quick eng` instead, or just implement directly.

---

## Presentation Rules

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Complexity Routing

Assess complexity at the start and adjust depth:

| Complexity | Signal | Behavior |
|------------|--------|----------|
| **Small** | Single file, clear fix, ≤3 files touched | Combine scope + review + TDD into one response. 1 gate (final TDD plan confirmation) |
| **Medium** | 3-8 files, known architecture, standard patterns | Stream phases continuously. 2 gates (after scope+approach, after TDD plan) |
| **Large** | Greenfield, 8+ files, architecture decisions needed, new domain | Full flow with arch decisions. 2 gates + per-decision confirmations for arch |

---

## Phase Roadmap (present this first)

| Phase | Name | When it runs | What happens |
|-------|------|-------------|-------------|
| 0 | Scope | Always | Identify system, constraints, determine mode, list what needs deciding |
| 1 | Architecture Decisions | If arch decisions needed | Walk decisions one at a time, validate, produce ADRs |
| 2 | Cross-Cut Validation | After Phase 1 only | Validate all decisions together, risk register |
| 3 | Implementation Review | Always | Single-pass review: premise, approach, architecture, errors, tests, performance |
| 4 | TDD Plan | Always | Test-first strategy: what tests to write before implementation, in what order |
| 5 | Summary | Always | Combined output, ADRs + findings + TDD plan, next steps |

> Phases 1-2 are **conditional** — skipped automatically in `/eng impl` mode
> or when Phase 0 determines no architecture decisions are needed.
>
> **Routing after Phase 0:**
> - Architecture decisions needed? → **Proceed to Phase 1**
> - No architecture decisions? → **Skip to Phase 3**
> - `/eng impl` mode? → **Skip to Phase 3**
> - `/eng hold` mode? → **Skip to Phase 3** (scope lock-down only)

---

## Before You Start

### Read Upstream Artifacts

Check for `/spec` and `/design` handoff summaries in the conversation context:
- **Spec handoff** → problem statement, user stories, requirements, acceptance criteria,
  constraints, assumptions. Read the full spec phases for detail.
- **Design handoff** → screens, components, states, flows, accessibility requirements.
  Read the full design phases for wireframes and state machines.
- **Design handoff compressed?** If the handoff contains only counts
  (e.g., "3 new components") without the component enumeration, error scenario table,
  or data flow matrix, warn before proceeding — implementation may be incomplete.

If no upstream artifacts exist, gather requirements from the user directly.

### Spec Challenge Gate

While reading the spec, actively look for flaws. If any of these are found,
raise a **Spec Challenge** before proceeding to Phase 0:

| Flaw type | Example |
|-----------|---------|
| Contradictory requirements | "Must be real-time" + "batch process nightly" |
| Missing edge case | No handling for zero-quantity items |
| Technically impossible | "Sub-millisecond response with full table scan" |
| Ambiguous acceptance criteria | "Should be fast" without a measurable target |
| Scope gap | Spec says multi-tenant but no AC tests tenant isolation |

**Spec Challenge format:**

```
⚠ SPEC CHALLENGE
──────────────────────────────────────────────
  Issue:    {what's wrong — specific, with evidence}
  Impact:   {what happens if we build this as-is}
  Options:
    A) Fix spec — {what needs to change}
    B) Proceed with known risk — {the risk accepted}
    C) Descope — {what to cut}

  Recommendation: {A, B, or C + one sentence why}
──────────────────────────────────────────────
```

> **STOP.** User must resolve the challenge before /eng proceeds.
> Multiple challenges? Present them one at a time.

If no flaws found, proceed silently — don't report "spec looks fine."

### Read Project Profile

Check the consumer's `CLAUDE.md` for a **Project Profile** section. If present,
use it to adapt review checks throughout all phases:
- Stack → which framework rules and patterns to validate against
- Architecture → what coupling and boundary checks matter
- Testing → what test runner and patterns to expect
- Deploy → what deployment constraints to consider
- MCP Servers → Context7 is required for doc verification; additional servers available

### Scan for Prior Learnings

Check `.retro/` for retro reports mentioning the affected codebase area.
If recurring issues exist, flag them as known risks in Phase 0.

Check Claude Code memories for relevant project/feedback memories from prior retros.
Recurring patterns flagged by `/retro` should inform scope decisions and risk assessment.

### Gather Context

```bash
git log --oneline -15
# Branch/base detection — see skills/shared/preflight.md
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
if command -v gh >/dev/null 2>&1; then
  BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null \
    || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
    || echo "main")
else
  echo "⚠ gh CLI not found — defaulting BASE to 'main'"
  BASE="main"
fi
echo "BRANCH: $BRANCH  BASE: $BASE"
git diff $(git merge-base HEAD $BASE)..HEAD --stat
grep -r "TODO\|FIXME\|HACK" -l --exclude-dir={node_modules,vendor,.git,dist,build} . | head -20
```

Read if they exist: `CLAUDE.md`, `TODOS.md`, `docs/designs/`.

<<<<<<< HEAD
### Task Pickup (decomposed features)

If TODOS.md contains decomposed tasks (has `### Feature:` sections with `#N`
task IDs), determine which task this `/eng` session is for:

1. **User specified a task:** User said "/eng task #3" or "/eng #3" or
   referenced a specific task → use that task. Verify its dependencies are met
   (all `depends:` tasks are `[x]`). If dependencies are NOT met, warn:

   ```
   ⚠ TASK #3 BLOCKED
   ─────────────────────────────────────────────────
   Depends on: #1 (incomplete), #2 (complete)
   → Work on #1 first, or override with "proceed anyway"
   ─────────────────────────────────────────────────
   ```

2. **User didn't specify a task:** Auto-detect the next available task:
   - Find the lowest-numbered `- [ ]` task whose dependencies are all `[x]`
   - Present it for confirmation:

   ```
   TASK PICKUP ─────────────────────────────────────────────────────

     Feature       {name}
     Progress      {done}/{total} tasks complete
     Next task     #{N} — {description}
     Dependencies  {all met | list of blocking tasks}

     ➤ Proceeding with task #{N}. Say "skip" to pick a different task.

   ─────────────────────────────────────────────────────────────────
   ```

3. **All tasks complete:** If all tasks in the feature are `[x]`:

   ```
   ✓ ALL TASKS COMPLETE — Feature: {name}
     Consider /retro to capture learnings.
   ```

4. **TODOS.md exists but no decomposed tasks:** Proceed normally — TODOS.md
   may contain standalone items unrelated to decompose.

**When working a decomposed task:** The task's Goal and AC from TODOS.md
become the requirements for this `/eng` session. Do NOT re-derive requirements —
the task IS the mini-spec. Carry the task ID through all phases so `/ship` can
mark it complete.

**Scope lock:** When `/eng` is working a decomposed task, scope is locked to that
task. If the review reveals work that belongs to a different task, note it as
"out of scope for #{N}, covered by #{M}" rather than expanding scope.

### Graph-Aware Scoping (if code-review-graph MCP is available)

If the consumer's Project Profile lists `code-review-graph`, query it to
inform the implementation review:
- **Blast radius** — for files being modified, identify all callers,
  dependents, and tests that will be affected by the change
- **Test coverage map** — feed into Phase 4 (TDD Plan) to identify which
  existing tests cover the affected code and where gaps exist
- **Dependency graph** — verify that the planned changes respect module
  boundaries and dependency direction (feeds Phase 3, section 3B)

If code-review-graph is not available, derive these from manual code reading.

### Verify with Context7

When the engineering plan involves libraries, frameworks, or APIs, query Context7
to verify assumptions before making architecture decisions:

1. **Resolve library IDs** — call `resolve-library-id` for each key library
   in the scope. Confirm the latest stable version matches what the
   codebase uses or intends to use.
2. **Spot-check capabilities** — call `query-docs` for any library capability
   that is load-bearing in the architecture (e.g., "Does library X support
   streaming responses?" or "Does ORM Y handle composite primary keys?").

**Rules:**
- Max 2 `resolve-library-id` calls during context gathering (not a research phase).
- Max 2 `query-docs` calls during context gathering.
- If a capability assumption cannot be verified, flag it as an **Unknown** in
  the scope card (Phase 0, row 5).
- Do NOT do full library evaluation here — that is `/research`'s job. This is
  a quick verification pass.

---

## Phase 0 — Scope

### Mode Detection

If mode wasn't specified by the user, determine it:

| Signal | Mode |
|--------|------|
| No codebase exists yet | Architecture (`/eng arch`) |
| User asking about system design, tech selection | Architecture |
| Multiple technology decisions unmade (>3) | Architecture |
| Codebase exists, architecture decisions settled | Implementation (`/eng impl`) |
| User provides specific files/branches/codepaths | Implementation |
| Ambiguous | Ask the user |

### Scope Card

| # | Step | What to answer | Output |
|---|------|----------------|--------|
| 1 | System | What system/subsystem? | System name |
| 2 | Constraints | What's already decided? (mandated tech, existing systems) | Constraint list |
| 3 | Scale | Users, data volume, team size | Scale parameters |
| 4 | Decisions | What needs deciding? (architecture and/or implementation) | Numbered list |
| 5 | Known risks | Recurring issues from retros? Prior review findings? | Risk list or "none" |

Present the scope card:

```
ENG SCOPE ───────────────────────────────────────────────────────

  System        {name}
  Mode          {arch | impl | auto}
  Active task   #{N} — {description} (from TODOS.md)   ← if decomposed task
  Constraints   {what's locked in}
  Scale         {targets}
  Known risks   {from retros, or "none"}

  DECISIONS / REVIEW ITEMS
  ─────────────────────────────────────────────────
  1.  [{topic}]  {decision or review item}
  2.  [{topic}]  {decision or review item}

─────────────────────────────────────────────────────────────────
```

> **GATE 1.** Confirm scope, mode, and decision/review list with the user.

**After user confirms, state the route explicitly:**

```
ROUTE: {Architecture decisions needed → Phase 1 | No arch decisions → Skip to Phase 3}
```

---

## Phase 1 — Architecture Decisions (conditional)

**Skip if:** `/eng impl` mode, or Phase 0 determined no arch decisions needed.

Process decisions **one at a time** in the agreed order.

### Context7 Verification (per decision)

For any architecture decision that depends on library or API capabilities:

1. Call `resolve-library-id` to confirm the library exists and get the latest
   stable version.
2. Call `query-docs` with the specific capability question (e.g., "Does
   Next.js App Router support parallel routes with shared layouts?").
3. Include the Context7 source version in the decision card's **Context** field.

If Context7 lacks the library, note it and recommend `/research` for that
specific decision before committing.

**Limits:** Max 3 `resolve-library-id` + 3 `query-docs` per architecture
decision. This is verification, not research.

### Per-Decision Card

```
DECISION #1: {title} ────────────────────────────────────────────

  Context       {why — 1 sentence}
  Depends on    {prior decisions}

| Option | Summary      | Standards  | Trade-off  |
|--------|--------------|------------|------------|
| A      | {1 sentence} | 12F, OWASP | {cost}     |
| B      | {1 sentence} | {which}    | {cost}     |
| C      | {if needed}  | {which}    | {cost}     |

  ➤ RECOMMEND: {letter} — {one sentence why}

  VALIDATION
  ─────────────────────────────────────────────────
  OWASP .............. {pass | violation: ...}
  12-Factor .......... {pass | violation: ...}
  Well-Architected ... {pass | violation: ...}
  Prior decisions .... {consistent | conflict}

─────────────────────────────────────────────────────────────────
```

### After User Decides — Record ADR

```
ADR-{NNN}: {title} ──────────────────────────────────────────────

  Status         Accepted
  Date           {today}
  Context        {why — 1-2 sentences}
  Decision       {what was decided}
  Consequences   {positive} / {trade-off}

─────────────────────────────────────────────────────────────────
```

> Confirm ADR, then move to next decision.

**Batch rule:** Independent, low-impact decisions can be grouped into one
table with individual recommendations. Reserve one-at-a-time for large or
interdependent decisions.

---

## Phase 2 — Cross-Cut Validation (conditional)

**Skip if:** Phase 1 was skipped (no architecture decisions made).

Stream directly after all architecture decisions are confirmed.

### Summary Table

| Check | Question | Result |
|-------|----------|--------|
| Consistency | Do all decisions work together? | pass / issues |
| OWASP | Security gaps across combined architecture? | pass / violations |
| 12-Factor | Cloud-native alignment? | pass / violations |
| Well-Architected | Reliability, performance, cost? | pass / violations |
| Gaps | Any decisions needed but not made? | none / list |
| Risks | Top 3-5 architectural risks | severity + mitigation |

### Detail (expand only if issues found)

```
⚠ ISSUE: {check name} — {one-line summary}
  ─────────────────────────────────────────────────
  Detail      {what's wrong}
  Affected    ADR-{NNN}, ADR-{NNN}
  Fix         {recommendation}
```

### Risk Register

| # | Risk | Severity | Mitigation |
|---|------|----------|------------|
| 1 | {risk description} | High | {plan} |
| 2 | {risk description} | Medium | {plan} |

Continue streaming to Phase 3 (no stop here unless issues need user resolution).

---

## Phase 3 — Implementation Review (always runs)

> In all modes: **no silent scope changes**. Every addition or cut is an
> explicit user decision.

This is a **single-pass review** that produces findings tables. Stream all
sections continuously — do not stop between sections. Present issues inline
and recommend fixes; only stop if a decision is needed from the user.

Stream sections 3A through 3E continuously, then present Review Findings Summary.
See `references.md` for all section templates, tables, and card formats.

**Sections:** 3A Premise & Approach (right problem, alternatives, scope check, implementation order, approach verdict) | 3B Architecture & Security (12-area status table, system diagram, runtime wiring check) | 3C Error Map & Code Quality (error map, handling patterns, quality checks) | 3D Tests (coverage map, edge cases, confidence check) | 3E Performance & Deployment (queries, indexes, deploy safety)

**Review Findings Summary** consolidates all issues with counts and lists critical gaps and decisions needed.

> If there are decisions needed, pause for user input.
> If all clear, continue streaming to Phase 4.

---

## Phase 4 — TDD Plan (always runs)

> **Test-Driven Development is mandatory.** Implementation follows TDD:
> write failing tests first, then implement to make them pass, then refactor.
> This phase produces the test-first execution plan.

See `references.md` for all TDD sub-section templates (4A-4D).

**Sections:** 4A Test Strategy (TDD approach by scope type) | 4B Test-First Execution Order (test sequence mapped from Phase 3, no placeholders rule) | 4C Red-Green-Refactor Checkpoints (CP-N verification gates) | 4D Test Infrastructure (runner, fixtures, mocks, CI)

**Key rules:** Every Phase 3 coverage row needs a test-first entry. Each test must name the specific function/endpoint, input scenario, and expected outcome. Error path tests are not optional.

> **GATE 2.** Confirm TDD plan with the user. This is the execution contract
> for implementation.

---

## Phase 5 — Summary

```
✓ ENG COMPLETE ──────────────────────────────────────────────────

  System       {name}
  Mode         {arch + impl | impl only}
  Branch       {branch} → {base}              ← impl-only mode
  ADRs         {N} produced                    ← arch mode only
  Standards    OWASP, 12-Factor, Well-Architected
  Risks        {N} identified, {N} mitigated   ← arch mode only

  FINDINGS
  ─────────────────────────────────────────────────
  Architecture       {N} issues
  Errors & Quality   {N} issues, {N} CRITICAL GAPS
  Tests              {N} gaps
  Performance        {N} issues
  TDD plan           {N} test-first entries, {M} checkpoints

  CRITICAL
  ─────────────────────────────────────────────────
  Critical gaps      {N} — {list}
  Unresolved         {N} — {list}

  ADR SUMMARY (arch mode only)
  ─────────────────────────────────────────────────
  001  {title} — {decision}

  TDD EXECUTION ORDER
  ─────────────────────────────────────────────────
  1.  {test} → {implementation} → {checkpoint}

  ➤ NEXT: /tdd → /review → /qa → /ship (see Next Step below)

─────────────────────────────────────────────────────────────────
```

> Omit rows marked "arch mode only" or "impl-only mode" when not applicable.

### Swarm Suggestion

If scope is large (>8 files, multiple tiers, or multiple independent areas):

> This scope spans {N} independent areas. Consider `/swarm plan` to
> decompose into parallel streams. The scope card and implementation
> order above can feed directly into stream decomposition.

### Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Standard feature | `/tdd` → `/review` | Execute TDD plan, then review |
| Scope is large (>8 files, multiple areas) | `/swarm plan` | Decompose into parallel streams |
| Implementation complete | `/review` | Peer review before QA |

**Autonomous mode:** If running inside a recipe chain, auto-invoke `/tdd` with
the TDD Execution Order from Phase 4. After `/tdd` completes, auto-invoke `/review`.
If `/review` returns APPROVED, auto-invoke `/qa`. Follow the autonomous gate rules
defined in `recipes.md`.

---

## How to Ask Questions

| Step | What to do |
|------|------------|
| 1. State problem | Concrete, with file:line refs. Assume user hasn't looked at code in 20 min |
| 2. Options | 2-3 lettered options (A, B, C). One sentence + effort + risk each |
| 3. Recommend | Pick one, say why in one sentence. Prefer completeness |
| 4. Label | Issue NUMBER + option LETTER (e.g., "3A", "3B") |

**Escape hatch:** No issues? Say so and move on. Obvious fix? State it and move on.

---

## Required Outputs

Produce these tables in the summary. See `references.md` for templates:
- **Failure Modes Registry** (from 3C) — Caught=N + Tested=N is a CRITICAL GAP
- **TDD Execution Plan** (from Phase 4) — RED/GREEN/Refactor/Checkpoint per order
- **Open Items** — deferred work and unresolved decisions (never silently default)

---

## Rules

| Rule | Why |
|------|-----|
| Read-only | No code changes, no file creation. ADRs inline; user decides where to save |
| Stream by default | Output continuously. Gate only at decision points (scope, TDD plan) |
| 2 gates default | Gate 1: scope confirmation. Gate 2: TDD plan confirmation. Add gates when a user decision is genuinely needed (multiple viable approaches, scope reduction, critical gap requiring user input). Don't gate for confirmation — gate for decisions |
| Single-pass review | Phase 3 is one continuous review, not 5 interactive sub-steps |
| Tables at column 0 | Tables inside card blocks must not be indented — breaks CommonMark rendering |
| Be opinionated | Recommend one option. "It depends" is not engineering |
| Carry context forward | Each section references relevant prior decisions |
| Don't implement | Redirect to implementation after TDD plan is approved |
| TDD is mandatory | Every implementation must have a test-first plan. No exceptions |
| Tests before code | The TDD plan defines what tests to write first. Implementation makes them pass |
| Favor simplicity | When options are close, pick fewer moving parts |
| High-level first | Always show summary table before expanding into detail |
| Tables over walls of text | Tables for structured data. Prose only for context |
| No silent scope changes | Every addition or cut is an explicit user decision |
| Respect Project Profile | Adapt checks to consumer's declared stack and architecture |
| Consume upstream artifacts | If `/spec` or `/design` handoffs exist, use them. Don't re-derive requirements |
| Verify via Context7 | Architecture decisions touching libraries must be spot-checked against Context7 docs. Flag unverifiable assumptions |
| Complexity routing | Small tasks get single-pass output. Large tasks get full flow. Match depth to scope |

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "The architecture is straightforward — skip to implementation" | Straightforward architectures still need scope confirmation and a TDD plan. Skipping scoping creates silent assumptions that surface as bugs. |
| "I'm confident about this API — no need to verify with Context7" | Confidence is not evidence. Training data contains outdated patterns. One Context7 query prevents hours of debugging deprecated APIs. |
| "This doesn't need an ADR — it's an obvious choice" | "Obvious" choices are the ones most often revisited. ADRs prevent future engineers from re-debating settled decisions. |
| "The TDD plan is overkill for this size" | The TDD plan IS the execution contract. Without it, `/tdd` has no roadmap and implementation becomes ad-hoc. |
| "I'll figure out error handling during implementation" | Error paths not planned are error paths not tested. The error map catches the cases that produce production incidents. |
| "We can refactor this later if it doesn't scale" | "Later" refactors are 5-10x more expensive. Architecture decisions made now compound — get them right with data, not hope. |

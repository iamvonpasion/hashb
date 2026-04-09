---
name: spec
description: >
  Product specification — problem discovery, requirements definition, acceptance criteria.
  Use when defining what to build: new features, user stories, requirements, acceptance criteria.
  Modes: /spec (auto-detect), /spec discover (problem exploration), /spec define (requirements),
  /spec decompose (break large goal into sequenced tasks in TODOS.md).
argument-hint: "[topic or \"discover\" or \"decompose\"]"
---

# Spec

Define **what** to build and **why**, before engineering decides **how**.
One skill, adaptive phases. Stream output continuously — gate only at decision points.

**This is read-only. No code changes.**

---

## When to Use

| Signal | Mode |
|--------|------|
| New product idea, unclear problem space | `/spec` or `/spec discover` |
| User research available, needs structuring | `/spec` or `/spec discover` |
| Problem understood, need structured requirements | `/spec` or `/spec define` |
| Existing PRD needs tightening | `/spec define` |
| Translating stakeholder request into actionable spec | `/spec` |
| Large feature, needs breakdown into manageable tasks | `/spec decompose` |
| "This is too big for one session" | `/spec decompose` |

---

## Presentation Rules

See `skills/shared/formatting.md` for presentation rules (progress indicators, discussion chunking, table formatting).

---

## Complexity Routing

Assess complexity at the start and adjust depth:

| Complexity | Signal | Behavior |
|------------|--------|----------|
| **Small** | Single story, clear problem, ≤3 requirements | Combine all phases into one response. 1 gate max (final confirmation) |
| **Medium** | 3-8 stories, known domain, moderate scope | Stream phases continuously. 2 gates (after scope, after requirements+AC) |
| **Large** | Unclear problem, many stakeholders, 8+ stories, new domain | Full flow with discovery. 2 gates + discovery confirmation. **Cascade check:** if scope spans 3+ distinct domains/areas, escalate to Product |
| **Product** | Entire product/app V1, multiple domains/areas (3+), greenfield, or 8+ stories across distinct functional areas | Auto-route to decompose. Never run full Define for product-level scope — it produces a wall of requirements nobody can act on. Decompose first, then each task gets its own mini-spec during `/eng` |
| **Decompose** | Large feature, multi-session, explicit task breakdown request | Phase 0 → Phase D → Phase 4. 2 gates (scope, task manifest) |

---

## Phase Roadmap (present this first)

| Phase | Name | When it runs | What happens |
|-------|------|-------------|-------------|
| 0 | Scope | Always | Identify the problem area, determine mode, list what needs answering |
| 1 | Discovery | If problem space unclear | Explore problem, users, pain points, constraints, opportunities |
| 2 | Define | Unless decompose mode | Structure requirements: user stories, functional/non-functional requirements |
| 3 | Acceptance Criteria | Unless decompose mode | Testable criteria for every requirement, edge cases, out-of-scope |
| D | Decompose | If decompose mode | Break goal into sequenced tasks, write to TODOS.md |
| 4 | Summary | Always | Combined output: spec document or task manifest, handoff for downstream skills |

> Phase 1 is **conditional** — skipped automatically in `/spec define` mode
> or when Phase 0 determines the problem is already well-understood.
>
> Phase D **replaces** Phases 2-3 in decompose mode. It produces tasks, not
> stories/requirements. Each task includes its own mini-spec (goal + AC).
>
> **Routing after Phase 0:**
> - Problem space unclear? → **Proceed to Phase 1**
> - Problem understood, need requirements? → **Skip to Phase 2**
> - `/spec define` mode? → **Skip to Phase 2**
> - `/spec decompose` mode? → **Skip to Phase D**
> - Large feature, needs breakdown? → **Skip to Phase D**
> - **Product-level scope** (V1, 3+ domains, 8+ stories across areas)? → **Skip to Phase D** (auto-route, do not run full Define)

---

## Before You Start

### Read Project Profile

Check the consumer's `CLAUDE.md` for a **Project Profile** section. If present,
use it to understand the stack, architecture, and constraints that will shape
what's feasible.

### Scan for Prior Learnings

Check `.retro/` for retro reports mentioning the affected area.
If recurring user-facing issues exist, flag them as known context in Phase 0.

### Scan Spec Registry

Check for an existing persistent spec registry:

```bash
# Check for spec registry and existing capability specs
ls specs/ specs/**/*.md 2>/dev/null
```

If `specs/` exists, read capability specs relevant to the topic. These are the
source of truth for what the system currently does. If the user is modifying
an existing capability, you'll produce **delta specs** (see Phase 4) instead of
full specs.

If `specs/` doesn't exist, that's fine — Phase 4 will create it if the user
approves.

### Gather Context

```bash
# Check for existing specs, requirements, or design docs
ls docs/ docs/specs/ docs/designs/ docs/requirements/ 2>/dev/null
ls TODOS.md 2>/dev/null
```

Read if they exist: `CLAUDE.md`, `TODOS.md`, `docs/`.

---

## Phase 0 — Scope

### Mode Detection

If mode wasn't specified by the user, determine it:

| Signal | Mode |
|--------|------|
| User describes a vague idea or problem area | Discovery (`/spec discover`) |
| User says "I want to understand..." or "we need to figure out..." | Discovery |
| User provides clear requirements but unstructured | Define (`/spec define`) |
| User has a ticket/brief with acceptance criteria draft | Define |
| User says "break this down", "too big", "multiple sessions" | Decompose (`/spec decompose`) |
| Large feature with existing spec output | Decompose |
| Goal spans multiple domains, areas, features, or concerns (3+) | Decompose (recommend proactively) |
| Entire product/app V1, greenfield project, or multi-domain system | Decompose (auto-route — never run full Define at product scale) |
| 8+ stories identified during scoping, touching distinct functional areas | Decompose (recommend proactively) |
| Ambiguous | Ask the user |

### Scope Card

```
SPEC SCOPE ──────────────────────────────────────────────────────

  Problem area    {name}
  Mode            {discover | define | decompose | auto}
  Complexity      {small | medium | large | product}
  Stakeholders    {who cares about this}
  Users           {who will use this}
  Constraints     {budget, timeline, tech limits}
  Known context   {retro findings, existing docs}

  QUESTIONS TO ANSWER
  ─────────────────────────────────────────────────
  1.  {what needs answering}
  2.  {what needs answering}

─────────────────────────────────────────────────────────────────
```

> **GATE 1.** Confirm scope, mode, and question list with the user.

**After user confirms, state the route explicitly:**

```
ROUTE: {Problem unclear → Phase 1 (Discovery) | Problem understood → Skip to Phase 2 (Define) | Large goal, needs breakdown → Skip to Phase D (Decompose)}
```

---

## Phase 1 — Discovery (conditional)

**Skip if:** `/spec define` mode, or Phase 0 determined the problem is already understood.

Stream all of 1A–1D continuously, then gate at the end.

### 1A. Problem Definition

| Question | Answer |
|----------|--------|
| **What problem are we solving?** | {specific, observable problem — not a solution} |
| **Who has this problem?** | {user segments, personas, roles} |
| **How do they solve it today?** | {current workarounds, competing products, manual processes} |
| **What's the cost of not solving it?** | {user pain, business impact, opportunity cost} |
| **How will we know we've solved it?** | {observable outcome, not feature list} |

### 1B. User Research Synthesis

If user research, feedback, or data exists, structure it:

| Source | Finding | Frequency | Severity |
|--------|---------|-----------|----------|
| {source} | {what users said/did} | {how often} | {impact level} |

If no research exists, note it as a gap:

```
⚠ NO USER RESEARCH AVAILABLE
  Proceeding with assumptions. Flag these for validation:
  1. {assumption about user behavior}
  2. {assumption about problem severity}
  3. {assumption about desired outcome}
```

### 1C. Opportunity Assessment

| Dimension | Assessment |
|-----------|------------|
| **Market/user demand** | {evidence or assumption} |
| **Feasibility** | {technical constraints from Project Profile} |
| **Effort estimate** | S / M / L / XL |
| **Risk** | {what could go wrong} |
| **Dependencies** | {what else needs to exist first} |

### 1D. Problem Statement

Synthesize discovery into a single, clear problem statement:

```
PROBLEM STATEMENT ───────────────────────────────────────────────

  {Who} needs a way to {do what}
  because {reason/pain point}.

  Today they {current workaround},
  which causes {negative outcome}.

  ➤ Success looks like: {observable outcome}.

─────────────────────────────────────────────────────────────────
```

> **Discovery complete.** Options:
> A) Proceed to Define — structure requirements from this discovery
> B) Refine — adjust the problem statement or explore further
> C) Stop here — save discovery, come back later
>
> If **A**: proceed to Phase 2.
> If **B**: revisit the specific section that needs refinement.
> If **C**: output the discovery brief and stop.

---

## Phase 2 — Define (always runs)

Transform the problem understanding into structured requirements.
Stream all of 2A–2E continuously — no stops between sub-sections.

### 2A. User Stories

For each distinct user need identified in discovery (or provided by the user):

```
AS A {role/persona}
I WANT TO {action/capability}
SO THAT {benefit/outcome}
```

**Rules:**
- Stories describe user goals, not implementation — "manage my notifications" not "click the settings gear icon"
- Each story is independently deliverable
- Stories are prioritized: MUST / SHOULD / COULD (MoSCoW)

| # | Story | Priority | Complexity |
|---|-------|----------|------------|
| 1 | As a {role}, I want to {action} so that {benefit} | MUST | S/M/L |
| 2 | As a {role}, I want to {action} so that {benefit} | MUST | S/M/L |
| 3 | As a {role}, I want to {action} so that {benefit} | SHOULD | S/M/L |

### 2B. Functional Requirements

For each user story, define the specific behaviors:

| # | Story | Requirement | Detail |
|---|-------|-------------|--------|
| FR-1 | S1 | {what the system must do} | {specifics} |
| FR-2 | S1 | {what the system must do} | {specifics} |
| FR-3 | S2 | {what the system must do} | {specifics} |

### 2C. Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | {response time, throughput} | {measurable target} |
| Security | {auth, data protection} | {standard or requirement} |
| Accessibility | {WCAG level, screen reader support} | {specific level} |
| Scalability | {concurrent users, data volume} | {target numbers} |
| Availability | {uptime, degradation tolerance} | {target SLA} |

Only include categories relevant to this spec. Don't pad with boilerplate.

### 2D. Constraints & Assumptions

| Type | Item | Impact |
|------|------|--------|
| Constraint | {hard limit — tech, budget, timeline, regulatory} | {what it rules out} |
| Assumption | {what we're assuming is true} | {what breaks if wrong} |
| Dependency | {what must exist or happen first} | {blocks what} |

### 2E. Out of Scope

Explicitly list what this spec does NOT cover:

| Item | Why out of scope | When to revisit |
|------|-----------------|-----------------|
| {feature/capability} | {reason} | {trigger or timeline} |

---

## Phase 3 — Acceptance Criteria (always runs)

Stream directly after Phase 2 — no gate between them.

For every MUST and SHOULD requirement, define testable acceptance criteria.

### Per-Requirement Criteria

```
FR-{N}: {requirement name}
  GIVEN {precondition}
  WHEN  {action}
  THEN  {expected outcome}

  GIVEN {precondition — edge case}
  WHEN  {action — edge case}
  THEN  {expected outcome — edge case}
```

### Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| {empty input} | {what happens} |
| {invalid input} | {what happens} |
| {concurrent access} | {what happens} |
| {network failure} | {what happens} |
| {permission denied} | {what happens} |
| {data not found} | {what happens} |

Only include scenarios relevant to this spec.

### Verification Matrix

| Requirement | Acceptance Criteria Count | Testable? | Coverage |
|-------------|--------------------------|-----------|----------|
| FR-1 | {N} | Yes/No | Happy + Error + Edge |
| FR-2 | {N} | Yes/No | Happy + Error + Edge |

> Every requirement with **Testable=No** must be reworked until testable.

---

## Phase D — Decompose (conditional)

**Runs if:** `/spec decompose` mode, or Phase 0 determined the goal is too large
for a single `/eng` cycle.

**Skip if:** `/spec discover` or `/spec define` mode.

**This is the ONE /spec mode that writes files** (TODOS.md only), parallel to
how `/ship` writes CHANGELOG.

### Re-entry Detection

Before decomposing, check for existing decomposed tasks:

```bash
ls TODOS.md 2>/dev/null
```

If TODOS.md exists, read it and look for `### Feature:` sections.

**If the requested feature already has a section:**

```
DECOMPOSE RE-ENTRY ──────────────────────────────────────────────

  Feature         {name}
  Tasks           {N total}  ({done} done, {remaining} remaining)

  PROGRESS
  ─────────────────────────────────────────────────
  ✓ #1  {description}
  ✓ #2  {description}
  ▸ #3  {description}  ← NEXT (dependencies met)
  ○ #4  {description}  (depends: #3)

  ➤ NEXT: /eng on task #3, or /spec decompose to revise plan

─────────────────────────────────────────────────────────────────
```

If all tasks are `[x]`: output "Feature complete. Run `/retro`?"

If a suggested next task has unmet dependencies, warn and suggest the
blocking task instead.

### Decomposition

Break the goal into **3-8 sequenced tasks**, each scoped to ≈ one `/eng` → `/tdd` cycle.

**Sizing rules:**
- < 3 tasks → "This is small enough for a single `/eng` — skip decompose"
- 3-8 tasks → proceed normally
- \> 8 tasks → group into 2-3 milestones, each with 3-5 tasks

**Input sources** (in priority order):
1. A `/spec` output from earlier in the conversation — extract stories and decompose
2. A user description of the feature
3. An existing spec document referenced by path

For each task, produce:

| Field | Content |
|-------|---------|
| Priority | P1 (blocking/critical), P2 (important), P3 (nice-to-have) |
| Effort | S (< half day), M (half day–2 days), L (2+ days) |
| ID | Sequential `#N`, continuing from highest existing ID in TODOS.md |
| Description | What to build (1 sentence, imperative) |
| Dependencies | `(depends: #N, #M)` or `(depends: none)` — immediate predecessors only |
| Goal | 1 sentence — what this task achieves |
| AC | 3-5 testable acceptance criteria |

**AC depth rule:** 3-5 acceptance criteria per task at this stage. Decompose
ACs define "what done looks like" — `/eng` expands them into full GIVEN/WHEN/THEN
detail during Phase 3. More than 5 means you're over-specifying for decompose.

### Decompose Output

Present the task manifest for review before writing:

```
DECOMPOSE ───────────────────────────────────────────────────────

  Feature         {name}
  Tasks           {N}
  Milestones      {N — or "single sequence"}

  MILESTONE 1: {name}  ═══════════════════════════════════════
  {one-line goal for this milestone}

  #1  P1 [M] {description} (depends: none)
      Goal: {1 sentence}
      AC:
        - {criterion}
        - {criterion}

  #2  P1 [S] {description} (depends: none)
      Goal: {1 sentence}
      AC:
        - {criterion}
        - {criterion}

  MILESTONE 2: {name}  ═══════════════════════════════════════
  {one-line goal for this milestone}

  #3  P1 [L] {description} (depends: #1, #2)
      Goal: {1 sentence}
      AC:
        - {criterion}
        - {criterion}
        - {criterion}

  SEQUENCE
  ─────────────────────────────────────────────────
  #1 ──→ #3 ──→ #4
  #2 ──┘

  PARALLEL OPPORTUNITIES
  ─────────────────────────────────────────────────
  {List independent tasks and note they can run concurrently.
   If >= 3 independent tasks, recommend /swarm.
   If no tasks are independent: "All tasks are sequential."}

─────────────────────────────────────────────────────────────────
```

**Milestone rules:**
- If total tasks ≤ 5 and form a single sequence, omit milestone headers
- If total tasks > 5 or tasks group into distinct phases, use milestone headers
- Each milestone gets a one-line goal statement

**Parallel rules:**
- Always include the PARALLEL OPPORTUNITIES section
- If ≥ 2 tasks have no mutual dependencies, list them
- If ≥ 3 independent tasks, explicitly recommend `/swarm`
- If no parallelism: "All tasks are sequential — no parallelism available."

> **GATE D.** User approves the task manifest. Approval means: writing tasks
> to TODOS.md. After that, work through tasks with `/eng` per task — each
> task gets its own `/eng` → `/tdd` → `/review` → `/qa` → `/ship` cycle.
> No code until `/eng` plans the task.

### Write to TODOS.md

After user approval, write tasks under a feature heading:

```markdown
### Feature: {name}

- [ ] P1 [M] #1 {description} (depends: none)
  - Goal: {1 sentence}
  - AC:
    - [ ] {criterion}
    - [ ] {criterion}

- [ ] P1 [S] #2 {description} (depends: none)
  - Goal: {1 sentence}
  - AC:
    - [ ] {criterion}
    - [ ] {criterion}
```

**Write rules:**
- If TODOS.md doesn't exist, create it with a standard header
- If TODOS.md exists, append the new feature section below existing content
- Never move or reformat existing items
- Start `#N` numbering after the highest existing ID
- If a task overlaps with an existing TODO, flag: "Task #N overlaps with existing: {description}. Skip? Replace?"

---

## Phase 4 — Summary

### If Decompose Mode

Phase 4 after decompose produces a brief product overview and handoff — not a
full spec document (Phases 2-3 were skipped, so stories/FRs/NFRs don't exist).

```
✓ DECOMPOSE COMPLETE ────────────────────────────────────────────

  Feature         {name}
  Tasks           {N} written to TODOS.md
  Milestones      {N}
  Dependencies    {summary of dependency chain}
  Parallel        {N} tasks can run concurrently

  PRODUCT OVERVIEW
  ─────────────────────────────────────────────────
  {2-3 sentence summary of what the feature/product does,
   the problem it solves, and who it's for. Derived from
   Phase 0 scope card and decompose discussion.}

  TASK SEQUENCE
  ─────────────────────────────────────────────────
  {copy of SEQUENCE diagram from Phase D}

  ➤ NEXT: /eng on task #{first available} (or /swarm
    for independent tasks). Each task follows:
    /eng → /tdd → /review → /qa → /ship

─────────────────────────────────────────────────────────────────
```

> Skip the Spec Registry, Handoff Summary, and Next Step table below —
> they apply to define mode only. Decompose output IS the handoff.
> TODOS.md IS the artifact.

### If Define Mode (default)

> **GATE 2.** Present the full spec for user approval.

### Spec Document

```
✓ SPEC COMPLETE ─────────────────────────────────────────────────

  Problem area       {name}
  Mode               {discover + define | define only}
  Problem statement  {one-liner from Phase 1D}
  Spec type          {new capability | delta (modifying existing)}

  DELIVERABLES
  ─────────────────────────────────────────────────
  User stories       {N}  (MUST: {n}  SHOULD: {n}  COULD: {n})
  Functional reqs    {N}
  Non-functional     {N}
  Acceptance crit    {N}  across {M} requirements
  Out of scope       {N}  items deferred

  OPEN ITEMS
  ─────────────────────────────────────────────────
  Constraints        {N}
  Assumptions        {N} — validate before shipping
  Dependencies       {N}

  ➤ NEXT: /design (if UI) or /eng (see Next Step below)

─────────────────────────────────────────────────────────────────
```

### Spec Registry — Persist to `specs/`

After user approves the spec, write it to the persistent spec registry.
This keeps a living record of what the system does, organized by capability.

**Directory structure:**

```
specs/
  {capability-name}.md     # One file per capability (kebab-case)
```

**Capability detection:** Derive the capability name from the problem area.
Use kebab-case (e.g., `user-notifications`, `auth-flow`, `payment-processing`).

#### New Capability (no existing spec in `specs/`)

Write the full spec as a capability spec:

```markdown
# {Capability Name}

> Last updated by /spec on {YYYY-MM-DD}

## Requirements

- **FR-1:** {requirement} (MUST)
  - GIVEN {precondition} WHEN {action} THEN {outcome}
- **FR-2:** {requirement} (SHOULD)
  - GIVEN {precondition} WHEN {action} THEN {outcome}

## Non-Functional

- {category}: {requirement} — {target}

## Constraints

- {constraint}

## Out of Scope

- {item} — {reason}
```

#### Delta Spec (existing spec found in `specs/`)

When modifying an existing capability, produce a delta spec showing only
what changed. This makes reviews clearer and prevents merge conflicts.

```markdown
# Delta: {Capability Name}

> Change: {one-line summary of what's changing}
> Date: {YYYY-MM-DD}
> Modifies: specs/{capability-name}.md

## ADDED

- **FR-N:** {new requirement} (MUST/SHOULD)
  - GIVEN {precondition} WHEN {action} THEN {outcome}

## MODIFIED

- **FR-M:** {changed requirement} (MUST/SHOULD)
  - Was: {previous behavior}
  - Now: {new behavior}
  - Reason: {why this changed}
  - GIVEN {precondition} WHEN {action} THEN {outcome}

## REMOVED

- **FR-K:** {removed requirement}
  - Reason: {why this is being removed}
```

**Delta rules:**
- Only include sections that have changes (omit empty ADDED/MODIFIED/REMOVED)
- Every MODIFIED entry must include "Was/Now/Reason" for reviewability
- Every REMOVED entry must include a reason
- Use RFC 2119 keywords (MUST, SHOULD, MAY) for requirement strength

**Write rules:**
- New capabilities: write directly to `specs/{capability-name}.md`
- Delta specs: write to `specs/deltas/{capability-name}-{date}.md`
- Delta specs are merged into main specs by `/ship` during archival
- If `specs/` doesn't exist, create it (and `specs/deltas/` if producing a delta)
- Ask user before writing: "Write spec to `specs/{file}`?"

### Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Feature has UI/UX work | `/design` | Translate requirements into wireframes, interactions, accessibility |
| Pure backend / API / infra | `/eng` | Technical planning + TDD |
| Need library/API research | `/research` → `/eng` | Evaluate options before committing to architecture |
| Decompose mode — tasks written | `/eng` on first task | Start working through the task sequence |
| Decompose mode — parallelizable | `/swarm` | Independent tasks benefit from parallel execution |

**Autonomous mode:** If running inside a recipe chain (e.g., Feature recipe), auto-proceed
to `/design` if any user story involves UI, otherwise `/eng`. Pass the Handoff Summary
as input.

### Handoff Summary

Compressed handoff for downstream skills. The full spec is available in conversation
context — downstream skills reference it directly, not this summary.

```
SPEC → {DESIGN | ENG}:
  Problem:      {one-line problem statement}
  Stories:      {N} total ({M} must-have)
  Key FRs:      {top 3-5 requirements by priority}
  Constraints:  {hard limits that affect design/architecture}
  Assumptions:  {unvalidated — flag if building on these}
```

This summary is the **pointer** — `/eng` and `/design` read the full spec phases above
when they need detail.

---

## Rules

| Rule | Why |
|------|-----|
| Read-only (except decompose) | No code changes. Decompose mode writes to TODOS.md only — like how `/ship` writes CHANGELOG |
| Stream by default | Output continuously. Gate only at decision points (scope, final approval) |
| 2 gates default | Gate 1: scope confirmation. Gate 2: final spec approval. Add gates when a user decision is genuinely needed (ambiguous requirement, conflicting constraints). Don't gate for confirmation — gate for decisions |
| Be opinionated | Recommend priorities. "Everything is P1" is not a spec |
| Testable criteria only | Every acceptance criterion must be verifiable. No vague "should be fast" |
| Explicit out-of-scope | Silence on scope = scope creep. Name what's excluded |
| Problem before solution | Discovery defines the problem. Define structures the solution space. Neither prescribes implementation |
| Respect constraints | Don't spec what can't be built given the declared constraints |
| Carry context forward | Phase 2 references Phase 1 findings. Phase 3 references Phase 2 requirements |
| No implementation details | "System sends email" not "Use SendGrid with retry queue." That's `/eng`'s job |
| High-level first | Summary table before expanding into detail |
| Tables over walls of text | Tables for structured data. Prose only for context |
| Complexity routing | Small tasks get single-pass output. Large tasks get full flow. Match depth to scope |

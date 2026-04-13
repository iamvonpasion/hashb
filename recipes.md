# Workflow Recipes

Predefined skill chains for common development workflows. Each recipe lists
the skills to run in order, the gates between them, and what gets handed off.

> These are reference patterns — run each skill manually, or use them as
> a checklist for your own workflow.

---

## Patterns (apply to all recipes)

**Testing pattern:** `/qa` reports bugs. `/fix` addresses them. `/qa` verifies. Repeat until clean — max 3 cycles. If bugs persist after 3 QA/fix cycles, escalate to user with remaining issues.

**Review pattern:** `/review` peer-reviews implementation before `/qa`. APPROVED → proceed. CHANGES REQUESTED → fix and re-review (max 2 cycles). ESCALATE → stop for user.

**TDD pattern:** `/eng` produces a test-first plan. `/tdd` executes it: writes failing tests first (RED), then code to pass them (GREEN), then refactors. Every checkpoint verifies the cycle. `/tdd` requires `/eng` output as input.

**Ship-readiness pattern:** The default chain before shipping is `/review` → `/qa` → `/ship`. After `/review` returns APPROVED or APPROVED WITH NOTES, the next step is `/qa` — not `/ship`. If `/review` findings are fixed, re-run `/review` to confirm, then proceed to `/qa`. The user can override this (e.g., "skip QA, just ship") — respect explicit overrides.

**Simplification pattern:** `/simplify` cleans up code without changing behavior.
It can be invoked standalone (`/simplify src/utils/`) or triggered by `/review`
when quality smells are noted. After `/simplify`, re-run `/review` only if the
simplification was significant (>5 files changed). Otherwise proceed to `/qa`.

**Isolation pattern:** Feature work should use a git worktree or feature branch
to isolate changes from the base branch. For recipes that use `/swarm`, worktrees
are created automatically. For single-stream work, create a worktree before
implementation begins:

```bash
git worktree add -b feature/<name> .worktrees/<name>
cd .worktrees/<name>
# ... implement, test, review ...
# After /ship: worktree is cleaned up
```

> Worktree isolation is recommended, not mandatory. Skip for trivial changes
> (< 3 files, single commit).

---

## Recipe: Feature

**Purpose:** Spec, design (if UI), engineer, build (TDD), review, test, and ship a feature.

```
/spec → /design (if UI) → /eng → /tdd → /review → /simplify (if warranted) → /qa → /fix loop → /ship → /retro
```

> **Unclear requirements?** Start with `/explore` before `/spec`. Explore produces
> no artifacts — it's collaborative thinking. Transition to `/spec` when the problem
> is clear enough to formalize.

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 0 | `/spec` | Discover problem, define requirements, acceptance criteria. Streams continuously, 2 gates (scope, final) | User approves spec |
| 0.5 | `/design` | Wireframes, components, interactions, accessibility (if UI). Streams continuously, 2 gates (wireframes, final) | User approves design |
| 1 | `/eng` | Review architecture (if needed) + single-pass implementation review + TDD plan. 2 gates (scope, TDD plan) | User approves TDD plan |
| 2 | `/tdd` | Execute TDD plan from `/eng`. RED → GREEN → REFACTOR per cycle. Full suite check at end | All tests green → proceed to review |
| 2.5 | `/review` | Agent reviews diff against plan, rules, quality standards | Autonomous: APPROVED → QA, CHANGES REQUESTED → fix + re-review (max 2x), ESCALATE → user |
| 2.6 | `/simplify` | Simplify code flagged by review (if quality smells noted) | Autonomous: complete → QA. Reverts → user |
| 3 | `/qa` | Test: happy path, errors, edge cases. Requirement verification | Clean → ship. Bugs → `/fix` loop |
| 4 | `/ship` | Merge base, run tests, version, changelog, PR | Output: PR URL |
| 5 | `/retro` | What worked, what didn't, action items, persist learnings | Optional |

**Example:**
```
/spec Add user notification preferences.
      Email, push, and in-app channels with per-event opt-in/out.
/eng
```

**Tips:**
- Requirements unclear? Run `/explore` first to think through the problem space.
- Need tech research? Run `/research` before `/eng`.
- Too big for one stream? Run `/swarm plan` to decompose into parallel streams.
- Feature too big for one session? Run `/spec decompose` — see **Decomposed Feature** below.
- Problem space unclear? Start with `/spec discover` for deep discovery.
- No UI work? Skip `/design` and go straight from `/spec` to `/eng`.
- Modifying existing feature? `/spec` produces delta specs (ADDED/MODIFIED/REMOVED) that `/ship` merges into the spec registry.

### Decomposed Feature

When a feature is too large for a single session, decompose it first, then
work through tasks one at a time. TODOS.md is the shared state across sessions.

```
/spec decompose → [per task: /eng → /tdd → /review → /qa → /ship] → /retro
```

**Lifecycle:**

| Phase | What happens | Persistence |
|-------|-------------|-------------|
| 1. Decompose | `/spec decompose` breaks the feature into sequenced tasks | Tasks written to TODOS.md |
| 2. Pick task | `/eng` reads TODOS.md, identifies next task with dependencies met | Task #{N} becomes the session scope |
| 3. Build task | `/eng` → `/tdd` → `/review` → `/qa` for the picked task | Normal feature workflow per task |
| 4. Ship task | `/ship` marks task #{N} complete in TODOS.md, creates PR | `- [x]` with completion date and PR ref |
| 5. Next task | Start a new session. `/eng` auto-detects next available task | TODOS.md tracks progress |
| 6. Repeat | Until all tasks are `[x]` | — |
| 7. Retro | `/retro` after the last task ships | Learnings persisted |

**Rules:**
- One task per `/eng` session. Finish it (eng → tdd → review → qa → ship) before starting the next.
- Respect the dependency graph — don't start a task until its dependencies are complete.
- Independent tasks can run in parallel with `/swarm` (spec decompose identifies these).
- If a task has UI work, include `/design` in that task's cycle (after `/eng`, before `/tdd`).
- If a task turns out to be too large during `/eng`, run `/spec decompose` on that task to break it down further.
- Re-running `/spec decompose` on an existing feature shows progress and suggests the next task.

**Cross-session handoff:** TODOS.md is the only persistence layer. Each `/eng`
session reads it, picks a task, and scopes to that task. Each `/ship` marks the
task complete. No other state is needed.

**Example:**
```
# Session 1: Decompose
/spec decompose Build a notification system with email, push, and in-app channels.
# → writes 5 tasks to TODOS.md

# Session 2: Task #1
/eng
# → picks task #1 (user model + migrations), plans, builds, ships

# Session 3: Task #2
/eng
# → picks task #2 (notification service), plans, builds, ships

# ... continue until all tasks complete

# Final session
/retro
```

---

## Recipe: Bugfix

**Purpose:** Investigate, fix, review, verify, and ship a bug fix.

```
/fix → /review → /qa → /ship → /retro
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/fix` | Gather evidence, identify root cause, write regression test first (TDD), apply fix | User approves root cause before fix |
| 1.5 | `/review` | Verify fix addresses root cause, regression test covers failure mode | Autonomous gate |
| 2 | `/qa` | Verify bug is fixed, check for regressions | Clean → ship. Issues → `/fix` loop |
| 3 | `/ship` | PR with full RCA in commit | Output: PR URL |
| 4 | `/retro` | Was root cause preventable? Should a rule or gate be added? | Optional |

**Example:**
```
/fix Dashboard shows stale data after saving preferences.
     Affects users who changed settings in the last 24 hours.
```

---

## Recipe: Release

**Purpose:** Review, test, and ship the current branch.

```
/review → /simplify (if warranted) → /qa → /fix loop → /ship → /retro
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/review` | Review all code on branch | Autonomous gate |
| 1.5 | `/simplify` | Simplify code flagged by review (if quality smells noted) | Autonomous: complete → QA |
| 2 | `/qa` | Test changes on branch | Clean → ship. Issues → fix |
| 3 | `/fix` | Address QA issues (only if needed) | Re-run `/qa` |
| 4 | `/ship` | PR | Output: PR URL |
| 5 | `/retro` | Release quality assessment | Optional |

**Example:**
```
# Just review, test, and ship what's on the branch
/review
```

---

## Recipe: Architecture

**Purpose:** Research, plan, and document architecture for a new system.

```
/research → /eng arch → iterate decisions → produce ADRs
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/research` | Investigate technology options, patterns, ecosystem | User reviews brief |
| 2 | `/eng arch` | Walk through decisions one at a time, validate against standards. Cross-cut validation streams after last decision | Per-decision confirmation. Final gate: TDD plan |
| 3 | — | Architecture complete. ADRs + validation + TDD plan produced | — |

**Output:** Set of ADRs + architecture validation summary.

**Example:**
```
/research What's the best approach for a TypeScript API with PostgreSQL?
/eng arch Design the backend for a task management SaaS. TypeScript, PostgreSQL, multi-tenant.
```

**Next steps after architecture:**
- Feature recipe to implement each area
- `/swarm plan` if multiple areas can be built in parallel

---

## Recipe: Exploration

**Purpose:** Think through an unclear problem before committing to a plan.

```
/explore → /spec (or /quick, /research, /eng — whatever fits)
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/explore` | Investigate codebase, compare approaches, map problem space, think freely | No gates — user drives direction |
| 2 | Varies | Transition to appropriate skill when clarity is sufficient | User decides when to transition |

**Use when:**
- "I have an idea but I'm not sure what we need"
- "Let me understand how this part works first"
- "What would happen if we changed X?"
- New domain, unfamiliar codebase, or half-formed requirements

**Example:**
```
/explore How does our auth system handle session refresh?
         I'm thinking about adding token rotation but not sure
         what the impact would be.
```

**Transitions:** `/explore` suggests next steps but never forces them.
The user decides when to move from exploration to action.

---

## Recipe: Brownfield Onboarding

**Purpose:** Understand an existing codebase, bootstrap hashb, and verify compliance.

```
/understand → /init → /audit → [feature work]
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/understand` | Map architecture, module boundaries, data flows, conventions. Produces `.understand/architecture-map.md` | User validates architecture map |
| 2 | `/init` | Bootstrap hashb — consumes architecture map to pre-populate Profile fields and domain boundaries | User approves scaffolding |
| 3 | `/audit` | Verify compliance with architecture-aware context | Review findings |
| 4 | Varies | Feature work, bugfixes, or further exploration | — |

**Use when:**
- Adopting hashb on an existing codebase
- Joining a project for the first time
- Onboarding a new team member to an unfamiliar repo
- Before major refactoring of a legacy system

**Example:**
```
/understand
# → produces architecture map, validates with user
# → auto-proceeds to /init (architecture context feeds Profile)
/init
# → scaffolds rules using detected domain boundaries
/audit
# → verifies compliance, flags remaining gaps
```

**Tips:**
- If graphify is installed, run `/graphify . --mcp` before `/understand` for graph-accelerated analysis.
- For very large codebases, scope `/understand` to a subsystem first: `/understand src/billing/`.
- If hashb is already set up, skip `/init` and go straight to `/audit`.
- **Optional follow-up:** after `/audit`, run `/grill` if you want an architecture fitness evaluation (over/under-engineering, refactoring ROI). Not part of the default chain — see the **Architecture Health Check** recipe below.

---

## Recipe: Architecture Health Check (Optional / On-Demand)

**Purpose:** Evaluate the fitness of an existing architecture. Challenge decisions,
detect over/under-engineering, produce an ROI-ranked list of refactoring
opportunities.

> **This is an opt-in recipe.** `/grill` is never auto-invoked. It does not run
> as part of Feature, Bugfix, Release, or Brownfield Onboarding. Users initiate
> it deliberately when they want an architecture fitness evaluation.

```
/understand → /grill → [/eng arch per revisited decision] → [/simplify per flagged module]
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/understand` | Map architecture (if map is missing or > 30 days old) | User validates architecture map |
| 2 | `/grill` | Five evaluation passes: Simplicity, Structural Fitness, Coupling, Technology Fit, Assumption Freshness. Produces scored report with ROI matrix | User reviews raw findings (gate 1) and final report (gate 2) |
| 3 | `/eng arch` | Redesign each flagged decision (one per session) | Per-decision confirmation |
| 4 | `/simplify` | Reduce complexity in modules flagged by grill | Standard simplify flow |

**Use when:**
- Onboarding to a brownfield codebase and wanting more than just a map
- Before a major refactoring initiative
- Quarterly or annual architecture health review
- When the team suspects architectural debt is slowing delivery
- After significant growth (team size, traffic, feature scope) that may have outgrown original decisions

**Do NOT use as:**
- A per-feature step (that's overhead — grill is for infrequent, deliberate evaluation)
- A replacement for `/audit` (different question: fitness, not hashb compliance)
- A replacement for `/understand` (grill requires `/understand` output as input)

**Example:**
```
/understand
# → maps architecture, writes .understand/architecture-map.md

/grill
# → challenges decisions, produces .grill/grill-{date}.md with:
#   - fitness scores per dimension
#   - ROI-ranked refactoring matrix
#   - explicit handoffs to /eng arch and /simplify

/eng arch  # run per decision worth revisiting, one at a time
/simplify  # run per module flagged for complexity reduction
```

**Tips:**
- `/grill` is idempotent — re-run to track fitness trend over time. Prior reports
  are preserved under `.grill/`.
- Score trends ARE the fitness function: dropping scores indicate drift; rising
  scores indicate successful remediation.
- If the repo has no ADRs, the Assumption Freshness pass runs in degraded mode
  (inferred from code). Still useful, but labeled as speculative.
- Effort estimates in the ROI matrix are AI-guessed and labeled as rough —
  validate with the team before prioritizing.


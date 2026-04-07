# Workflow Recipes

Predefined skill chains for common development workflows. Each recipe lists
the skills to run in order, the gates between them, and what gets handed off.

> These are reference patterns — run each skill manually, or use them as
> a checklist for your own workflow.

---

## Patterns (apply to all recipes)

**Testing pattern:** `/qa` reports bugs. `/fix` addresses them. `/qa` verifies. Repeat until clean.

**Review pattern:** `/review` peer-reviews implementation before `/qa`. APPROVED → proceed. CHANGES REQUESTED → fix and re-review (max 2 cycles). ESCALATE → stop for user.

**TDD pattern:** `/eng` produces a test-first plan. `/tdd` executes it: writes failing tests first (RED), then code to pass them (GREEN), then refactors. Every checkpoint verifies the cycle. `/tdd` requires `/eng` output as input.

**Ship-readiness pattern:** The default chain before shipping is `/review` → `/qa` → `/ship`. After `/review` returns APPROVED or APPROVED WITH NOTES, the next step is `/qa` — not `/ship`. If `/review` findings are fixed, re-run `/review` to confirm, then proceed to `/qa`. The user can override this (e.g., "skip QA, just ship") — respect explicit overrides.

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
/spec → /design (if UI) → /eng → /tdd → /review → /qa → /fix loop → /ship → /retro
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 0 | `/spec` | Discover problem, define requirements, acceptance criteria. Streams continuously, 2 gates (scope, final) | User approves spec |
| 0.5 | `/design` | Wireframes, components, interactions, accessibility (if UI). Streams continuously, 2 gates (wireframes, final) | User approves design |
| 1 | `/eng` | Review architecture (if needed) + single-pass implementation review + TDD plan. 2 gates (scope, TDD plan) | User approves TDD plan |
| 2 | `/tdd` | Execute TDD plan from `/eng`. RED → GREEN → REFACTOR per cycle. Full suite check at end | All tests green → proceed to review |
| 2.5 | `/review` | Agent reviews diff against plan, rules, quality standards | Autonomous: APPROVED → QA, CHANGES REQUESTED → fix + re-review (max 2x), ESCALATE → user |
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
- Need tech research? Run `/research` before `/eng`.
- Too big for one stream? Run `/swarm plan` to decompose into parallel streams.
- Feature too big for one session? Run `/spec decompose` to break it into sequenced tasks in TODOS.md, then work through them with `/eng` → `/tdd` per task.
- Problem space unclear? Start with `/spec discover` for deep discovery.
- No UI work? Skip `/design` and go straight from `/spec` to `/eng`.

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
/review → /qa → /fix loop → /ship → /retro
```

| Phase | Skill | What happens | Gate |
|-------|-------|-------------|------|
| 1 | `/review` | Review all code on branch | Autonomous gate |
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


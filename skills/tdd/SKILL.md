---
name: tdd
description: >
  Test-driven implementation — execute the TDD plan from /eng.
  Use after /eng produces a TDD execution plan. Writes failing tests first (RED), implements code to pass (GREEN), then refactors. Requires /eng output as input.
  Modes: /tdd (start), /tdd continue (resume from last checkpoint).
argument-hint: "[\"continue\" to resume]"
---

# TDD

Execute the implementation plan from `/eng` using strict test-driven development.
**No /eng output = no /tdd.** This skill requires a TDD Execution Order as input.

**This skill modifies code.**

---

## When to Use

| Signal | Action |
|--------|--------|
| `/eng` produced a TDD plan | `/tdd` — execute it |
| Implementation interrupted mid-cycle | `/tdd continue` — resume from last checkpoint |
| No `/eng` output exists | Stop. Run `/eng` first. |

---

## Presentation Rules

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Phase 0: Validate Input

Read the `/eng` output. Extract the TDD Execution Order.

**Required fields from /eng:**

| Field | What it contains |
|-------|-----------------|
| Test order | Numbered list of test files to create |
| Per test | What to verify (mapped to acceptance criteria) |
| Implementation order | What code to write after each test |

**If missing or incomplete:**

> **STOP.** The /eng output doesn't include a TDD plan.
> Run `/eng impl` first to generate the TDD Execution Order.

**Output:** Confirm the plan:

```
TDD PLAN — {N} test cycles
──────────────────────────────────────────────
  1. {test file} → {what it verifies}
  2. {test file} → {what it verifies}
  ...

Proceed with cycle 1?
```

> **STOP.** Confirm plan with user before writing any code.

---

## Phase 1: Branch Setup

Before writing any code, ensure you're on a feature branch:

```bash
# Branch/base detection — see skills/shared/preflight.md
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "master" ]; then
  echo "⚠ ON PROTECTED BRANCH — creating feature branch"
  SLUG=$(echo "{feature-description}" | sed 's/[^a-zA-Z0-9]/-/g' | tr '[:upper:]' '[:lower:]')
  SLUG="${SLUG:0:30}"
  git checkout -b "feat/$SLUG"
fi
```

> **Never write code directly on main/master.**

---

## Phase 2: RED → GREEN → REFACTOR

Execute each test cycle from the TDD plan **one at a time, in order.**

### For each cycle:

#### Step 1: RED — Write the test

Write the test file as specified in the TDD plan.

**Rules:**
- Test must describe expected behavior, not implementation details
- Test must be runnable — no placeholders, no `skip`, no `todo`
- Test must target the acceptance criteria from /spec

**Run the test. It MUST fail.**

```
✕ {test name} — FAILED (expected)
```

If the test passes before any implementation code:
- The test is wrong (testing existing behavior, not new)
- Or the feature already exists (check with user)

> **RED GATE:** Test must fail. If it passes, stop and investigate.

#### Step 2: GREEN — Write the implementation

Write the **minimum code** to make the test pass.

**Rules:**
- Smallest change that makes the test green
- No extra features, no "while I'm here" additions
- No premature abstractions — three similar lines > one premature helper
- Follow patterns from the codebase — read existing code first
- **Scope discipline:** Touch only what the task requires. Do not clean up
  adjacent code, refactor unrelated imports, remove comments you disagree with,
  add features not in the TDD plan, or modernize syntax in files you're only
  reading. Note issues for later — don't fix them now.

**Run the test. It MUST pass.**

```
✓ {test name} — PASSED
```

> **GREEN GATE:** Test must pass. If it fails, fix the implementation, not the test.

#### Step 3: REFACTOR — Clean up

Tests are green. Now improve the code without changing behavior.

**Rules:**
- Run tests after every refactor step — they must stay green
- Improve naming, extract methods only if genuinely needed
- Apply patterns from the existing codebase
- Do not add features during refactor

**Run tests again. Still green?**

```
✓ All tests passing after refactor
```

> **REFACTOR GATE:** Tests must stay green. If any fail, undo and try again.

### Cycle checkpoint

After each RED → GREEN → REFACTOR cycle, report:

```
CYCLE {N}/{total} COMPLETE
──────────────────────────────────────────────
  Test:    {test file}
  Verifies: {acceptance criteria}
  Status:  GREEN ✓
  Files:   {files created/modified}
──────────────────────────────────────────────
```

**If more cycles remain:** Proceed to next cycle.

**If user asked to pause:** Save state and report where to resume with `/tdd continue`.

### Checkpoint Persistence

After each cycle checkpoint, persist state to disk so `/tdd continue` can resume
after context compaction:

```bash
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
cat > ".tdd-checkpoint-${BRANCH}.json" << CKPT
{
  "branch": "$BRANCH",
  "total_cycles": {total},
  "completed": {N},
  "last_test": "{test file}",
  "last_status": "GREEN",
  "eng_plan_summary": "{one-line summary of remaining cycles}",
  "updated": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
CKPT
```

**On `/tdd continue`:** Read `.tdd-checkpoint-{branch}.json` first. If it exists,
use it to determine which cycle to resume from instead of re-parsing conversation context.

**On completion (Phase 3 pass):** Clean up: `rm -f ".tdd-checkpoint-${BRANCH}.json"`

---

## Phase 3: Suite Check

After all cycles are complete, run the **full test suite** — not just the new tests.

```bash
# Run whatever test command the project uses
npm test    # or pytest, dotnet test, etc.
```

**All tests must pass.** If existing tests break:
1. Read the failing test to understand what changed
2. Determine if the failure is expected (intentional behavior change) or a regression
3. If regression: fix the implementation, not the existing test
4. If intentional: update the existing test with user approval

**Output:**

```
SUITE CHECK
──────────────────────────────────────────────
  New tests:      {N} passing ✓
  Existing tests: {N} passing ✓
  Regressions:    {N} (0 = good)
──────────────────────────────────────────────
```

> **STOP if regressions > 0.** Do not proceed to handoff with broken tests.

---

## Phase 4: Handoff

Summarize what was built and hand off to `/review`.

```
✓ TDD COMPLETE ─────────────────────────────────────────────────

  Feature       {feature name from /eng}
  Cycles        {N} RED → GREEN → REFACTOR
  Tests added   {N} — all passing
  Files created {list}
  Files modified {list}
  Suite status  ALL GREEN ✓

  Acceptance criteria coverage:
    AC-1: {description} → {test file} ✓
    AC-2: {description} → {test file} ✓
    ...

  ➤ NEXT: /review → /qa → /ship

─────────────────────────────────────────────────────────────────
```

### Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| All cycles complete, suite green | `/review` | Peer review before QA |
| Regressions found, can't resolve | Escalate to user | Need human judgment |
| Scope grew during implementation | `/eng hold` | Re-scope before continuing |

**Autonomous mode:** If running inside a recipe chain:
- Suite green → auto-invoke `/review`
- If `/review` returns CHANGES REQUESTED → fix and re-run affected cycles
- Max 2 review rounds, then escalate to user

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "I'll write tests after the code works" | Tests written post-facto verify implementation, not behavior. You end up testing what you built instead of what you should have built. |
| "This is too simple to need a test" | Simple code evolves into complex code. The test documents intent and catches regressions when someone changes it later. |
| "The RED step is slowing me down" | Skipping RED means you don't know if your test actually verifies anything. A test that never failed might never catch a bug. |
| "I need to refactor first before writing tests" | Refactor happens in the REFACTOR phase, after GREEN. Refactoring without test coverage is guessing. |
| "I'll just write all the tests first, then implement" | Batching tests breaks the feedback loop. You lose the signal of which implementation change satisfies which test. |
| "The existing tests cover this already" | If existing tests covered the new behavior, they would have caught the need for it. New behavior needs new tests. |
| "I know this implementation is correct" | Confidence is not evidence. The test suite is your proof — without it, the next developer has only your word. |

---

## Rules

1. **Requires /eng input.** No TDD plan = no /tdd. Period.
2. **One cycle at a time.** Never write multiple tests before implementing.
3. **RED must fail.** A test that passes on first run is suspicious.
4. **GREEN means minimum code.** Just enough to pass. Nothing extra.
5. **Refactor doesn't add features.** Only clean up what's there.
6. **Tests must stay green.** After every refactor, after every cycle.
7. **No placeholders.** Every test has real assertions. Every implementation is real code.
8. **Follow existing patterns.** Read the codebase before writing. Match what's there.
9. **Minimal diff per cycle.** Small commits, clear purpose.
10. **Full suite before handoff.** New tests passing isn't enough — everything must pass.

---
name: fix
description: >
  Systematic debugging with root cause analysis.
  Use when something is broken, errors appear, or behavior is unexpected. Gathers evidence, tests hypotheses, writes regression tests, fixes the root cause. No band-aid fixes.
argument-hint: "[bug description]"
---

# Investigate

**Iron Law: NO FIXES WITHOUT ROOT CAUSE.**

Fixing symptoms creates whack-a-mole debugging. Every fix that doesn't
address root cause makes the next bug harder to find.

---

## Presentation Rules

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Phase 1: Gather Evidence

Collect before hypothesizing. Do not guess.

**Read the symptoms:** Error messages, stack traces, reproduction steps.
If the user hasn't provided enough context, ask ONE focused question.

**Trace the code path:** Follow execution from symptom back to potential
causes. Grep all references, read the logic.

**Check recent changes:**

```bash
git log --oneline -20 -- <affected-files>
```

Was this working before? If so, the root cause is in the diff.

**Check known issues:** Scan `TODOS.md` and git log for prior fixes in
the same area. Recurring bugs in the same files = architectural smell.

**Graph-Aware Tracing (if code-review-graph MCP is available):**
If the consumer's Project Profile lists `code-review-graph`, query it to
accelerate evidence gathering:
- Trace callers and dependents of the symptomatic function to find the
  full call chain from entry point to failure site
- Identify which tests cover the affected code paths — if they're passing,
  the bug may be in an untested path
- Map the blast radius of the suspected root cause before fixing

If code-review-graph is not available, trace manually via grep and file reads.

**Reproduce:** Can you trigger it deterministically? If not, gather more
evidence before proceeding.

**Output:** "Root cause hypothesis: ..." — a specific, testable claim
about what is wrong and why.

---

## Phase 2: Test the Hypothesis

Before writing ANY fix, verify your hypothesis with instrumentation.

### Instrumentation Gate

Determine whether instrumentation is needed based on the bug type:

| Bug type | Instrument first? | Example |
|----------|-------------------|---------|
| **Obvious** — root cause visible in code | No — fix directly | Typo, missing import, wrong variable, config error |
| **Non-obvious** — hypothesis needs verification | **Yes — mandatory** | State bugs, race conditions, hydration, lifecycle, async timing, silent failures |

**If instrumentation is needed:**

1. Add `console.log` / `logger.debug` / breakpoint at the suspected root cause
2. Add logging at the entry and exit of the affected code path
3. Run the reproduction and **paste the diagnostic output**
4. Only proceed to a fix if the output confirms your hypothesis

If you cannot reproduce with instrumentation, say so and present options
to the user. Do not guess.

> **Why this matters:** Retro evidence shows that skipping instrumentation
> on non-obvious bugs leads to 3-6x fix attempts. Reading logs is faster
> than reading diffs.

**Confirm it:** Does the evidence (code reading or diagnostic output) match your hypothesis?

### 5 Whys Analysis

Once the immediate cause is identified, apply 5 Whys to reach the systemic
root cause. Shallow fixes ("the variable was null") create recurring bugs.
Deep fixes ("the schema allows null where it shouldn't") prevent entire classes
of bugs.

```
WHY #1:  Why did {symptom} happen?
         → {immediate cause}

WHY #2:  Why did {immediate cause} happen?
         → {deeper cause}

WHY #3:  Why did {deeper cause} happen?
         → {process/design cause}

WHY #4:  Why did {process/design cause} happen?
         → {systemic cause}

WHY #5:  Why did {systemic cause} happen?
         → {root cause — fix HERE}
```

**Rules:**
- Stop early if you reach a root cause before 5 — don't force it
- Each "why" must be supported by evidence, not speculation
- If a "why" has multiple answers, branch and investigate each
- The final "why" should point to something **preventable** — a missing
  validation, a design gap, a process failure. If it points to "human error,"
  go one level deeper: why did the system allow the error?

**Output:** State the root cause chain:
```
ROOT CAUSE CHAIN ────────────────────────────────────────────────

  Symptom      {what the user saw}
       │
  Why #1       {immediate cause}
       │
  Why #2       {deeper cause}
       │
  Root cause   {the thing to fix}
       │
  Fix target   {code/process/rule change that prevents recurrence}

─────────────────────────────────────────────────────────────────
```

**If wrong:** Return to Phase 1. Gather more evidence. Do not guess.

**Check if it matches a known pattern:**

| Pattern | Signature | Where to look |
|---------|-----------|---------------|
| Race condition | Intermittent, timing-dependent | Concurrent access to shared state |
| Nil propagation | NoMethodError, TypeError | Missing guards on optional values |
| State corruption | Inconsistent data, partial updates | Transactions, callbacks, hooks |
| Integration failure | Timeout, unexpected response | External API calls, service boundaries |
| Config drift | Works locally, fails elsewhere | Env vars, feature flags, DB state |
| Stale cache | Shows old data, fixes on clear | Redis, CDN, browser cache |

**3-strike rule (HARD STOP — not a suggestion):**

If 3 hypotheses fail, you **MUST** stop. Do not attempt a 4th fix.
Do not continue investigating silently. Present this to the user and WAIT:

```
■ 3-STRIKE STOP ─────────────────────────────────────────────────

  Hypotheses tested:
  1. {hypothesis} — {why it failed}
  2. {hypothesis} — {why it failed}
  3. {hypothesis} — {why it failed}

  Options:
  A) New lead — {describe what's different this time}
  B) Escalate — this needs domain knowledge I don't have
  C) Instrument deeper — add more logging, reproduce again

─────────────────────────────────────────────────────────────────
```

> **Why this is a hard stop:** Retro evidence shows that blowing past
> 3 strikes leads to 6+ attempts, wasted commits, and trust erosion.
> Stopping at 3 and asking is always cheaper than guessing at 6.

### Strike Persistence

Persist strike state to disk so the 3-strike rule survives context compaction:

**On each failed hypothesis**, write `.fix-state-{branch}.json`:

```json
{
  "branch": "{branch}",
  "bug": "{short description}",
  "strikes": [
    {"hypothesis": "...", "result": "disproved — ...", "timestamp": "..."}
  ],
  "status": "INVESTIGATING"
}
```

On 3rd strike, set `"status": "HARD_STOP"`.

**At Phase 2 start**, read `.fix-state-{branch}.json` if it exists:
- If `status` is `HARD_STOP` — present the 3-strike stop immediately.
- If strikes exist but status is `INVESTIGATING` — resume from recorded position.

**On resolution:** `rm -f ".fix-state-${BRANCH}.json"`

---

## Phase 3: Fix

Once root cause is **confirmed** (not suspected — confirmed):

### Branch Guard

Before writing any code, ensure you're on a feature branch:

```bash
# Branch/base detection — see skills/shared/preflight.md
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "master" ]; then
  echo "⚠ ON PROTECTED BRANCH — creating fix branch"
  SLUG=$(echo "{short-bug-description}" | sed 's/[^a-zA-Z0-9]/-/g' | tr '[:upper:]' '[:lower:]')
  SLUG="${SLUG:0:30}"
  git checkout -b "fix/$SLUG"
fi
```

> **Never write fixes directly on main/master.** If the user explicitly
> says to commit to main, warn them and proceed only with confirmation.

1. **Write a regression test FIRST — before writing the fix.**
   The test must target the exact bug scenario from the root cause chain.

2. **RED: Run the test. It MUST fail.** Paste the failure output.
   If it passes without a fix, the test is wrong — it's not testing the bug.
   Rewrite it until it fails for the right reason.

3. **Fix the root cause from the 5 Whys chain, not the symptom.** Smallest
   change that eliminates the actual problem at the deepest actionable level.
   Minimal diff — fewest files, fewest lines. Resist refactoring adjacent
   code during a bug fix.

4. **GREEN: Run the test again. It MUST pass.** Paste the passing output.
   This is the proof the fix works. Without RED then GREEN on the same test,
   you have no proof — only a claim.

5. **Run the full test suite.** Paste output. No regressions.

**Blast radius check:** If fix touches >5 files, stop and ask:

```
This fix touches N files — large for a bug fix.
A) Proceed — root cause genuinely spans these files
B) Split — fix critical path now, defer the rest
C) Rethink — maybe a more targeted approach exists
```

---

## Phase 4: Verify & Report

**Reproduce the original bug and confirm it's fixed.** Not optional.

Show THREE pieces of evidence — not just "tests pass":
1. The regression test failing BEFORE the fix (from Phase 3 step 2)
2. The regression test passing AFTER the fix (from Phase 3 step 4)
3. Full test suite green (from Phase 3 step 5)

If you cannot show all three, you cannot claim the fix works. "I'm confident
this works" without evidence is not acceptable — confidence is not proof.

```
✓ DEBUG REPORT ──────────────────────────────────────────────────

  Symptom          {what the user observed}
  Root cause       {what was actually wrong}
  Fix              {what changed — file:line refs}
  Evidence         {test output or repro showing fix works}
  Regression test  {file:line of new test}
  Related          {TODOS, prior bugs, architectural notes}
  Status           DONE | DONE_WITH_CONCERNS | BLOCKED

─────────────────────────────────────────────────────────────────
```

### Next Step

| Status | Next Skill | Why |
|--------|-----------|-----|
| DONE | `/review` (if not yet reviewed) or `/qa` | Verify fix in context |
| DONE_WITH_CONCERNS | `/qa` with extra attention | Verify, but watch for edge cases |
| BLOCKED | Escalate to user | Root cause unclear after investigation |

**Autonomous mode:** If running inside a recipe chain:
- DONE → auto-proceed to `/review` if this is the first fix, or `/qa` if already reviewed
- DONE_WITH_CONCERNS → auto-proceed to `/qa` with concerns flagged
- BLOCKED → stop the chain, present investigation to user

---

## Red Flags — Slow Down If You See These

- **"This should fix it" / "I'm confident this works"** — confidence without
  evidence is a guess. Show the RED → GREEN test output or don't claim it's fixed.
- **"Quick fix for now"** — there is no "for now." Fix it right or escalate.
- **Proposing a fix before tracing data flow** — you're guessing.
- **Each fix reveals a new problem elsewhere** — wrong layer, not wrong code.
- **Can't reproduce** — don't ship a fix you can't verify.
- **3+ failed attempts** — question the architecture, not your luck.
- **Touching library persistence/lifecycle internals** — if the fix involves
  changing how a library manages state, storage, hydration, or lifecycle
  (Zustand persist, React Query cache, auth token storage, ORM hooks, etc.),
  **STOP and run `/research` first.** Read the library source code or query
  Context7 for the internal behavior before changing it. "Simple" storage
  swaps can cascade into complex bugs when the library's internal assumptions
  are violated.

---

## Completion Status

- **DONE** — root cause found, fix applied, regression test passes, suite green
- **DONE_WITH_CONCERNS** — fixed but can't fully verify (intermittent, needs staging)
- **BLOCKED** — root cause unclear after investigation, escalated


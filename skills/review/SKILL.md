---
name: review
description: >
  Peer code review — agent-to-agent or agent-to-human. Confidence-scored, blocks ship on low confidence.
  Use when code is written and ready for review before shipping. Reviews diff against plan, rules, and quality standards.
---

# Peer Code Review

Review **implemented code** against requirements, architecture, and quality standards.
This is NOT engineering review (`/eng`) — this reviews actual code that has been written.

**Input:** Branch with commits to review (auto-detected or specified).
**Output:** Review verdict with confidence score. Blocking issues must be resolved before `/ship`.

---

## When to Use

| Situation | Skill |
|-----------|-------|
| Reviewing a plan before coding | `/eng` |
| Reviewing code after implementation | **`/review`** |
| Testing the running app | `/qa` |
| Pre-landing security/data check | `/ship` (built-in) |

`/review` fills the gap between `/eng` (engineering plan) and `/qa` (behavior).
It catches design drift, quality issues, and architectural violations
that tests won't find and QA can't see.

---

## Presentation Rules

See `skills/shared/formatting.md` for presentation rules (progress indicators, discussion chunking, table formatting).

---

## Phase 1: Scope the Review

```bash
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null \
  || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
  || echo "main")

echo "REVIEWING: $BRANCH → $BASE"
git diff $BASE...HEAD --stat
git log $BASE..HEAD --oneline
```

**Gather context:**
- Read the original requirement / user request
- Read `/eng` review summary if it exists (check conversation or commit messages)
- Read any ADRs or architecture docs referenced
- Check active rules that apply to changed files
- Read Project Profile from consumer's `CLAUDE.md` (if present) — adapt security
  and performance checks to the declared stack and architecture
- Check for rule overrides — if a consumer rule has `overrides:` targeting a
  generic rule, the override's guidance takes precedence for its matched paths

```bash
# Files changed — these are the review scope
git diff $BASE...HEAD --name-only
```

**Scope rule:** Review ONLY the diff. Don't review pre-existing code unless
the diff makes it worse or introduces a dependency on it.

---

## Phase 2: Review Checklist

Work through each category **in order**. The sequence is intentional:
spec/requirement compliance first (2A), then code quality (2B-2F). If the
implementation doesn't match the spec, code quality feedback is wasted effort —
the code may need to be rewritten. Catch design drift before polishing details.

For each finding, classify severity and confidence.

### 2A. Design Fidelity

Does the code match the plan?

- **Architecture compliance** — Does the implementation follow the approved approach
  from `/eng`? Flag deviations.
- **Module boundaries** — Are bounded contexts preserved? No direct cross-module
  DB access, no bypassing public APIs.
- **Dependency direction** — Dependencies flow the right way? No circular imports,
  no upstream depending on downstream internals.
- **Contract adherence** — Do new APIs match their planned signatures? Are events
  published as designed?
- **Design principles** — Check against: Single Responsibility, Explicit
  Dependencies, Composition over Inheritance, Separation of Concerns,
  Immutability by Default, Small Interfaces / Narrow Contracts.
  Adapt checks to the consumer's declared Stack:

  | Stack | What to look for |
  |---|---|
  | C# / .NET | Constructor DI via `IServiceCollection`, interface segregation, records/readonly for immutability, service layer separation |
  | React / Next.js | Component composition + custom hooks (not inheritance), props/context for dependencies, `useState`/`useReducer` (never mutate), small prop surfaces, server/client component separation |
  | TypeScript / Node | Module-level composition, dependency injection via constructors or factory functions, narrow exported interfaces |
  | Python | Parameter injection, protocols/ABCs for contracts, frozen dataclasses/tuples for immutability, module-level separation |

  Flag violations with specific principle and file:line.
- **Testability** — Can each new unit be tested in isolation? Are dependencies
  injectable? Are contracts narrow? Flag tightly-coupled code that requires
  integration tests where unit tests should suffice.

### 2B. Code Quality

- **Naming** — Types, functions, variables named for what they represent,
  not how they work.
- **Complexity** — Functions >30 lines, nesting >3 levels, cyclomatic
  complexity smells. Can it be simplified?
- **Duplication** — Identical or near-identical logic in multiple places.
  Flag with file:line references.
- **Dead code** — Unreachable branches, unused imports, commented-out code.
- **Error handling** — Errors caught and handled meaningfully? No swallowed
  exceptions, no generic catch-all without re-throw.

### 2C. Test Coverage

- **New codepaths tested?** — Every new function/method has at least one test.
- **Edge cases covered?** — Nil, empty, boundary, error paths.
- **Regression test for bugs?** — If this is a bugfix, is there a test that
  would have caught it?
- **Test quality** — Tests assert behavior, not implementation. No brittle
  mocks of internals.
- **Test coverage proportional to risk** — Critical business logic, data
  mutations, auth flows, and payment paths require thorough unit + integration
  tests. Utility functions and simple getters don't need dedicated tests
  unless they have edge cases. Apply industry-standard judgment — the goal
  is trust and confidence, not 100% coverage for its own sake.

### 2D. Security & Data Safety

- **Input validation** — User inputs validated at the boundary?
- **Auth checks** — New endpoints/mutations protected?
- **Secrets** — No hardcoded credentials, tokens, or keys?
- **SQL/XSS/injection** — Parameterized queries, escaped output?
- **Data migrations** — Reversible? Backfill plan for existing data?
- **Dependency versions** — Any new or changed dependencies must target the
  latest stable release. Flag outdated, pre-release (RC, canary, next), or
  abandoned (no release in >12 months) packages. Verify via Context7
  `resolve-library-id` when uncertain.

### 2E. Performance

- **N+1 queries** — Loops that hit the database?
- **Unbounded results** — Missing pagination or limits?
- **Missing indexes** — New queries without supporting indexes?
- **Resource cleanup** — Connections, file handles, subscriptions closed?

### 2F. Rule Compliance

Check the diff against every active rule that matches the changed files:

```bash
# Which rules activate for these files?
# Cross-reference changed files against rules/ paths: frontmatter
```

**Rule override handling:** If a consumer rule has `overrides: <rule-id>` in its
frontmatter, the override rule's guidance takes precedence over the generic rule
for the paths the override matches. Rules without overrides still apply normally.

Flag any rule violations with rule name and specific line.

---

## Phase 3: Findings

For each issue found:

```
⚠ FINDING #{N}
  ─────────────────────────────────────────────────
  Severity      BLOCKING | WARNING | SUGGESTION
  Confidence    HIGH | MEDIUM | LOW
  Category      design | quality | test | security | performance | rule
  File          {path}:{line}
  Description   {what's wrong}
  Suggestion    {how to fix — be specific}
```

**Severity definitions:**
- **BLOCKING** — Must fix before ship. Security holes, data loss risk,
  architectural violations, missing tests for critical paths.
- **WARNING** — Should fix. Quality issues, missing edge-case tests,
  performance concerns. Ship at user's discretion.
- **SUGGESTION** — Could improve. Naming, style, minor simplifications.
  Don't block on these.

**Confidence definitions:**
- **HIGH** — Certain this is an issue. Clear rule violation, obvious bug,
  proven pattern.
- **MEDIUM** — Likely an issue but context might justify it. Flag for
  discussion.
- **LOW** — Possible concern, might be wrong. Needs domain knowledge
  the reviewer doesn't have.

---

## Phase 4: Verdict

### Confidence Score

Calculate overall review confidence:

```
REVIEW SCORE
─────────────────────────────────────────────────
  Findings       {N} blocking, {N} warning, {N} suggestion
  Confidence     {weighted average of finding confidence}
  Test coverage  {assessed qualitatively: strong | adequate | weak | missing}
```

### Verdict

| Condition | Verdict | Action |
|-----------|---------|--------|
| 0 blocking, confidence HIGH | **APPROVED** | Proceed to `/qa` |
| 0 blocking, confidence MEDIUM | **APPROVED WITH NOTES** | Proceed, address warnings |
| Any blocking, confidence HIGH | **CHANGES REQUESTED** | Fix blockers, re-review |
| Any blocking, confidence LOW | **ESCALATE** | Need human reviewer — agent unsure |
| Confidence LOW overall | **ESCALATE** | Domain knowledge gap — flag for human |

### Output

```
✓ PEER REVIEW ───────────────────────────────────────────────────

  Branch           {branch} → {base}
  Reviewer         {agent | human}
  Date             {date}
  Files reviewed   {N}

  VERDICT: {APPROVED | APPROVED WITH NOTES | CHANGES REQUESTED | ESCALATE}

  BLOCKING ({N})
  ─────────────────────────────────────────────────
  1.  #{finding} — {one-line summary}

  WARNINGS ({N})
  ─────────────────────────────────────────────────
  1.  #{finding} — {one-line summary}

  SUGGESTIONS ({N})
  ─────────────────────────────────────────────────
  1.  #{finding} — {one-line summary}

  Confidence       {HIGH | MEDIUM | LOW}
  Reason           {why this confidence level}
  Rule compliance  {all rules checked | violations listed}

─────────────────────────────────────────────────────────────────
```

### Next Step

| Verdict | Next Skill | Why |
|---------|-----------|-----|
| APPROVED | `/qa` | Behavior testing — review caught design issues, QA catches runtime issues |
| APPROVED WITH NOTES | `/qa` | Proceed with warnings noted |
| CHANGES REQUESTED | Fix blockers → re-invoke `/review` | Max 2 cycles before escalating |
| ESCALATE | Stop — present to user | Agent unsure, needs human judgment |

**After fixing findings:** If WARNING or BLOCKING findings are fixed inline
(same session), re-run a scoped review of just the fixes before proceeding —
verify the fixes are correct and didn't introduce new issues. Output an updated
verdict, then hand off to `/qa`. SUGGESTION fixes don't require re-review.

**Autonomous mode:** If running inside a recipe chain:
- APPROVED / APPROVED WITH NOTES → auto-proceed to `/qa`
- CHANGES REQUESTED → return blockers to implementing agent, re-review after fixes (max 2 cycles)
- ESCALATE → stop the chain, present findings to user

---

## Agent-to-Agent Mode

When invoked by `/workflow` or `/swarm` (not directly by user):

**Autonomous rules:**
- **APPROVED** → pass verdict to orchestrator, proceed to next phase
- **APPROVED WITH NOTES** → pass verdict + warnings, proceed
- **CHANGES REQUESTED** → return blockers to implementing agent, request fixes,
  then re-review (max 2 cycles before escalating to user)
- **ESCALATE** → stop workflow, present findings to user for decision

**Review cycle limit:** Max 2 review rounds per implementation phase.
If blockers remain after 2 rounds, escalate to user with full context.

**Fresh context:** When invoked as a subagent, the reviewer gets ONLY:
- The diff
- The requirement / eng summary
- Active rules
- No conversation history from the implementing agent

This prevents confirmation bias — the reviewer forms independent conclusions.

---

## Rules

- **Review the diff, not the codebase.** Pre-existing issues are out of scope
  unless the diff makes them worse.
- **Be specific.** File:line references for every finding. No vague "consider
  improving" without pointing at code.
- **Explain why.** Every finding needs a reason — not just "this is wrong"
  but "this is wrong because X could happen."
- **Don't nitpick in blocking.** Style preferences are SUGGESTION, not BLOCKING.
  Only block on real risks.
- **Confidence is honest.** LOW confidence is valuable — it means "I'm not sure,
  get a human to check." That's better than a false HIGH.
- **No rubber stamps.** "LGTM" without checking every category is not a review.
  Work the checklist.
- **Independent judgment.** Don't defer to the implementing agent's reasoning.
  Review the code as if you've never seen it before.

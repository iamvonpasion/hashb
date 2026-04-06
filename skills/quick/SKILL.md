---
description: >
  Quick planning — single-response plans with no ceremony.
  Auto-detects intent or accepts explicit mode.
    /quick           — auto-detect (build, research, fix, design, spec)
    /quick eng       — lightweight implementation plan
    /quick spec      — lightweight requirements
    /quick design    — lightweight UI/UX plan
    /quick research  — lightweight options comparison
    /quick fix       — lightweight debug plan
    /quick review    — lightweight code scan
---

# Quick

One response. No gates. No phases. No ceremony.

Auto-detects what you need, or use an explicit mode. If the task is too
complex for a quick plan, redirects to the appropriate full skill.

**This is read-only. No code changes.**

---

## Mode Detection

When no mode is specified, detect from user input:

| Signal | Mode |
|--------|------|
| "build", "implement", "add", "create", code-related | `eng` |
| "what should we build", requirements, stakeholder needs | `spec` |
| "UI", "wireframe", "design", "layout", "screen" | `design` |
| "evaluate", "compare", "which library", "options" | `research` |
| "broken", "bug", "error", "fix", "debug" | `fix` |
| "review", "check this code", "look at my PR" | `review` |
| Unclear or generic planning | `generic` |

State the detected mode in the output header so the user knows which
template is being used.

---

## Before You Output

1. **Read the relevant code** — scan the files and codebase context related to
   the request. Don't guess at file structure or existing patterns.
2. **Check Project Profile** — if CLAUDE.md has a Project Profile, use it to
   inform stack-specific advice.
3. **Assess complexity** — if the task clearly needs a full skill (see
   Escalation below), redirect immediately instead of producing a thin plan.

---

## Output Templates

### quick eng

```
QUICK ENG ───────────────────────────────────────────────────────

  Goal        {one sentence}
  Approach    {one sentence}
  Files       {file list with what changes in each}

  STEPS
  ─────────────────────────────────────────────────
  1. {step}
  2. {step}

  TESTS
  ─────────────────────────────────────────────────
  - {what to test}

  RISKS
  ─────────────────────────────────────────────────
  - {risk — or "None"}

  ➤ NEXT: implement, or /eng for full ceremony
─────────────────────────────────────────────────────────────────
```

### quick spec

```
QUICK SPEC ──────────────────────────────────────────────────────

  Problem     {one sentence}
  Users       {who is affected}

  STORIES
  ─────────────────────────────────────────────────
  1. As a {role}, I want {what} so that {why}
  2. ...

  ACCEPTANCE CRITERIA
  ─────────────────────────────────────────────────
  - [ ] {testable criterion}
  - [ ] ...

  OUT OF SCOPE
  ─────────────────────────────────────────────────
  - {what we're NOT doing}

  ➤ NEXT: /eng or /quick eng
─────────────────────────────────────────────────────────────────
```

### quick design

```
QUICK DESIGN ────────────────────────────────────────────────────

  Feature     {what}
  Screens     {count}

  SCREEN MAP
  ─────────────────────────────────────────────────
  1. {screen} — {purpose} — {new | existing}
  2. ...

  COMPONENTS
  ─────────────────────────────────────────────────
  New:    {list}
  Reuse:  {list}

  KEY STATES
  ─────────────────────────────────────────────────
  - {loading / empty / error / success states to handle}

  ➤ NEXT: implement, or /design for full wireframes
─────────────────────────────────────────────────────────────────
```

### quick research

```
QUICK RESEARCH ──────────────────────────────────────────────────

  Question    {what we're evaluating}

  OPTIONS
  ─────────────────────────────────────────────────

| Option | Pros | Cons | Effort |
|--------|------|------|--------|
| {A} | {1-line} | {1-line} | S/M/L |
| {B} | {1-line} | {1-line} | S/M/L |
| {C} | {1-line} | {1-line} | S/M/L |

  ➤ PICK: {recommendation + one sentence why}

  ➤ NEXT: implement, or /research for deep-dive
─────────────────────────────────────────────────────────────────
```

### quick fix

```
QUICK FIX ───────────────────────────────────────────────────────

  Symptom      {what's happening}
  Likely cause {best guess from evidence}

  INVESTIGATE
  ─────────────────────────────────────────────────
  1. {check this first}
  2. {then this}

  PROBABLE FIX
  ─────────────────────────────────────────────────
  {what to change and where}

  ➤ NEXT: implement fix, or /fix for full RCA
─────────────────────────────────────────────────────────────────
```

### quick review

```
QUICK REVIEW ────────────────────────────────────────────────────

  Scope       {N files, N lines changed}
  Verdict     {LGTM | CONCERNS | NEEDS WORK}

  FINDINGS
  ─────────────────────────────────────────────────
  - [{severity}] {finding} — {file:line}
  - ...

  ➤ NEXT: address findings, or /review for full review
─────────────────────────────────────────────────────────────────
```

### generic (no mode matched)

```
QUICK PLAN ──────────────────────────────────────────────────────

  Goal        {one sentence}
  Approach    {one sentence}

  STEPS
  ─────────────────────────────────────────────────
  1. {step}
  2. ...

  RISKS
  ─────────────────────────────────────────────────
  - {risk — or "None"}

  ➤ NEXT: {suggested action}
─────────────────────────────────────────────────────────────────
```

---

## Escalation

If the task is genuinely complex, don't produce a thin plan. Redirect:

| Signal | Redirect to |
|--------|------------|
| 8+ files to touch | `/eng` |
| Multiple architecture decisions needed | `/eng arch` |
| Unknown problem space, needs discovery | `/spec discover` |
| Need formal wireframes + state machines | `/design` |
| Need library verification via Context7 | `/research` |
| Bug requires instrumentation to diagnose | `/fix` |
| PR needs confidence-scored blocking review | `/review` |

**Escalation format:**

```
⚠ This needs the full skill.
  Reason: {why — too many files, arch decisions needed, etc.}
  Run:    /eng (or whichever full skill)
```

---

## Rules

- **Read-only** — no code changes, planning only
- **Be opinionated** — one approach, no "it depends", no listing alternatives
- **One response** — no stops, no gates, no "shall I continue?"
- **Under 80 lines output** — if your plan is longer, the task needs a full skill
- **Read before planning** — scan relevant code, don't guess at file structure
- **No Context7** — this is a quick pass. Use `/research` for library verification
- **No TDD checkpoints** — just list what to test. Use `/eng` for formal TDD plans
- **Escalate honestly** — a thin plan for a complex task is worse than redirecting

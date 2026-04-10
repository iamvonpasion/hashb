---
name: retro
description: >
  Post-ship retrospective — what worked, what didn't, root causes, action items.
  Use after shipping to reflect on the development process. Analyzes planning accuracy, quality gates, surprises, and persists top learnings as memories.
---

# Retrospective

Run after `/ship` lands. Capture what happened so the next cycle is better.

**Input:** PR URL or branch name (just shipped).
**Output:** Retro report with scored findings and concrete action items.

---

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Phase 1: Gather the Record

Reconstruct what actually happened during this cycle.

```bash
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

# What was planned vs what shipped
gh pr view --json title,body,commits,reviews,comments

# Timeline: first commit to PR merge
git log $BASE..HEAD --format="%h %ai %s" --reverse

# Scope: how much changed
git diff $BASE...HEAD --stat
git diff $BASE...HEAD --numstat | awk '{added+=$1; removed+=$2} END {print "+" added " -" removed " lines"}'

# Fix loops: how many qa/fix cycles happened
git log $BASE..HEAD --oneline --grep="fix" --grep="qa" --grep="review"
```

**Collect from conversation context (if available):**
- Original requirements / user request
- `/eng` review findings
- `/review` findings (if used)
- `/qa` report(s) — how many cycles, what failed
- `/fix` reports — root causes found
- `/ship` pre-landing review findings

---

## Phase 2: Analyze

Score each dimension. Be honest — the point is learning, not credit.

### 2A. Planning Accuracy

| Question | Answer |
|----------|--------|
| Did the plan match what was built? | Yes / Partial / No |
| Files planned vs files touched | N planned, M actual |
| Scope creep? | None / Minor / Major — what was added |
| Were the right skills used in the right order? | Yes / No — what should change |

### 2B. Quality Gates

| Gate | Result |
|------|--------|
| `/eng` review | Caught N issues / Missed N issues found later |
| `/review` peer review | Caught N issues / Skipped |
| `/qa` first pass | Score: N/100 — N issues found |
| `/qa` cycles needed | N cycles before clean |
| `/fix` root causes | N bugs — were any preventable? |
| `/ship` pre-landing | N issues found at the last gate |

**Red flag:** Issues found at `/ship` pre-landing that should have been caught
earlier. This means an upstream gate has a blind spot.

### 2C. What Went Well

List 2-5 things that worked. Be specific — "tests caught the regression"
is better than "testing was good."

### 2D. What Didn't Go Well

List 2-5 things that caused friction, rework, or risk. For each:

```
PROBLEM:     [what happened]
ROOT CAUSE:  [why it happened — process, skill gap, missing gate, unclear requirement]
IMPACT:      [rework, delay, risk shipped, near-miss]
```

### 2E. Surprises

Anything unexpected — good or bad. Edge cases nobody anticipated,
tools that didn't work as expected, assumptions that were wrong.

---

### Phase 2 Completeness Check

> **STOP.** Before proceeding to Phase 3, verify all five subsections are addressed.
> Every subsection must have at least one finding or an explicit "N/A — {reason}".
> Do NOT skip a subsection without marking it.

```
PHASE 2 CHECKLIST:
  2A. Planning Accuracy:  {done | N/A}
  2B. Quality Gates:      {done | N/A}
  2C. What Went Well:     {done | N/A}
  2D. What Didn't Go Well:{done | N/A}
  2E. Surprises:          {done | N/A}
```

---

## Phase 3: Action Items

Convert findings into concrete, actionable improvements.

**Categories:**

| Category | Example |
|----------|---------|
| **Skill improvement** | "/eng should check X — we missed it twice" |
| **Rule addition** | "Add a rule for Y pattern — it caused a bug" |
| **Process change** | "Run /research before /eng when touching Z" |
| **Knowledge gap** | "Team needs to understand X better" |
| **Tooling** | "Add linting for X to catch this automatically" |

**For each action item:**

```
ACTION:      [what to do]
CATEGORY:    [skill | rule | process | knowledge | tooling]
PRIORITY:    P1 (do now) | P2 (next cycle) | P3 (backlog)
OWNER:       [who — or "any agent"]
EVIDENCE:    [link to the specific problem this prevents]
```

**Rule:** Every "what didn't go well" must produce at least one action item.
If you can't think of one, the root cause analysis isn't deep enough.

### Phase 3 Validation

> **STOP.** Before proceeding to Phase 4, verify every "what didn't go well"
> from Phase 2D has a corresponding action item. Count them:

```
ACTION ITEM COVERAGE:
  2D findings:    {N}
  Action items:   {N}
  Unaddressed:    {list any 2D findings without actions}
```

> If any 2D finding lacks an action item, go back and add one before proceeding.

---

## Phase 4: Report

Write to `.retro/retro-{branch}-{date}.md`:

```
✓ RETROSPECTIVE ─────────────────────────────────────────────────

  Branch         {branch}
  PR             {PR URL}
  Date           {date}
  Scope          +{added} -{removed} lines, {files} files
  QA cycles      {N}
  Fix cycles     {N}

  PLANNING ACCURACY
  ─────────────────────────────────────────────────
  Plan vs reality    {match score}
  Scope creep        {none | minor | major}
  Skill sequence     {correct | should change: ...}

  QUALITY GATE EFFECTIVENESS
  ─────────────────────────────────────────────────
  /eng               {caught N / missed N}
  /review            {caught N / skipped}
  /qa first pass     {score}/100
  /ship pre-landing  {N issues}
  Blind spots        {list gates that missed issues}

  WHAT WENT WELL
  ─────────────────────────────────────────────────
  1.  ...
  2.  ...

  WHAT DIDN'T GO WELL
  ─────────────────────────────────────────────────
  1.  PROBLEM:     ...
      ROOT CAUSE:  ...
      IMPACT:      ...

  SURPRISES
  ─────────────────────────────────────────────────
  1.  ...

  ACTION ITEMS
  ─────────────────────────────────────────────────
  P1:  ...
  P2:  ...
  P3:  ...

  PATTERNS TO WATCH
  ─────────────────────────────────────────────────
  {recurring themes across retros — check prior retros}

─────────────────────────────────────────────────────────────────
```

### Cross-Retro Patterns

If prior retros exist in `.retro/`, scan them:

```bash
ls .retro/retro-*.md 2>/dev/null | head -10
```

Look for recurring themes:
- Same root cause appearing in multiple retros → systemic issue, escalate priority
- Action items from prior retros that weren't addressed → flag as overdue
- Improving trends → note what's working

---

## Phase 4.5: Persist Learnings (MANDATORY by default)

After writing the retro report, extract the top actionable learnings and
persist them so future conversations can benefit.

**This phase runs by default.** To skip, the user must explicitly pass
`--no-memory` when invoking `/retro`. Do not skip silently.

### What to Persist

From the retro findings, extract learnings that are:
- **Non-obvious** — not derivable from reading the current code
- **Reusable** — will apply to future work in this project
- **Actionable** — a future skill invocation can use this information

### How to Persist

1. **Save as Claude Code memories** — use the memory system to create:
   - **Feedback memories** for process learnings ("Always do X when Y")
     - Example: "Always run /research before /eng when touching the payment module — retro from 2026-03-20 showed we picked a deprecated Stripe API"
   - **Project memories** for codebase-specific findings ("Watch out for X in Z")
     - Example: "The notification module has a recurring N+1 issue — flagged in 3 retros, treat as architectural debt"

2. **Cross-retro escalation** — when scanning prior retros:
   - Same root cause in **2 retros** → save as a project memory with warning
   - Same root cause in **3+ retros** → escalate: suggest creating a new **rule file**
     to enforce the fix automatically

3. **Limit to top 3 learnings** — don't flood memory with minor findings.
   Prioritize: P1 action items > recurring patterns > surprising findings.

### Output

```
LEARNINGS PERSISTED
─────────────────────────────────────────────────
  1.  [feedback]  {learning summary}
  2.  [project]   {learning summary}
  3.  [project]   {learning summary} — RECURRING (3x), suggest rule

  Skipped: {N findings not significant enough to persist}
```

---

## Phase 5: Apply (Optional — User Decides)

If action items include skill or rule improvements, offer to apply them:

```
Retro produced N action items. Apply now?
A) Apply P1 items — update skills/rules in this session
B) Save report only — I'll handle the improvements later
C) Review each — walk me through them one by one
```

**If applying:**
- Skill changes → edit the relevant SKILL.md
- Rule additions → create or update the relevant rule file
- Process changes → update CLAUDE.md or workflow SKILL.md
- Commit improvements separately from the feature work

### Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Action items include rule/skill changes | Apply changes in this session | Improve the toolkit immediately |
| Starting a new feature | `/spec` | Begin the next cycle with lessons learned |
| Compliance drift suspected | `/audit` | Check overall repo health |

**Autonomous mode:** Retro is typically the last skill in a recipe chain.
Auto-persist learnings (Phase 4.5) unless `--no-memory` was passed.
Do not auto-proceed to another skill — retro is a reflection point.

---

## Rules

- **Honesty over comfort.** If the process failed, say so. Retros that say
  "everything went well" are useless.
- **Root causes, not blame.** "The gate missed it" not "the agent failed."
- **Concrete actions only.** "Be more careful" is not an action item.
  "Add X check to /eng Step 1" is.
- **Compare to prior retros.** Recurring issues need escalation, not repetition.
- **Never skip the action items.** A retro without actions is just a report.
- **Keep it short.** 5 minutes of reflection beats 30 minutes of ceremony.

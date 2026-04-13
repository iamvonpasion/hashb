---
name: grill
description: >
  Architecture fitness evaluation — optional, on-demand skill that challenges existing
  architecture and engineering decisions. Produces a scored report with ROI-ranked
  refactoring recommendations. Adversarial posture: challenges rather than describes.
  Requires .understand/architecture-map.md as input. Read-only.
argument-hint: "[scope or subsystem path]"
---

# Grill

Challenge the architecture and engineering decisions of an existing codebase.
Produce a scored fitness report with ROI-ranked refactoring recommendations.

**Writes `.grill/grill-{date}.md` only. No code changes.**

**This skill is OPTIONAL and on-demand.** It is never a required step in any recipe.
Users invoke it deliberately when they want an architecture fitness evaluation.

---

## When to Use

| Signal | Why grill? |
|--------|------------|
| "Is our architecture still fit for purpose?" | This is the core question grill answers |
| "Are we over-engineered?" | Simplicity audit identifies unjustified abstractions |
| "Should we refactor X?" | ROI matrix ranks which debts to pay down first |
| Before a major refactoring initiative | Challenge decisions before redesigning |
| Quarterly or annual architecture review | Periodic health check on the foundation |
| After significant growth (team, traffic, features) | Check whether original decisions still hold |
| Onboarding to a brownfield codebase (optional follow-up to `/understand`) | Go beyond mapping to evaluation |

**Don't use when:**
- You want a map of what exists — use `/understand`
- You want hashb rule compliance scoring — use `/audit`
- You want to design a new subsystem — use `/eng arch`
- You want to review code on a diff — use `/review`
- You want to simplify a specific file — use `/simplify`
- You have a specific question to investigate — use `/explore`

---

## Hard Boundaries

| Does NOT | Delegated to |
|----------|--------------|
| Map the architecture from scratch | `/understand` (required input) |
| Score hashb rule compliance | `/audit` |
| Design the replacement architecture | `/eng arch` (handoff) |
| Review code-level quality on a diff | `/review` |
| Apply code simplification | `/simplify` (handoff) |
| Modify any files (other than `.grill/` report) | — |

**Core principle:** Grill surfaces decisions worth revisiting and estimates ROI.
It does NOT design the replacement. That is `/eng arch`'s job, fed by this skill's findings.

---

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Execution Flow (MANDATORY)

```
Phase 0: Prerequisites
    |
    v
Phase 1: Inventory
    |
    v
Phase 2: Grill (5 evaluation passes)
    |
    v
STOP -- User reviews findings before synthesis
    |
    v
Phase 3: Synthesize (Fowler debt quadrant + ROI)
    |
    v
Phase 4: Report
    |
    v
STOP -- User reviews final report
```

Max 2 gates. Single session.

---

## Phase 0: Prerequisites

### Required Input

```bash
# The architecture map from /understand is a hard prerequisite.
ls .understand/architecture-map.md 2>/dev/null

# Project Profile for quality-attribute scenario selection
cat CLAUDE.md 2>/dev/null | head -80

# Existing ADRs (optional — enables Assumption Freshness pass)
ls -d docs/adr/ docs/adrs/ adr/ 2>/dev/null
ls docs/adr/*.md adr/*.md 2>/dev/null
```

### Prerequisite Gate

| Condition | Action |
|-----------|--------|
| `.understand/architecture-map.md` missing | **STOP.** Redirect: "Run `/understand` first. Grill evaluates what `/understand` maps." |
| Map exists but is stale (> 30 days old) | Warn user. Offer to re-run `/understand` or proceed with stale map. |
| Map exists and is current | Proceed. |
| No Project Profile | Proceed with degraded scenarios. Flag in report. |
| No ADRs found | Proceed. Assumption Freshness pass runs in "infer from code" mode. |

### Scope Card

Present before proceeding:

```
GRILL ──────────────────────────────────────────────────────────

  Repo             {name}
  Date             {date}
  Map source       .understand/architecture-map.md ({generated date})
  Profile          {Architecture | Platform | Stack from CLAUDE.md — or "none"}
  ADRs             {count — or "none found"}
  Modules          {N from architecture map}
  Focus            {full system | user-specified subsystem}

─────────────────────────────────────────────────────────────────
```

No gate. Proceed to Phase 1.

---

## Phase 1: Inventory

Build the Architecture Inventory from the map and codebase.

### Enumerate

For each module in the architecture map, collect:

| Column | Source |
|--------|--------|
| Module | From architecture map Module Boundaries |
| Declared pattern | From Project Profile Architecture field |
| Detected patterns | From convention detection — read 2-3 representative files per module |
| Dependencies | From architecture map Key Dependencies |
| Coupling level | From architecture map Coupling & Risk Areas |
| Age signal | Git history — first commit date, last substantive change |

Budget: max 5 files per module. Sample, don't exhaustively scan.

### Inventory Table

```
ARCHITECTURE INVENTORY
─────────────────────────────────────────────────
| Module | Pattern | Deps | Coupling | Age |
|--------|---------|------|----------|-----|
| {name} | {pattern} | {count} | {H/M/L} | {months} |
```

No gate. Proceed to Phase 2.

---

## Phase 2: Grill (5 Evaluation Passes)

> **All five passes are mandatory** unless prerequisites are missing (e.g., no ADRs).
> Skip a pass only by explicitly marking it "SKIPPED: {reason}" in the report.

See `references.md` for detailed per-pass checklists, signals, and scoring rubrics.

### Pass A — Simplicity Audit (Karpathy-inspired)

Challenge unjustified complexity. For each abstraction layer ask: "Is this justified
by actual variation, or is it speculative extensibility?"

**Countable signals** (every finding must cite a concrete signal, not vibes):

| Signal | What it suggests |
|--------|-------------------|
| Interface with 1 implementation | Speculative abstraction |
| Generic type instantiated with 1 concrete type | Speculative extensibility |
| Plugin/strategy system with 1 plugin | Speculative extensibility |
| Factory returning a single type | Unjustified indirection |
| Abstract class with 1 subclass | Premature abstraction |
| Wrapper functions that only call through | Accidental indirection |
| Configuration for values that never change | Speculative configurability |
| Layers beyond what the problem demands | "Layer cake" over-engineering |

**Senior Engineer Test per module:** "Would a senior engineer joining today say
this is overcomplicated for what it does?"

Record findings with file:line evidence.

### Pass B — Structural Fitness (ATAM-lite)

Define 3–5 quality-attribute scenarios derived from Project Profile. For each,
trace through the architecture: does the current structure support this well,
poorly, or not at all?

See `references.md` for scenario templates per stack (SaaS, CLI, API-only, library,
mobile, embedded).

**Per-scenario output:**

| Scenario | Support | Sensitivity Point | Trade-off |
|----------|---------|-------------------|-----------|
| {scenario} | {well/poor/none} | {where small changes cascade} | {what improving this would cost} |

### Pass C — Coupling / Cohesion Assessment

Go beyond `/understand`'s descriptive coupling map to evaluate: is this coupling
level *appropriate*?

| Check | Signal |
|-------|--------|
| Modules coupled but serving unrelated concerns | Should split |
| Modules decoupled but always changing together | Should merge (shotgun surgery) |
| God modules | Architectural hotspot, likely refactor target |
| Circular dependencies | Boundary violation |
| Dependency direction violations (e.g., domain → infrastructure) | Inverted dependency |
| Cross-module DB access | Leaky boundary |
| Public API surface disproportionate to module size | Missing encapsulation |

### Pass D — Technology Fit

For each major dependency / framework, evaluate fit against current constraints.

| Check | Signal |
|-------|--------|
| Over-powered tool for the problem | e.g., enterprise framework for CRUD app |
| Under-powered tool for the scale | e.g., SQLite for multi-tenant SaaS at scale |
| Deprecated / abandoned dependency still load-bearing | Hidden debt |
| Multiple libraries solving the same problem | Inconsistency, duplicate maintenance |
| Build/toolchain complexity disproportionate to app complexity | Meta-complexity tax |

If Context7 MCP is available, check deprecation status and current best practices
for each major dependency. Max 3 queries total.

### Pass E — Assumption Freshness

Challenge the original assumptions behind architectural decisions.

**If ADRs exist:** For each ADR, check whether the "Context" section is still true.
Flag ADRs where the stated constraint has changed (team size, scale, tech landscape,
business model).

**If no ADRs exist:** Infer likely original assumptions from the architecture itself.
Mark pass as degraded: "No ADRs — assumptions inferred from code structure, speculative."

| Assumption Type | Example |
|-----------------|---------|
| Scale assumption | "Designed for ~100 users" — is that still true? |
| Team assumption | "Designed for solo developer" — is the team now 10? |
| Tech assumption | "Designed when X library was the standard" — still is? |
| Business assumption | "Designed for single-tenant" — now multi-tenant? |

### Phase 2 Gate

> **STOP.** Present raw findings before synthesis.
>
> "Here are the raw findings from each pass. Before I synthesize and compute ROI,
> are there findings you want to override, dismiss as known trade-offs, or add
> context to?"

User may:
- Dismiss findings as deliberate trade-offs (mark "deliberate-prudent" in synthesis)
- Add context that changes severity
- Flag findings as blockers (force top of report)

---

## Phase 3: Synthesize

### Fowler Debt Quadrant Classification

Every finding is classified on two axes:

```
                    Deliberate           Inadvertent
                ┌─────────────────┬──────────────────┐
      Prudent   │ Known trade-off │ Honest learning  │
                │ (accept or plan)│ (fix when ready) │
                ├─────────────────┼──────────────────┤
    Reckless    │ Tech debt by    │ Carelessness     │
                │ choice (justify)│ (fix soon)       │
                └─────────────────┴──────────────────┘
```

See `references.md` for classification heuristics.

### ROI Scoring

For each finding:

| Axis | Values | Meaning |
|------|--------|---------|
| Effort | S / M / L | Rough refactoring cost (hours / days / weeks) |
| Value | H / M / L | Risk reduced + delivery unblocked |
| ROI rank | Derived | Sort: high-value + low-effort first |

> **AI-estimated effort is unreliable.** Label the Effort column explicitly as
> "rough estimate, validate with team before prioritizing."

### Fitness Scoring

Each pass scores 0–100. Deduction rubric in `references.md`.

- **Simplicity** — 100 − (N unjustified abstractions × weight)
- **Structural** — 100 − (N scenarios with poor/none support × weight)
- **Coupling** — 100 − (N coupling violations × weight)
- **Technology** — 100 − (N fitness concerns × weight)
- **Assumption Age** — 100 − (N stale assumptions × weight) or "degraded" if no ADRs

**Overall** — weighted average (Simplicity 25 %, Structural 25 %, Coupling 20 %,
Technology 15 %, Assumption Age 15 %).

---

## Phase 4: Report

Write to `.grill/grill-{date}.md`:

```
GRILL REPORT ────────────────────────────────────────────────────

  Repo             {name}
  Date             {date}
  Source           .understand/architecture-map.md ({generated date})
  Architecture     {declared from Profile}
  Modules          {N evaluated}

  FITNESS SCORES
  ─────────────────────────────────────────────────
  Simplicity       {score}/100    {N unjustified abstractions}
  Structural       {score}/100    {N scenario concerns}
  Coupling         {score}/100    {N coupling issues}
  Technology       {score}/100    {N fitness concerns}
  Assumption Age   {score}/100    {N stale assumptions | degraded — no ADRs}
  ─────────────────────────────────────────────────
  Overall          {weighted}/100

  TOP FINDINGS
  ─────────────────────────────────────────────────
  1.  [{severity}] {finding} — {recommendation}
  2.  [{severity}] {finding} — {recommendation}
  3.  [{severity}] {finding} — {recommendation}

  REFACTORING ROI MATRIX
  ─────────────────────────────────────────────────
  | Rank | Finding | Debt Type | Effort* | Value | Handoff |
  |------|---------|-----------|---------|-------|---------|
  |  1   | {name}  | {quadrant}| S/M/L   | H/M/L | /eng arch / /simplify |
  |  2   | ...     | ...       | ...     | ...   | ...     |

  * Effort is a rough estimate — validate with the team before prioritizing.

  DECISIONS TO REVISIT
  ─────────────────────────────────────────────────
  1. {decision}
     Why: {reason it should be revisited}
     ➤ /eng arch  {scope prompt}
  2. ...

  SIMPLIFICATIONS
  ─────────────────────────────────────────────────
  1. {module / layer}
     Signal: {countable signal — e.g., "interface with 1 impl"}
     ➤ /simplify {target}

  STRENGTHS
  ─────────────────────────────────────────────────
  - {What the architecture gets right — evidence-based}
  - {Keep explicit: deficit-only reports lose credibility}

  ➤ NEXT: {see Next Step below}

─────────────────────────────────────────────────────────────────
```

### Report Rules

- **Evidence is mandatory.** Every finding must cite a file:line, module, or pattern signal.
- **Strengths section is non-negotiable.** At least 2 strengths must be listed.
- **ROI matrix sorted by rank**, not alphabetical.
- **Handoff column is explicit.** Every actionable finding names `/eng arch` or `/simplify`.
- **Target 150–250 lines.** If longer, you are documenting implementation details
  — step back to system-level findings.

---

## Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Decisions to revisit exist | `/eng arch` per decision | Design the replacement |
| Simplifications identified | `/simplify` per target | Reduce complexity in specific modules |
| Only low-severity findings | No action needed | Architecture is healthy |
| User wants to track findings over time | Re-run `/grill` after changes | Score trend is the fitness function |

**Interactive mode (default):** State the top handoff as a recommendation:
"Grill complete. Overall fitness {score}/100. Top recommendation: `/eng arch` for
{highest-ROI decision}."

**Autonomous mode:** Do NOT auto-proceed. Grill is opt-in by design — the user
initiates, the user decides what to act on.

---

## Rules

| Rule | Why |
|------|-----|
| Requires `/understand` output | Don't duplicate mapping — build on it |
| Read-only | Only writes `.grill/grill-{date}.md` |
| Challenge, don't replace | Surface decisions; `/eng arch` designs the replacement |
| Evidence-based | Every finding cites a countable signal or file:line — no vibes |
| Include Strengths | Deficit-only reports breed noise and lose credibility |
| ROI-ranked, not alphabetical | Top of report = highest-ROI refactoring opportunity |
| Single session, ≤ 2 gates | Not a multi-day workshop |
| Respect Project Profile | SaaS vs CLI vs library get different quality scenarios |
| Label effort estimates as rough | AI-estimated effort is unreliable — don't pretend otherwise |
| Never auto-invoke | Always user-initiated; never inserted into Feature/Bugfix/Release recipes |
| Idempotent | Re-running produces a new dated report; prior reports preserved |

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "We already know our architecture has issues — we don't need a report" | Implicit knowledge evaporates between sessions and disagrees across teammates. A scored report is falsifiable evidence that survives staff turnover. |
| "The `/understand` map already covers this" | `/understand` describes. Grill challenges. The map says "Module X has high coupling." Grill says "That coupling is unjustified because Y, and refactoring it has high ROI because Z." |
| "We'll just extend `/audit` instead" | `/audit` scores hashb rule compliance. Architecture fitness is a different question with a different methodology. Blurring them dilutes both. |
| "Effort estimates are too unreliable to include" | They are unreliable, which is why they are labeled as rough estimates. A rough estimate beats no estimate when ranking refactoring priority. |
| "The Senior Engineer Test is subjective" | Yes, which is why every Simplicity finding must cite a countable signal (1 impl, 1 plugin, etc.) rather than pure judgment. The test frames the question; the signal grounds the answer. |
| "We don't have ADRs, so Assumption Freshness is useless" | The pass degrades gracefully to "infer from code." The output is labeled as speculative, but still surfaces assumptions worth challenging. |
| "Grill should be part of every brownfield onboarding" | No. It's opt-in by design. Forcing it on every onboarding is exactly the workflow bloat this skill was scoped to avoid. |

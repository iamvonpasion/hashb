---
name: simplify
description: >
  Code simplification — reduces complexity, eliminates duplication, improves naming, removes dead code.
  Preserves all functionality. Use after writing or modifying code, or point at any file/directory/function.
argument-hint: "[file, directory, or function — omit for changed files]"
---

# Simplify

Simplify code **without changing what it does**. Reduce complexity, remove
duplication, improve naming, eliminate dead code. Clarity is the metric —
not line count, not cleverness.

**Input:** Changed files (default) or explicit target (file, directory, function name).
**Output:** Simplified code with test-verified behavior preservation.

---

## When to Use

| Situation | Skill |
|-----------|-------|
| Code works but is complex, hard to read, or messy | **`/simplify`** |
| Full code review before shipping (correctness, security, design) | `/review` |
| Refactoring during a TDD cycle | `/tdd` (REFACTOR step) |
| Something is broken | `/fix` |
| Quick scan for obvious issues | `/quick review` |

**Skip when:** Code is already clean, or the change is a single-line fix where
`/review`'s surface-level quality notes are sufficient.

`/simplify` fills the gap between writing code and reviewing it.
`/tdd` refactors within a cycle. `/simplify` refactors across the scope.

---

## Presentation Rules

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Phase 1: Scope

Determine what to simplify.

### Target Detection

| Input | Scope |
|-------|-------|
| `/simplify` (no args) | Staged + unstaged + branch diff files |
| `/simplify src/auth/` | All source files in directory, recursively |
| `/simplify src/utils/helpers.ts` | Single file |
| `/simplify calculateTax` | Search for function/class by name, analyze its file |

```bash
# Branch/base detection — see skills/shared/preflight.md
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
if command -v gh >/dev/null 2>&1; then
  BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null \
    || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
    || echo "main")
else
  echo "WARNING: gh CLI not found — defaulting BASE to 'main'"
  BASE="main"
fi

# Default mode: changed files
STAGED=$(git diff --cached --name-only 2>/dev/null)
MODIFIED=$(git diff --name-only 2>/dev/null)
BRANCH_FILES=$(git diff $BASE...HEAD --name-only 2>/dev/null)
# Combine, deduplicate, filter to source files (exclude configs, lockfiles, generated files)
```

**Function-name targeting:** When the argument doesn't match a file or directory
path, treat it as a function/class/module name. Search the codebase, present
matches if ambiguous, and analyze the containing file.

**Empty scope:** If no files are in scope, stop:
"No changed files detected. Specify a target: `/simplify src/utils/` or `/simplify myFunction`"

**Read Project Profile** from consumer's `CLAUDE.md` (if present) — adapt
checks to the declared Stack.

**Output:** Scope summary — file count, target description, detected stack.

---

## Phase 2: Analysis

Analyze before changing. Work through each dimension in order.

### Graph-Aware Analysis (if code-review-graph MCP is available)

If the consumer's Project Profile lists `code-review-graph`, query blast
radius for each proposed change before applying it. Catches cases where a
rename or extraction ripples across many callers.

If code-review-graph is not available, fall back to grep-based analysis.
Don't block on graph availability.

### 2A. Complexity

- Functions exceeding 30 lines.
- Nesting deeper than 3 levels.
- Cyclomatic complexity smells (many branches/conditions).
- Long parameter lists (>4 params).
- God functions doing too many things.

### 2B. Duplication

- Identical or near-identical logic in multiple places.
- Copy-paste patterns with slight variations.
- Opportunities for extraction into shared helpers.
- Flag all instances with file:line references.

### 2C. Dead Code

- Unreachable branches.
- Unused imports/requires.
- Commented-out code blocks.
- Unused variables, functions, or type definitions.

### 2D. Naming

- Types, functions, variables named for *how* they work instead of *what*
  they represent.
- Abbreviations that obscure meaning.
- Inconsistent naming patterns within a module.

### 2E. Clarity

- Deeply nested ternaries or conditionals that could be guard clauses / early returns.
- Redundant logic (double-checking conditions already guaranteed).
- Unnecessary comments that restate the code (keep comments that explain *why*).
- Complex expressions that could be named intermediate variables.
- Consolidatable logic (multiple branches doing similar work).

### Stack-Specific Checks

Adapt to the consumer's declared Stack:

| Stack | What to check |
|-------|--------------|
| TypeScript/React | Nested ternaries in JSX, prop drilling, missing named exports, arrow-function components that should be function declarations |
| Python | Manual loops replaceable by comprehensions (when readable), raw dicts replaceable by dataclasses, deep `if/elif/else` chains |
| Go | Deep error-check nesting (should be early returns), complex select/switch, repeated error handling boilerplate |
| C#/.NET | Classes replaceable by records, verbose switch replaceable by pattern matching, complex LINQ chains |
| Rust | Deep `match` arms, repeated `.unwrap()` patterns, manual iteration replaceable by iterators |
| Generic | Early returns, guard clauses, extract-then-use, single responsibility |

When no Stack is declared, use language-agnostic heuristics only.

### Findings Format

For each issue found:

```
FINDING #{N}
  ─────────────────────────────────────────────────
  Impact        HIGH | MEDIUM | LOW
  Confidence    HIGH | MEDIUM | LOW
  Category      complexity | duplication | dead-code | naming | clarity
  File          {path}:{line}
  Description   {what's wrong}
  Proposed      {specific simplification — before/after sketch}
```

**Impact definitions:**
- **HIGH** — Complexity that makes the code dangerous to modify. Significant
  duplication that will cause bugs when one copy is updated but not the other.
- **MEDIUM** — Meaningful simplification that improves maintainability.
- **LOW** — Minor clarity improvement. Nice-to-have.

### GATE: Confirm Findings

**Interactive mode:** Present findings summary and confirm before applying.
"Found {N} simplification opportunities ({H} high, {M} medium, {L} low impact).
Proceed with all, or select specific findings?"

**Autonomous mode (recipe chain):** Proceed automatically unless HIGH-impact
findings span more than 5 files. In that case, stop and present to user.

---

## Phase 3: Simplify

Apply approved findings. Process in this order to avoid cascading conflicts:

1. **Dead code removal** — least risk, simplifies remaining analysis
2. **Naming improvements** — low risk, high readability impact
3. **Duplication consolidation** — medium risk, may require extracting shared code
4. **Complexity reduction** — guard clauses, extraction, restructuring
5. **Clarity improvements** — cosmetic-level changes

### Principles

1. **Preserve functionality** — never change what the code does, only how
   it does it. Same inputs, same outputs, always.
2. **Apply project standards** — read coding standards from consumer's
   CLAUDE.md and Project Profile. Adapt to the project's stack and conventions,
   not hardcoded patterns.
3. **Clarity over brevity** — explicit code beats compact code. Don't create
   "clever" one-liners that are harder to debug.
4. **Maintain balance** — don't over-simplify. Don't combine too many concerns
   into single functions. Don't remove helpful abstractions that improve
   organization. Don't prioritize fewer lines over readability.
5. **Minimal diff** — smallest change that achieves the simplification.
   Touch only what the finding requires. Do not clean up adjacent code,
   refactor unrelated imports, or add features.

### Test After Each Change

Run relevant tests after each simplification (or batch of related changes).
If tests break:
1. Revert the change immediately.
2. Flag it as "SKIPPED — test regression."
3. Continue with remaining findings.

### No Test Suite Warning

If no test suite is detected, warn:
"No test suite detected. Cannot verify behavior preservation. Simplifications
will be applied but behavior preservation is not proven. Proceed at your own risk."

---

## Phase 4: Verify

1. **Run the full test suite.** Paste output. No regressions allowed.
2. **Diff review** — show what changed, grouped by simplification type.
3. If any test fails and was not caught in Phase 3, revert the offending
   change and report it as "REVERTED."

### Output

```
SIMPLIFY REPORT ────────────────────────────────────────────────

  Scope            {target description}
  Files analyzed   {N}
  Findings         {N} total ({H} high, {M} medium, {L} low)
  Applied          {N}
  Skipped          {N} (user deferred or test regression)
  Reverted         {N} (behavior changed)
  Tests            ALL GREEN | {failure count}

  APPLIED
  ─────────────────────────────────────────────────
  1.  {file}:{line} — {category}: {one-line description}
  2.  ...

  SKIPPED
  ─────────────────────────────────────────────────
  1.  {file}:{line} — {reason}

  Status           DONE | DONE_PARTIAL | NO_CHANGES

────────────────────────────────────────────────────────────────
```

### Next Step

| Status | Next Skill | Why |
|--------|-----------|-----|
| DONE | `/review` | Code simplified — now review for correctness, security, design |
| DONE_PARTIAL | `/review` | Some simplifications skipped — reviewer should note |
| NO_CHANGES | `/review` | Code already clean — proceed to review |

**Autonomous mode:** If running inside a recipe chain:
- DONE / DONE_PARTIAL / NO_CHANGES → auto-proceed to next skill in chain
- Test regressions that couldn't be resolved → stop chain, present to user

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "This whole file needs rewriting" | Simplify what's in scope. Full rewrites are `/eng` → `/tdd` territory. |
| "Fewer lines = simpler" | Fewer lines can mean denser, harder to debug. Clarity is the metric, not line count. |
| "I'll refactor the API while simplifying" | API changes are behavior changes. Simplify the implementation, not the interface. |
| "These tests are slow, let me skip them" | Tests are the safety net. Without them, you're guessing that behavior is preserved. |
| "This abstraction is unnecessary" | If it has callers outside this scope, it's not yours to remove. |
| "I know this preserves behavior, no need to test" | Confidence is not proof. Run the tests. |

---

## Rules

| Rule | Why |
|------|-----|
| Never change behavior | Simplification is not refactoring. Same inputs → same outputs, always. |
| Run tests after every change | Catch regressions immediately, not at the end. |
| Don't chase line count | 3 clear lines > 1 dense line. Clarity is the goal. |
| Respect existing abstractions | If an abstraction has callers, simplify inside it — don't flatten it. |
| One simplification type per pass | Don't rename variables AND restructure control flow in the same hunk. |
| Skip if uncertain | When unsure if a simplification preserves behavior, skip it and note why. |
| Scope discipline | Touch only what the finding requires. No adjacent cleanup, no feature additions. |

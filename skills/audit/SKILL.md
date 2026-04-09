---
name: audit
description: >
  Compliance audit — scan entire repo against hashb standards. Scored report with prioritized findings.
  Use to check how compliant a repo is. Produces a scored report (0-100) with per-category breakdown, prioritized findings, and actionable next steps.
---

# Compliance Audit

Scan the entire consumer repo against hashb standards. Produce a scored
compliance report with prioritized findings.

**This is read-only. No code changes.**

**Input:** Consumer repo (current working directory).
**Output:** Compliance report with per-category scores, prioritized findings, and next steps.

---

## When to Use

| Situation | Skill |
|-----------|-------|
| New repo adopting hashb | `/init` first, then `/audit` to verify |
| Periodic compliance check | **`/audit`** |
| Reviewing code on a branch | `/review` (diff-scoped, not repo-wide) |
| Testing running app behavior | `/qa` |

`/audit` fills the gap between `/review` (diff-scoped) and `/init` (scaffolding).
It answers: "How compliant is this repo against hashb standards right now?"

---

## Execution Flow (MANDATORY)

> **This is the ONLY valid sequence. Never skip or reorder phases.**
> Each gate marked **STOP** requires completion before proceeding.

```
Phase 0: Discover
    |
    v
STOP -- User confirms scope and applicable rules
    |
    v
Phase 1: Profile Compliance
    |
    v
Phase 2: Rule-by-Rule Scan (2A-2G, all mandatory)
    |
    v
Phase 3: Architecture Alignment
    |
    v
Phase 4: Gap Analysis
    |
    v
Phase 5: Report
```

---

See `skills/shared/formatting.md` for presentation rules (progress indicators, discussion chunking, table formatting).

---

## Phase 0: Discover

Gather context about the consumer repo.

```bash
# What exists?
ls CLAUDE.md README.md CHANGELOG.md TODOS.md .gitignore .claudeignore 2>/dev/null
ls -d rules/ rules/business/ .retro/ .qa-reports/ 2>/dev/null

# Detect stack from dependency files
ls package.json requirements.txt Pipfile pyproject.toml go.mod Cargo.toml \
   *.csproj *.sln composer.json Gemfile 2>/dev/null
```

**Read if they exist:** `CLAUDE.md`, `package.json` (or equivalent), test config files.

**Determine applicable rules:**

| File types detected | Rules to check |
|---------------------|----------------|
| Always | `security`, `testing`, `git`, `docs` |
| `rules/business/**` exists | Business rules |
| CI/CD files detected | `infrastructure/` |

Present discovery summary:

```
AUDIT SCOPE ─────────────────────────────────────────────────────

  Repo            {name}
  Stack           {detected from dependency files}
  Rules to check  {list}
  File types      {detected}

─────────────────────────────────────────────────────────────────
```

> **STOP.** Confirm scope with user. "I'll check against these rules. Any areas to skip or add?"

---

## Phase 1: Profile Compliance

Check CLAUDE.md and Project Profile completeness.

### 1A. CLAUDE.md

| Check | Status | Notes |
|-------|--------|-------|
| CLAUDE.md exists | {yes / missing} | — |
| Project Profile section exists | {yes / missing / incomplete} | — |
| Stack field | {populated / missing} | {value if present} |
| Architecture field | {populated / missing} | {value if present} |
| Tenancy field | {populated / missing} | {value if present} |
| Testing field | {populated / missing} | {value if present} |
| Deploy field | {populated / missing} | {value if present} |
| MCP Servers field | {populated / missing} | {value if present} |
| Context7 in MCP Servers | {present / missing — CRITICAL} | Required for research + plan skills |
| code-review-graph in MCP Servers | {present / missing — RECOMMENDED} | Enhances `/review`, `/fix`, `/eng`, `/audit`, `/swarm` with structural code intelligence |
| Skill Routing section | {present / missing} | Required for natural language skill invocation |

> **Context7 is required.** If the MCP Servers field is missing or does not
> include Context7, flag as **CRITICAL** — research and plan skills cannot
> function at full capability without it.
>
> **Skill Routing is required.** If the Skill Routing section is missing, flag
> as **GAP** — without it, skills only activate on explicit `/` commands.

### 1A-2. CLAUDE.md Leanness Analysis

A lean CLAUDE.md keeps only high-level pointers — Project Profile, Skill Routing,
and short references. Everything else belongs in rule files with `paths:` frontmatter
so it loads on-demand. This is critical for token efficiency and Claude's adherence
to instructions.

**Read the full CLAUDE.md and classify every section:**

| Section / Block | Lines | Classification | Recommended Action |
|-----------------|-------|---------------|-------------------|
| {section name} | {N} | {KEEP / EXTRACT / REMOVE} | {rationale} |

**Classification rules:**

| Classification | Criteria | Examples |
|---------------|----------|---------|
| **KEEP** | Essential every session, < 20 lines | Project Profile, Skill Routing, 1-line project description |
| **EXTRACT** | Useful but not needed every session, or > 20 lines | Architecture docs, coding standards, API conventions, workflow docs, long instruction blocks |
| **REMOVE** | Duplicates rule/skill content, stale, or auto-derivable | Pasted rule text, inlined skill docs, copy of README content |

**For each EXTRACT item, specify the extraction target:**

```
EXTRACTION PLAN
─────────────────────────────────────────────────
  Section: "{section name}" ({N} lines)
  Extract to: rules/{suggested-file}.md
  paths: ["{glob pattern matching relevant source files}"]
  Replace in CLAUDE.md with: "See rules/{suggested-file}.md"
```

**Severity thresholds:**

| CLAUDE.md state | Severity | Action |
|-----------------|----------|--------|
| < 200 lines, no inlined content | OK | No action |
| 200-300 lines OR some inlined content | **HIGH** | Extract bloated sections |
| > 300 lines OR large blocks inlined | **CRITICAL** | Mandatory refactor — Claude's instruction adherence degrades significantly |

> **Why this matters:** Claude Code loads CLAUDE.md into every conversation.
> A 500-line CLAUDE.md wastes tokens on every message and pushes important
> instructions out of the attention window. A lean CLAUDE.md with on-demand
> rules via `paths:` means Claude only loads what's relevant to the files
> being worked on.

### 1B. Rules Structure

| Check | Status | Notes |
|-------|--------|-------|
| `rules/` directory exists | {yes / no} | — |
| `rules/business/` exists | {yes / empty / no} | — |
| Business rules have valid frontmatter | {yes / issues / n/a} | — |
| Rule overrides present | {N found} | {list if any} |
| Override targets valid | {yes / broken refs / n/a} | — |

### 1C. Supporting Files

| Check | Status | Notes |
|-------|--------|-------|
| CHANGELOG.md exists | {yes / no} | {follows docs.md format?} |
| .gitignore exists | {yes / no} | {has hashb entries?} |
| .claudeignore exists | {yes / no} | {has recommended entries?} |
| .claudeignore excludes deps | {yes / no / n/a} | node_modules, vendor, .venv, etc. |
| .claudeignore excludes build output | {yes / no / n/a} | dist, build, .next, coverage, etc. |
| .claudeignore excludes lock files | {yes / no / n/a} | package-lock.json, yarn.lock, etc. |
| .claudeignore excludes secrets | {yes / no / n/a} | .env, .env.* (not .env.example) |
| TODOS.md exists | {yes / no} | {follows docs.md format?} |
| `.retro/` directory | {exists / missing} | — |
| `.qa-reports/` directory | {exists / missing} | — |

### 1D. Stray Document Detection

Scan the repo for documents that contain content overlapping with what hashb
expects to live in `rules/`, `CLAUDE.md`, or specific locations. These are files
that likely predate hashb adoption and should be consolidated.

**Scan for common stray document patterns:**

```bash
# Common documentation files that may overlap with hashb structure
ls CONTRIBUTING.md CODING_STANDARDS.md STYLE_GUIDE.md ARCHITECTURE.md \
   CONVENTIONS.md GUIDELINES.md DEVELOPMENT.md WORKFLOW.md STANDARDS.md \
   CODE_OF_CONDUCT.md TESTING.md SECURITY.md 2>/dev/null

# Docs directories with potentially overlapping content
ls -d docs/ doc/ documentation/ wiki/ guides/ 2>/dev/null

# Find markdown files outside of rules/ that may contain rule-like content
find . -maxdepth 3 -name "*.md" \
  -not -path "./rules/*" \
  -not -path "./node_modules/*" \
  -not -path "./.git/*" \
  -not -path "./.retro/*" \
  -not -path "./.qa-reports/*" \
  -not -path "./.audit/*" \
  -not -name "README.md" \
  -not -name "CLAUDE.md" \
  -not -name "CHANGELOG.md" \
  -not -name "TODOS.md" \
  -not -name "HASHB.md" \
  -not -name "LICENSE.md" \
  2>/dev/null
```

**For each document found, read it and classify:**

| Document | Lines | Overlaps With | Content Summary | Recommendation |
|----------|-------|---------------|-----------------|----------------|
| {file} | {N} | {target: e.g., `rules/security.md`, `CLAUDE.md Profile`, `rules/business/`} | {1-line summary} | {MERGE / MOVE / DELETE / KEEP} |

**Classification rules:**

| Classification | Criteria | Action for `/init` |
|---------------|----------|-------------------|
| **MERGE** | Content overlaps with an existing or expected hashb file — combine the useful parts into the hashb target | Merge unique content into target, delete original |
| **MOVE** | Content is useful but lives in the wrong location — should be a rule file with `paths:` frontmatter | Move to `rules/{name}.md` with proper frontmatter |
| **DELETE** | Content is fully duplicated by an existing hashb file, or is stale/outdated | Delete after user confirmation |
| **KEEP** | Content serves a purpose that hashb doesn't cover (e.g., legal docs, external contributor guides) | No action — document why it's kept |

**Overlap detection heuristics — read each candidate file and check for:**

| Signal | Likely overlaps with |
|--------|---------------------|
| Coding standards, style guide, conventions | `rules/conventions.md` or CLAUDE.md extract target |
| Security guidelines, OWASP references | `rules/security.md` |
| Testing guidelines, test patterns | `rules/testing.md` |
| Architecture docs, system design | `rules/architecture.md` or `rules/business/` |
| Git workflow, branching strategy | `rules/git.md` |
| Contributing guidelines with code standards | Split: process → CONTRIBUTING.md (KEEP), standards → `rules/` (MOVE) |
| API documentation, endpoint specs | `rules/api.md` or `rules/business/` |
| Development setup, workflow docs | `HASHB.md` or `README.md` |

**Severity:**

| State | Severity |
|-------|----------|
| No stray documents found | OK |
| 1-3 stray documents with partial overlap | MEDIUM |
| 4+ stray documents OR documents with significant duplication | HIGH |
| Critical content (security, architecture) scattered across multiple files | CRITICAL |

> **Report only.** List findings for `/init` to act on. Do not suggest manual fixes.

### 1E. Repo Standards

| Check | Status | Notes |
|-------|--------|-------|
| README.md exists | {yes / no} | — |
| README has setup/install section | {yes / no / n/a} | Can a new dev get running? |
| README has usage/running section | {yes / no / n/a} | How to start, test, build? |
| README has contributing section | {yes / no / n/a} | — |
| LICENSE file exists | {yes / no} | {type if present} |
| .env.example exists (if .env used) | {yes / no / n/a} | .env must be gitignored |
| .env is gitignored | {yes / no / n/a} | **CRITICAL** if .env exists and not ignored |
| Lock file committed | {yes / no / n/a} | package-lock.json / yarn.lock / pnpm-lock.yaml etc. |
| Formatting config exists | {yes / no} | .editorconfig / .prettierrc / eslint / etc. |
| HASHB.md (getting started) | {yes / no} | Team workflow guide |

---

## Phase 2: Rule-by-Rule Scan

For each applicable rule, **sample 5-10 representative files** matching the rule's
`paths:` patterns. This is a strategic audit, not an exhaustive lint.

### Graph-Aware Sampling (if code-review-graph MCP is available)

If the consumer's Project Profile lists `code-review-graph`, use cluster analysis
to select representative files from each module cluster rather than random sampling.
This ensures coverage of all architectural boundaries. Max 1 query for sample selection.

If code-review-graph is not available, sample from file paths as usual.

> **All subcategories (2A-2G) are mandatory.** Mark as "N/A" if the rule doesn't
> apply to this repo. Do NOT skip a category without explicitly marking it.

See `references.md` for the detailed check tables for each category (2A Security,
2B Testing, 2C Git, 2D Documentation, 2E Business Rules, 2F Infrastructure,
2G Context & Token Optimization).

---

## Phase 2.5: Structural Health Check

Quick-pass health assessment of overall repo structure and quality signals.
See `references.md` for the full health check matrix (11 dimensions).

**Scoring:** Each dimension scores 0 (critical), 5 (warning), or 10 (healthy).
Health score = sum / 110 × 100. Include in Phase 5 report.

---

## Phase 3: Architecture Alignment

Cross-reference the declared Project Profile against the actual codebase.

| Declared | Actual | Match? |
|----------|--------|--------|
| Stack: {from Profile} | {detected from deps} | {yes / mismatch: ...} |
| Architecture: {from Profile} | {observed patterns} | {yes / mismatch: ...} |
| Testing: {from Profile} | {test config files} | {yes / mismatch: ...} |
| Deploy: {from Profile} | {deploy config files} | {yes / mismatch: ...} |

### Graph-Aware Architecture Check (if code-review-graph MCP is available)

If the consumer's Project Profile lists `code-review-graph`, query the dependency
graph to verify that actual module boundaries and dependency directions match the
declared Architecture. Flag tightly coupled modules declared as independent, or
dependency directions that violate the declared pattern. Max 1 query.

If code-review-graph is not available, infer architecture from directory structure
and dependency files.

**Flag mismatches.** A declared "Vitest" but configured "Jest" means either the
Profile is stale or the config is wrong. Report it; don't judge which is correct.

**If no Project Profile exists:** Skip this phase. Note it as a CRITICAL finding
in the Gap Analysis — the profile is the foundation for all skill adaptation.

---

## Phase 4: Gap Analysis

Identify what is **missing** — not what's wrong (that's Phase 2), but what
should exist and doesn't.

### Missing Rules

| Gap | Why it should exist | Priority |
|-----|---------------------|----------|
| {rule that should exist} | {file types present but no matching rule} | HIGH / MEDIUM |

### Missing Overrides

| Path Pattern | Should Override | Reason |
|--------------|-----------------|--------|
| {e.g., test helpers} | testing | {relaxed rules for test utilities} |
| {e.g., generated code} | security | {auto-generated, not human-written} |
| {e.g., internal tools} | security | {internal-only, behind VPN} |

### Business Rules to Formalize

| Domain Area | Evidence | Suggested Rule |
|-------------|----------|----------------|
| {e.g., payment processing} | {files in src/payments/} | {validation patterns, audit logging} |

---

## Phase 5: Report

### Scoring

Per-category scoring (start at 100, deduct per severity). See `references.md`
for the full severity deduction table and category weight breakdown.

**Overall score:** Weighted average of all 10 categories (Profile, Security,
Testing, Git, Docs, Repo Standards, Architecture, Context, Health, Rule Coverage).

### Report Output

Write to `.audit/audit-{date}.md`:

```
✓ COMPLIANCE AUDIT ──────────────────────────────────────────────

  Repo             {name}
  Date             {date}
  Stack            {detected or declared}
  Rules checked    {N}
  Files sampled    {N}
  Overall score    {score}/100

  CATEGORY SCORES
  ─────────────────────────────────────────────────
  Profile          {score}/100
  Security         {score}/100
  Testing          {score}/100
  Git              {score}/100
  Documentation    {score}/100
  Repo Standards   {score}/100
  Architecture     {score}/100
  Context/Tokens   {score}/100
  Health           {score}/100
  Rule Coverage    {score}/100

  TOP FINDINGS
  ─────────────────────────────────────────────────
  1.  [{severity}] {finding} — {recommendation}
  2.  [{severity}] {finding} — {recommendation}
  3.  [{severity}] {finding} — {recommendation}

  QUICK WINS (fix these first)
  ─────────────────────────────────────────────────
  1.  {low-effort, high-impact finding}
  2.  ...
  3.  ...

  VIOLATIONS ({N} total)
  ─────────────────────────────────────────────────
  {rule violations — breaks an active rule}

  GAPS ({N} total)
  ─────────────────────────────────────────────────
  {missing rules, overrides, or coverage}

  ARCHITECTURE MISMATCHES
  ─────────────────────────────────────────────────
  {declared vs actual discrepancies}

  ➤ NEXT: /init (see Next Step below)

─────────────────────────────────────────────────────────────────
```

### Structured Output for /init

If `/init` will consume this report, include the INIT ACTIONS machine-readable
block. See `references.md` for the INIT ACTIONS output format.

---

## Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Structural issues found (Profile, rules, scaffolds) | `/init` | Fix structural compliance gaps |
| Code-level findings (security, testing) | `/fix` per finding | Address violations in code |
| Score is healthy | No action needed | Repo is compliant |

**Autonomous mode:** If running inside a recipe chain (e.g., Onboarding or Compliance):
- Structural issues → auto-proceed to `/init` with the INIT ACTIONS block as input
- After `/init` → auto-re-run `/audit` for verification pass
- Code-level findings are NOT auto-fixed — present to user for manual triage

---

## Rules

- **Read-only.** No file modifications. Document only.
- **Sample, don't exhaustively lint.** 5-10 representative files per rule
  category. This is a strategic audit, not a linter.
- **Score honestly.** A repo that doesn't use hashb patterns should score low
  on hashb compliance. That's the point of the audit.
- **Be specific.** Every finding needs a file:line reference or a concrete
  "missing X" statement. No vague "consider improving."
- **Distinguish violations from gaps.** A violation breaks an active rule.
  A gap means a rule should apply but doesn't exist yet. Both matter, but
  they require different fixes.
- **Respect the declared profile.** If the consumer declared "Testing: Jest"
  but uses Vitest, that's an architecture alignment issue, not a testing
  compliance issue.
- **Stay in scope.** This audits hashb compliance, not general code quality.
  General quality is `/review`'s job.
- **Never refuse a category.** If a rule category doesn't apply, mark it N/A
  with a reason. Don't silently skip it.

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "The repo is small — it doesn't need an audit" | Small repos accumulate compliance debt fastest because nobody's watching. An early audit prevents drift before it compounds. |
| "We just ran /init — everything should be compliant" | `/init` scaffolds structure; it doesn't verify usage. Code written after `/init` can violate every rule it set up. |
| "Sampling 5-10 files isn't representative" | Strategic sampling catches systemic patterns. If 3 of 5 sampled files violate a rule, the violation is likely repo-wide. |
| "The score doesn't matter — we know our weak spots" | Quantified scores track improvement over time. "We know" is unfalsifiable; a score is evidence. |
| "Some categories don't apply to us" | Mark them N/A with a reason. Silently skipping categories is how compliance gaps hide. |

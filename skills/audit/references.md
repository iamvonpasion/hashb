# Audit References

Detailed check tables and scoring criteria referenced by SKILL.md.

---

## Phase 2: Rule-by-Rule Check Tables

### 2A. Security (rules/security.md)

Sample source files. Check:

| Check | Files Sampled | Findings |
|-------|---------------|----------|
| Input validation at boundaries | {N} | {ok / violations} |
| Auth checks on endpoints/mutations | {N} | {ok / gaps} |
| Hardcoded secrets | {N} | {ok / found} |
| SQL injection / parameterized queries | {N} | {ok / violations} |
| XSS / output escaping | {N} | {ok / violations} |
| Sensitive data in logs | {N} | {ok / found} |
| Eval / dynamic code execution | {N} | {ok / found} |
| Dependency verification | {N} | {ok / concerns} |

### 2B. Testing (rules/testing.md)

| Check | Files Sampled | Findings |
|-------|---------------|----------|
| Tests exist for the project | {N} | {ok / missing — CRITICAL if none} |
| Test-to-source ratio | {N test files / M source} | {healthy > 0.3 / warning 0.1-0.3 / critical < 0.1} |
| Critical paths tested (auth, payments, data) | {sampled} | {ok / gaps} |
| Tests exist and co-located | {N} | {ok / missing} |
| Naming: describe behavior, not implementation | {N} | {ok / issues} |
| Edge cases: nil, empty, boundary, malformed | {N} | {ok / gaps} |
| Test isolation (no shared state) | {N} | {ok / violations} |
| No brittle mocks of internals | {N} | {ok / found} |

### 2C. Git (rules/git.md)

```bash
# Sample recent commits
git log --oneline -20
# Check branch naming
git branch --list | head -10
```

| Check | Findings |
|-------|----------|
| Conventional commit format | {ok / violations: N of 20} |
| Commit message quality | {ok / vague messages found} |
| Branch naming convention | {ok / non-standard names} |
| One logical change per commit | {ok / mixed commits found} |

### 2D. Documentation (rules/docs.md)

| Check | Findings |
|-------|----------|
| CHANGELOG format (reverse chronological, conventional categories) | {ok / issues} |
| TODOS.md format (P1-3 priority, S/M/L effort) | {ok / issues / missing} |
| Markdown standards (ATX headings, code block languages) | {ok / issues} |
| README quality (not just boilerplate, reflects actual project) | {ok / stale / minimal} |
| README setup instructions match actual stack | {ok / outdated / missing} |
| API/endpoint documentation (if service project) | {ok / partial / missing / n/a} |
| Inline comments on complex logic (sample 5-10 files) | {ok / excessive / absent} |

### 2E. Business Rules

**Skip if:** No `rules/business/` directory. Mark as "N/A".

| Check | Findings |
|-------|----------|
| Rule files have valid YAML frontmatter | {ok / issues} |
| `paths:` patterns match actual source files | {ok / stale patterns} |
| `description:` present and meaningful | {ok / missing} |
| Rules cover declared domain boundaries | {ok / gaps} |

### 2F. Infrastructure

| Check | Findings |
|-------|----------|
| CI/CD files exist (e.g., `.github/workflows/`) | {yes / no} |
| Infrastructure rules exist for CI patterns | {yes / gap} |
| Deploy configuration matches declared Deploy profile | {ok / mismatch / n/a} |

### 2G. Context & Token Optimization (rules/context.md)

Assess whether the repo follows on-demand loading and token hygiene standards.
Reference: [Anthropic Claude Code best practices](https://docs.anthropic.com/en/docs/claude-code).

**2G-1. CLAUDE.md Hygiene**

| Check | Measurement | Findings |
|-------|-------------|----------|
| CLAUDE.md line count | {N} lines | {healthy < 200 / warning 200-300 / critical > 300} |
| Large content inlined | {yes / no} | {ok / violation: rule text, skill docs, or code pasted in} |
| `@path` imports count | {N} | {ok ≤ 5 / warning 6+ / critical: importing path-scoped rules} |
| Project Profile present & inline | {yes / no} | {ok / missing} |
| Skills referenced by name only | {yes / no} | {ok / violation: skill content inlined or imported} |

**2G-2. On-Demand Loading**

| Check | Measurement | Findings |
|-------|-------------|----------|
| Rules with `paths:` frontmatter | {N}/{total} ({%}) | {healthy > 80% / warning 50-80% / critical < 50%} |
| Domain rules scoped to domain paths | {N}/{total} | {ok / catch-all patterns found} |
| Largest rule file line count | {N} lines | {ok < 100 / warning 100-150 / critical > 150} |
| Unconditional rules (no `paths:`) | {N} | {ok ≤ 3 / warning 4-5 / critical > 5} |
| Skills inlined or `@`-imported | {N} | {ok = 0 / violation: any found} |

**2G-3. Sub-File Structure**

| Check | Measurement | Findings |
|-------|-------------|----------|
| Sub-rules declare own `paths:` scope | {N}/{total} | {ok / gaps: list files missing paths} |
| Rule `paths:` match actual source files | {N}/{total} | {ok / stale: patterns matching no files} |
| No duplicate loading (import + paths:) | {yes / no} | {ok / violation: files loaded twice} |
| Business rules avoid catch-all `**/*` | {yes / no / n/a} | {ok / violation: catch-all on domain rules} |

---

## Phase 2.5: Structural Health Check

Quick-pass health assessment of overall repo structure and quality signals.

| Dimension | Healthy | Warning | Critical | Actual |
|-----------|---------|---------|----------|--------|
| CLAUDE.md size | < 200 lines | 200-300 lines | > 300 lines | {N} |
| Rule coverage (rules with paths vs total) | > 80% | 50-80% | < 50% | {%} |
| Skill files present for declared skills | All match | Minor gaps | Missing skills | {N}/{total} |
| Supporting files (.gitignore, .claudeignore, CHANGELOG, LICENSE) | All present | Some missing | Core missing | {list} |
| Repo standards (README sections, .env.example, lock file) | All met | Minor gaps | Core missing | {list} |
| HASHB.md getting started guide | Present | — | Missing | {yes / no} |
| Stray documents (overlapping with hashb structure) | 0 | 1-3 | > 3 | {N}: {list} |
| Dead references (CLAUDE.md → non-existent files) | 0 | 1-2 | > 2 | {N} |
| Rule frontmatter validity (id, description, paths) | All valid | Minor issues | Broken rules | {N}/{total} |
| Duplicate or overlapping rule paths | None | Minor overlap | Conflicting rules | {list} |
| Context budget risk | Low (lean CLAUDE.md, scoped rules) | Medium (some bloat) | High (monolithic, unscoped) | {assessment} |

**Scoring:** Each dimension scores 0 (critical), 5 (warning), or 10 (healthy).
Health score = sum / 110 × 100. Include in Phase 5 report.

---

## Scoring Details

Per-category scoring. Start at 100, deduct per finding:

| Severity | Deduction |
|----------|-----------|
| CRITICAL | -25 |
| HIGH | -15 |
| MEDIUM | -8 |
| LOW | -3 |

Minimum score per category: 0.

| Category | Weight | Score |
|----------|--------|-------|
| Profile completeness | 10% | {score}/100 |
| Security compliance | 20% | {score}/100 |
| Testing compliance | 15% | {score}/100 |
| Git conventions | 10% | {score}/100 |
| Documentation & standards | 10% | {score}/100 |
| Repo standards | 5% | {score}/100 |
| Architecture alignment | 10% | {score}/100 |
| Context & token optimization | 10% | {score}/100 |
| Structural health | 5% | {score}/100 |
| Rule coverage (gaps) | 5% | {score}/100 |

**Overall score:** Weighted average of all categories.

---

## Structured Output for /init

If `/init` will consume this report, include a machine-readable section:

```
INIT ACTIONS (for /init consumption)
─────────────────────────────────────────────────
  PROFILE      {complete | update | create}
  ROUTING      {present | missing}
  RULES        {ok | create business rules}
  OVERRIDES    {list of suggested overrides}
  CLAUDEIGNORE {present | incomplete | missing}
  SUPPORTING   {list of missing files}
  REPO_STD     {list: README sections, LICENSE, .env.example, lock file, formatting}
  HASHB_MD     {present | generate}
  STRAY_DOCS   {none | list: file → recommendation (MERGE/MOVE/DELETE/KEEP)}
```

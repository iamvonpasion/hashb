---
id: context
description: Token optimization, on-demand resource loading, and CLAUDE.md hygiene
paths:
  - "**/CLAUDE.md"
  - "**/.claude/rules/**"
  - "**/rules/**/*.md"
  - "**/skills/**/*.md"
---

# Context & Token Optimization Rules

Lean instruction files improve Claude's adherence, reduce cost, and prevent
important rules from being lost in noise. These rules align with
[Anthropic's official Claude Code guidance](https://docs.anthropic.com/en/docs/claude-code).

---

## Tier 1: CLAUDE.md Hygiene

| # | Rule | Rationale |
|---|------|-----------|
| C1 | **CLAUDE.md must stay under 200 lines** | Files beyond 200 lines degrade Claude's instruction adherence. Quality drops sharply past this threshold. |
| C2 | **Never inline large content into CLAUDE.md** | Reference via path (`See rules/security.md`) or `@path` import — never paste rule bodies, skill docs, or code into CLAUDE.md. |
| C3 | **Use `@path` imports for essential references** | `@docs/git-instructions.md` expands at session start. Use for content that every session needs but would bloat CLAUDE.md. Max depth: 5 imports. |
| C4 | **One sentence per instruction line** | Easier to scan, diff, and maintain. Matches the docs.md markdown rule. |
| C5 | **Project Profile is the only large inline block allowed** | The 6-field Profile block (~8 lines) is the exception — skills need it every session. Everything else should be referenced. |

---

## Tier 2: On-Demand Resource Loading

| # | Rule | Rationale |
|---|------|-----------|
| D1 | **Rule files must use `paths:` frontmatter for scoped activation** | Rules with `paths:` only load when Claude reads matching files. Rules without `paths:` load every session — use sparingly. |
| D2 | **Skills load on invocation only — never pre-load skill content** | Skill SKILL.md files should never be `@`-imported or inlined. They load when the user invokes `/skill-name`. |
| D3 | **Sub-files must declare their own `paths:` scope** | A rule in `rules/business/payments.md` must have `paths:` targeting payment source files. Never create catch-all `**/*` patterns on domain rules. |
| D4 | **Split rules that exceed 150 lines** | Large rules lose effectiveness. Split by concern (e.g., `security-auth.md` + `security-input.md`) with distinct `paths:` patterns. |
| D5 | **Recipes, changelogs, and docs stay as file references** | `See recipes.md for workflow chains` is correct. Never inline recipe content into CLAUDE.md. |

---

## Tier 3: Context Budget Awareness

| # | Rule | Rationale |
|---|------|-----------|
| E1 | **Audit and review skills must sample, not exhaustively scan** | 5-10 representative files per category. Exhaustive scans consume context that degrades analysis quality. |
| E2 | **Phase handoffs pass only required inputs** | Orchestrators must not forward full conversation history. Each phase receives its structured input only. |
| E3 | **Prefer CLI tools over MCP servers when equivalent — except Context7** | MCP tool listings consume context tokens on every call. Native CLI tools have zero listing overhead. Context7 is exempt: it provides live library docs unavailable via CLI. |
| E3a | **code-review-graph queries replace grep chains, not add to them** | When the graph answers a structural question (dependencies, call graph, module boundaries), skip the equivalent grep/read calls. Budget: max 3 queries per skill invocation unless the skill specifies otherwise. |
| E4 | **Use `/compact` before context reaches 70%** | At 70%+ context, precision degrades. At 85%+, hallucinations increase. Compact proactively. |
| E5 | **CLAUDE.md survives compaction — rules and skills do not** | Design critical instructions for CLAUDE.md placement. Ephemeral guidance belongs in rules (reloaded on file access) or skills (reloaded on invocation). |

---

## Anti-Patterns to Flag

| Anti-Pattern | Why It's Harmful | Fix |
|--------------|------------------|-----|
| Pasting full rule text into CLAUDE.md | Doubles token cost — rule loads via `paths:` AND from CLAUDE.md | Remove inline, keep `paths:` reference |
| Rule file without `paths:` frontmatter | Loads every session regardless of relevance | Add appropriate `paths:` glob patterns |
| Catch-all `paths: ["**/*"]` on domain rules | Domain rule loads on every file read, defeating on-demand loading | Narrow to specific source paths |
| Inlining skill SKILL.md into CLAUDE.md | Skill content loads every session instead of on-demand | Reference skill by name only |
| Single monolithic rule file >200 lines | Exceeds effective attention. Rules at the end get ignored | Split by concern with distinct paths |
| `@`-importing files that are also path-scoped rules | Double-loads: once at startup via import, once on file match | Remove the `@` import; let `paths:` handle it |

---

## Health Check: Structure Quality

Quick self-assessment for any hashb-compliant repo:

| Check | Healthy | Warning | Critical |
|-------|---------|---------|----------|
| CLAUDE.md line count | < 200 lines | 200-300 lines | > 300 lines |
| Rules with `paths:` | > 80% of rules | 50-80% | < 50% |
| Largest rule file | < 100 lines | 100-150 lines | > 150 lines |
| `@` imports in CLAUDE.md | 0-3 essential | 4-6 | > 6 (context bloat) |
| Skills referenced (not inlined) | All | Most | Any inlined |
| Domain rules scoped to domain paths | All | Most | Catch-all patterns |
| Unconditional rules (no `paths:`) | ≤ 3 | 4-5 | > 5 (always-load bloat) |

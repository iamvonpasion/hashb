---
description: Rules for documentation files — changelogs, TODOs, and markdown
paths:
  - "**/CHANGELOG*"
  - "**/TODOS*"
  - "**/*.md"
---

# Documentation Rules

## CHANGELOGs

- Reverse chronological order (newest first)
- Use conventional commit categories: Added, Changed, Fixed, Removed, Deprecated, Security
- Each entry: one line, past tense, references PR/issue if applicable
- Date format: YYYY-MM-DD

## TODOS.md

- Format: `- [ ] P{1-3} [{S|M|L}] Description`
- P1 = blocking or critical, P2 = important, P3 = nice-to-have
- S/M/L = effort estimate (Small/Medium/Large)
- Mark completed items with `[x]` and date: `- [x] (2024-01-15) Description`
- Keep completed items for one release cycle, then remove

### Extended format (from `/spec decompose`)

Items produced by `/spec decompose` add optional fields to the base format:

- **Task IDs:** `- [ ] P1 [M] #1 Description` — `#N` is a sequential ID for dependency tracking
- **Dependencies:** `(depends: #1, #2)` or `(depends: none)` appended to the description
- **Feature sections:** `### Feature: {name}` groups related tasks under a heading
- **Mini-spec:** indented sub-list with goal and acceptance criteria

```markdown
### Feature: Auth System

- [ ] P1 [M] #1 Auth middleware — JWT validation + refresh (depends: none)
  - Goal: Request authentication layer that validates and refreshes JWTs
  - AC:
    - [ ] Valid JWT passes through to handler
    - [ ] Expired JWT triggers refresh flow
    - [ ] Invalid JWT returns 401

- [ ] P1 [S] #2 User model + migrations (depends: none)
  - Goal: User entity with email, role, password hash
  - AC:
    - [ ] User CRUD operations work
    - [ ] Password is bcrypt-hashed, never stored plain
```

All extended fields are optional — existing TODOS.md items without `#N` IDs or
`(depends:)` refs continue to work unchanged. Skills that read TODOS.md
(`/fix`, `/eng`, `/ship`, `/audit`) should treat the extended fields as additive context.

### Completion metadata (for decomposed tasks)

When `/ship` completes a decomposed task, mark it with:

```markdown
- [x] P1 [M] #1 Auth middleware — JWT validation + refresh (depends: none)
  - Goal: Request authentication layer that validates and refreshes JWTs
  - AC:
    - [x] Valid JWT passes through to handler
    - [x] Expired JWT triggers refresh flow
    - [x] Invalid JWT returns 401
  - **Completed:** v1.2.0 (2024-01-15) — PR #42
```

**Rules:**
- Mark task `[x]` only when ALL acceptance criteria are `[x]`
- If some ACs are met but not all, the task stays `[ ]`
- Version, date, and PR reference come from the `/ship` session that completed the work
- Keep completed tasks in TODOS.md until the feature is fully done (all tasks `[x]`),
  then follow the standard retention rule (one release cycle)

### Active Work (session handoff)

When a recipe chain is in progress, maintain an `## Active Work` section at the
top of TODOS.md. Skills update this as they complete. Next session reads it to
resume.

```markdown
## Active Work

- **Branch:** feature/notification-prefs
- **Recipe:** Feature (spec → design → eng → tdd → review → qa → ship → retro)
- **Completed:** /spec, /eng
- **Current:** /tdd (cycle 2/5 — see .tdd-checkpoint-*)
- **Next:** /review
- **Key decisions:**
  - Using WebSocket for real-time (ADR-001)
  - Deferred push notifications to task #3
- **Blocked:** nothing
```

**Rules:**
- Only ONE active work section at a time (one active recipe per TODOS.md)
- `/ship` clears the Active Work section on successful PR creation
- `/retro` clears it after writing the retro report
- If no active recipe, omit the section entirely
- Active Work is a handoff aid, not a formal artifact — keep it terse

## General Markdown

- Use ATX headings (`#`), not Setext (underline)
- One sentence per line in prose (easier diffs)
- Code blocks must specify language for syntax highlighting
- Links: prefer reference-style for repeated URLs

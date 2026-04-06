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

## General Markdown

- Use ATX headings (`#`), not Setext (underline)
- One sentence per line in prose (easier diffs)
- Code blocks must specify language for syntax highlighting
- Links: prefer reference-style for repeated URLs

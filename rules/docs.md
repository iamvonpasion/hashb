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

## General Markdown

- Use ATX headings (`#`), not Setext (underline)
- One sentence per line in prose (easier diffs)
- Code blocks must specify language for syntax highlighting
- Links: prefer reference-style for repeated URLs

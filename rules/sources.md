---
id: sources
description: >
  Source-driven development — verify framework-specific code against official
  documentation. Detect stack versions, cite sources, flag unverified patterns.
  Prevents hallucinated APIs and deprecated patterns in generated code.
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.py"
  - "**/*.cs"
  - "**/*.go"
  - "**/*.rs"
  - "**/*.rb"
---

# Source-Driven Development Rules

## Core principle

Ground all framework-specific code in official documentation for the detected
version. Training data contains outdated patterns — verify before implementing.

## Version detection

- Read dependency files first (`package.json`, `requirements.txt`, `go.mod`,
  `Cargo.toml`, `*.csproj`, `Gemfile`) to identify exact versions
- State detected versions explicitly before writing framework-specific code
- If versions are ambiguous, ask — version determines correctness

## Source hierarchy

Use sources in this order — higher overrides lower:

1. **Context7 / MCP documentation servers** — verified, version-pinned
2. **Official documentation** — library/API docs for the detected version
3. **Official changelogs and migration guides** — breaking changes, deprecations
4. **Web standards** — MDN, web.dev, language specs

**Not authoritative:** Stack Overflow, blog posts, tutorials, AI-generated docs,
training data memories.

## Citation requirements

- Every framework-specific pattern needs a source reference
- Use full URLs with deep links when available
- In code: comment with source URL for non-obvious patterns
- In conversation: state the source and version when recommending an approach

## Handling conflicts

When official docs conflict with existing project code:

```
CONFLICT DETECTED:
Existing code uses {old pattern},
but {framework} {version} docs recommend {new pattern}.
(Source: {URL})

Options:
A) Use the modern pattern
B) Match existing code for consistency
→ Which approach do you prefer?
```

Surface conflicts — never decide silently.

## Unverified patterns

When official documentation cannot be found for a pattern:

```
UNVERIFIED: Official docs for this pattern not found.
This relies on training data and may be outdated.
Verify before production use.
```

Honesty about limitations is more valuable than false confidence.

## Common rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "I'm confident about this API" | Confidence is not evidence. Training data contains outdated patterns that appear correct but fail against current versions. |
| "Fetching docs wastes tokens" | Hallucinating an API costs more. Users debug for hours discovering the function signature changed. |
| "The docs won't cover this" | Missing documentation is valuable information — the pattern may lack official recommendation. |
| "This is simple, no docs check needed" | Simple patterns with wrong APIs become templates. Users copy the deprecated approach across the project before discovering the issue. |

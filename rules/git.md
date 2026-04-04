---
description: >
  Git workflow conventions — commit message format, branch naming, PR standards.
  Enforces conventional commits with type-specific templates: features require
  context and scope, bug fixes require root cause analysis and evidence.
  Always active (no path filter) since git operations span all file types.
---

# Git Workflow Rules

## Branch Naming

Format: `<type>/<short-description>`

| Type | Use when |
|------|----------|
| `feature/` | New functionality or capability |
| `fix/` | Bug fix |
| `hotfix/` | Urgent production fix |
| `chore/` | Maintenance, dependencies, config |
| `refactor/` | Code restructuring without behavior change |
| `docs/` | Documentation only |
| `test/` | Adding or updating tests only |
| `experiment/` | Exploratory work, spikes, prototypes |
| `research/` | Technology evaluation, library comparison, API investigation |
| `migrate/` | Data migration scripts and transformations |

Rules:
- Use lowercase kebab-case: `feature/add-user-auth`, not `Feature/AddUserAuth`
- Keep descriptions under 5 words
- Include ticket/issue number if available: `fix/GH-42-login-timeout`
- Never commit directly to `main` or `master`

## Commit Message Format

All commits use conventional commit format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

- **subject**: imperative mood, lowercase, no period, max 72 chars ("add auth middleware", not "Added auth middleware.")
- **scope**: optional, the module or area affected (`auth`, `api`, `ui`, `db`)
- **body**: wrapped at 80 chars, explains *what* and *why* (not *how* — the diff shows how)
- **footer**: references, co-authors, breaking changes

### Types

| Type | When to use |
|------|------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `docs` | Documentation changes |
| `chore` | Build, CI, dependencies, config |
| `perf` | Performance improvement |
| `style` | Formatting, whitespace (no logic change) |
| `revert` | Reverting a previous commit |

## Commit Templates

### Standard commits (feat, refactor, docs, chore, perf)

```
<type>(<scope>): <subject>

Context:
- Why this change is needed

Changes:
- <file or module>: <what changed>

Notes:
- Design decisions, trade-offs, or deployment considerations (if any)
```

### Bug fix commits (fix, hotfix)

Include root cause analysis. A fix without a known cause is a guess.

```
fix(<scope>): <what was fixed>

Root Cause:
- What was happening: <observed behavior>
- Why it was happening: <the actual cause, not symptoms>

Fix:
- <what was changed and why this addresses the root cause>

Evidence:
- <how the fix was verified — test output, reproduction steps, before/after>

Refs: #<issue-number>
```

## Commit Hygiene

- Each commit should be independently valid — no broken imports, no failing tests
- A source file and its test file belong in the same commit
- Group related changes logically — one coherent change per commit
- Small changes (< 50 lines, < 4 files) can be a single commit
- Larger changes should be split into bisectable chunks ordered by dependency:
  1. Infrastructure (migrations, config, schemas)
  2. Core logic (models, services, utilities)
  3. Interface (controllers, views, API routes)
  4. Tests (if not co-located with their source commits)
  5. Documentation (CHANGELOG, README, version bump)

## Pull Request Conventions

- PR title follows commit format: `<type>(<scope>): <summary>` (max 72 chars)
- One logical change per PR — avoid combining unrelated work
- PR description must include:
  - **Summary**: what changed and why (bullet points)
  - **Test plan**: how changes were verified
  - For bug fixes: root cause summary (abbreviated from commit)
- Keep PRs reviewable — aim for < 400 lines changed. Large PRs correlate with review quality degradation
- Link related issues with `Closes #N` or `Refs #N`

## Breaking Changes

If a commit introduces a breaking change:
- Add `BREAKING CHANGE:` in the commit footer with migration instructions
- Use `!` after the type/scope: `feat(api)!: change auth token format`
- PR description must include a migration guide

---
description: >
  Self-shipping rule — commit and push changes to this plugin repo after every
  modification session. Ensures no work is left uncommitted or unpushed.
globs:
  - skills/**/*
  - rules/**/*
  - .claude-plugin/plugin.json
  - CLAUDE.md
  - CHANGELOG.md
---

# Self-Shipping Rule

This repo is a live plugin. Changes only reach users when pushed to `main`.
After every modification session, commit and push — don't leave work stranded.

## Workflow

After making changes to any skills, rules, or plugin config:

1. **Stage only what changed.** Use specific file paths, not `git add -A`.
2. **Write a proper commit message.** Follow the conventions in `rules/git.md`:
   - Use conventional commit format: `<type>(<scope>): <subject>`
   - Scope should reflect what was touched: `research`, `git`, `plugin`, etc.
   - Subject is imperative, lowercase, no period, max 72 chars
   - Body explains *what* and *why* — the diff shows *how*
3. **Bump version if needed.** Follow `rules/versioning.md` — content changes
   get a minor or patch bump, with a CHANGELOG.md entry.
4. **Push to main.** This is the delivery mechanism. Unpushed = unshipped.
5. **Confirm success.** Run `git status` after push to verify clean state.

## Commit Scope Guide for This Repo

| What changed | Type | Scope | Example |
|-------------|------|-------|---------|
| New skill | `feat` | skill name | `feat(research): add Context7 integration` |
| Update skill content | `feat` or `fix` | skill name | `fix(guard): tighten destructive command list` |
| New rule | `feat` | rule name | `feat(versioning): add version bump rule` |
| Update rule | `fix` or `refactor` | rule name | `refactor(git): simplify commit templates` |
| Plugin config | `chore` | `plugin` | `chore(plugin): add new keyword` |
| Version bump + changelog | `chore` | — | `chore: bump version to 2.4.0` |
| CLAUDE.md update | `docs` | — | `docs: update skills table` |

## Rules

- **Never leave uncommitted changes.** If we modified files, we commit and push
  before the session ends.
- **One logical change per commit.** Don't lump unrelated skill and rule changes
  into a single commit. Version bump + changelog can be bundled with the feature
  commit or as a separate `chore` commit.
- **Always push.** A commit without a push is invisible to plugin users.
- **No force push to main.** Ever.
- **Verify after push.** `git status` should show clean working tree.

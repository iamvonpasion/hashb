---
description: >
  Plugin versioning rule — when and how to bump the version in plugin.json,
  marketplace.json, and CHANGELOG.md. Triggered when skills, rules, or plugin config change.
globs:
  - skills/**/*
  - rules/**/*
  - .claude-plugin/plugin.json
  - .claude-plugin/marketplace.json
  - CLAUDE.md
---

# Versioning Rule

When changes are committed to skills, rules, or plugin configuration,
bump the version in `.claude-plugin/plugin.json` **and**
`.claude-plugin/marketplace.json`, then update `CHANGELOG.md`.

The marketplace is required for plugin distribution — `claude plugin update`
reads the version from `marketplace.json`, not `plugin.json`. Both must match.

## When to Bump

| Change type | Version bump | Examples |
|-------------|-------------|----------|
| New skill or rule | Minor (x.**Y**.0) | Add `/deploy` skill, add `performance` rule |
| Meaningful skill/rule content change | Minor (x.**Y**.0) | Add Context7 to research, rewrite rule logic |
| Typo, formatting, wording tweak | Patch (x.y.**Z**) | Fix typo in skill, reword a rule sentence |
| Plugin config change (plugin.json) | Depends on impact | New keyword = patch, new field = minor |
| CLAUDE.md update | Patch (x.y.**Z**) | Update skill table, fix instructions |
| Breaking change (rename/remove skill) | Major (**X**.0.0) | Remove a skill, rename a skill namespace |

## How to Bump

1. Update `"version"` in `.claude-plugin/plugin.json`
2. Update `"version"` in `.claude-plugin/marketplace.json` — **both** the
   `metadata.version` and the plugin entry's `version` field must match
3. Add an entry to `CHANGELOG.md` at the top, under `## Unreleased` or a new
   version heading. Follow the format already established in that file.
4. Commit the version bump and changelog together with the feature commit,
   or as a separate `chore: bump version to x.y.z` commit — either is fine.

## Changelog Entry Format

```
## x.y.z — YYYY-MM-DD

### Added / Changed / Fixed / Removed
- Short description of what changed and why
```

## Rules

- Never skip the changelog — every version bump needs a corresponding entry.
- Use semantic versioning strictly. Don't bump major for non-breaking changes.
- If multiple changes land in one session, combine them into a single bump.

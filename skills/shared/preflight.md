# Shared Pre-flight: Branch & Base Detection

Canonical branch/base detection block. Skills reference this file instead of
duplicating the detection logic.

---

## Branch & Base Detection

```bash
# Detect current branch
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")

# Detect base branch (3-fallback chain)
# 1. Existing PR target  2. Repo default branch  3. Hardcoded fallback
if command -v gh >/dev/null 2>&1; then
  BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null \
    || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
    || echo "main")
else
  echo "⚠ gh CLI not found — defaulting BASE to 'main'. Install: https://cli.github.com"
  BASE="main"
fi

echo "BRANCH: $BRANCH  BASE: $BASE"
```

### Dependency Check

If `gh` is not installed, BASE defaults to `main` with a warning. This is
a reasonable fallback for most repos, but may be wrong for repos using
`master`, `develop`, or custom default branches.

Skills that need BASE for critical operations (merge, diff, PR creation)
should treat the warning as a soft blocker — proceed but note the assumption.

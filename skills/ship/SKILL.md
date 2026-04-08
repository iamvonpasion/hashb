---
name: ship
description: >
  Ship — test, review, version, changelog, commit, PR. Fully automated.
  Use when code is ready to ship. Runs tests, checks review status, bumps version, updates changelog, creates commit and PR. Say /ship and it handles the rest.
---

# Ship

Fully automated. The user said `/ship` — run straight through and output
the PR URL at the end.

**Only stop for:**
- On the base branch (abort)
- Merge conflicts that can't be auto-resolved
- Test failures
- Pre-landing review findings that need user judgment
- MINOR or MAJOR version bump (ask)

**Never stop for:**
- Uncommitted changes (include them)
- Version bump choice for MICRO/PATCH (auto-decide)
- CHANGELOG content (auto-generate)
- Commit message approval (auto-commit)

See `skills/shared/formatting.md` for presentation rules (progress indicators, discussion chunking, table formatting).

---

## Step 1: Pre-flight

```bash
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null \
  || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
  || echo "main")
echo "BRANCH: $BRANCH  BASE: $BASE"
```

If on the base branch: abort. "Ship from a feature branch."

**QA check:** Look for evidence that `/qa` was run this session (QA report in
conversation context, QA verdict, or explicit user skip). If no evidence found,
warn before proceeding:

```
⚠ NO QA DETECTED
─────────────────────────────────────────────────
No /qa report found in this session. The ship-readiness
pattern expects: /review → /qa → /ship.

Proceeding — but consider running /qa first.
─────────────────────────────────────────────────
```

If the user explicitly skipped QA (e.g., "skip QA", "just ship"), respect
the override and proceed without warning.

```bash
git status
git diff $BASE...HEAD --stat
git log $BASE..HEAD --oneline
```

---

## Step 2: Merge Base Branch

Merge base into feature branch so tests run against merged state:

```bash
git fetch origin $BASE && git merge origin/$BASE --no-edit
```

If merge conflicts: try auto-resolve for simple cases (VERSION,
CHANGELOG ordering). If complex, STOP and show conflicts.

---

## Step 2.5: Build & Lint

Read the consumer's Project Profile `Stack` field and detect build/lint
commands from the project toolchain:

| Stack signal | Build | Lint |
|---|---|---|
| .NET (`*.csproj`, `*.sln`) | `dotnet build` | `dotnet format --verify-no-changes` |
| Node (`package.json`) | `npm run build` (if script exists) | `npm run lint` (if script exists) |
| Python (`pyproject.toml`) | — | `ruff check` or `flake8` (detect from config) |
| Go (`go.mod`) | `go build ./...` | `golangci-lint run` (if installed) |
| Rust (`Cargo.toml`) | `cargo build` | `cargo clippy` |

**Build:** Run the detected build command. **If build fails: STOP.** Show
errors. Code that doesn't compile does not ship.

**Lint:** Run the detected lint command. If lint fails:
- Auto-fixable violations → fix silently, stage changes
- Non-fixable violations → **STOP.** Show violations.

If no build/lint commands detected for the stack, skip with a warning:
"No build/lint commands detected — consider adding stack info to Project Profile."

---

## Step 3: Run Tests

Run the project's test suite. If multiple suites exist, run in parallel.

```bash
# Detect and run whatever test infrastructure exists
# Examples: bin/test, npm test, pytest, cargo test, etc.
```

**If any test fails: STOP. Show failures. Do not proceed.**

If all pass: note counts briefly, continue.

---

## Step 3.5: Pre-Landing Review

**If `/review` was already run on this branch** (check conversation context or
commit messages for review verdicts), skip the full review — trust the verdict.
Only do a lightweight final check:

- **Secrets:** Any hardcoded credentials, tokens, or API keys in the diff?
- **Data safety:** Destructive migrations without rollback plan?
- **Mechanical issues:** Auto-fixable (trailing whitespace, import ordering)

**If `/review` was NOT run**, perform a full pre-landing check:

```bash
git diff origin/$BASE
```

Check for:
- **Security:** SQL injection, unescaped user input, hardcoded secrets,
  missing auth checks
- **Data safety:** Destructive migrations, missing backfills, schema issues
- **Code quality:** Dead code, N+1 queries, stale comments, missing error
  handling
- **Domain rules:** Check any active rules for domain-specific review criteria
- **If frontend files changed:** Accessibility gaps, missing responsive
  behavior, AI-slop patterns

Classify each finding:
- **AUTO-FIX:** Mechanical issues with obvious fixes → apply silently
- **ASK:** Issues needing judgment → batch into one question for user

If any fixes applied: commit them, then re-run tests (Step 3) before
continuing. If no issues: continue.

Output: `Pre-Landing Review: N issues — M auto-fixed, K asked`

---

## Step 4: Version Bump

Read `VERSION` file. Auto-decide bump level based on diff size as a heuristic
(teams can adjust thresholds — this is an opinionated default):

| Diff size | Bump |
|-----------|------|
| < 50 lines, trivial | MICRO (4th digit) |
| 50+ lines, bug fixes, features | PATCH (3rd digit) |
| Major feature / architecture | MINOR — **ask user** |
| Milestone / breaking change | MAJOR — **ask user** |

Bumping a digit resets all digits to its right to 0.

If no VERSION file exists, skip this step.

---

## Step 5: CHANGELOG

Auto-generate from all commits on the branch:

```bash
git log $BASE..HEAD --oneline
```

Categorize into: Added, Changed, Fixed, Removed.
Insert after file header, dated today.
Format: `## [X.Y.Z.W] - YYYY-MM-DD`

---

## Step 5.5: TODOS.md Update

If `TODOS.md` exists: cross-reference the diff against open items.
If the diff clearly completes a TODO, move it to the Completed section
with `**Completed:** vX.Y.Z (YYYY-MM-DD)`.

Be conservative — only mark items when the diff clearly shows the work
is done. Output summary of what was marked complete.

If `TODOS.md` doesn't exist: skip silently.

---

## Step 5.7: Merge Delta Specs

If `specs/deltas/` exists and contains delta spec files:

```bash
ls specs/deltas/*.md 2>/dev/null
```

For each delta spec, merge changes into the corresponding main spec:

1. **Read** the delta spec (ADDED/MODIFIED/REMOVED sections)
2. **Read** the target main spec in `specs/`
3. **Apply** changes:
   - ADDED requirements: append to the Requirements section
   - MODIFIED requirements: replace the matching requirement in-place
   - REMOVED requirements: delete the matching requirement
4. **Update** the "Last updated" line to today's date
5. **Move** the delta file to `specs/deltas/archive/{filename}`
6. **Stage** the changed files

If the target main spec doesn't exist (delta references a file that was
deleted or never created), warn and skip that delta.

If no deltas exist, skip silently.

---

## Step 5.8: Archive Artifacts

Preserve the full artifact chain for this branch in `.history/`:

```bash
mkdir -p .history
```

Collect artifacts produced during this feature's lifecycle and archive them:

```
.history/{branch-name}-{YYYY-MM-DD}/
  spec.md          # From /spec output (if produced this session)
  design.md        # From /design output (if produced)
  eng-plan.md      # From /eng output (if produced)
  tdd-plan.md      # From /eng TDD plan (if produced)
  review.md        # From /review verdict (if produced)
  qa-report.md     # From /qa report (if exists in .qa-reports/)
  retro.md         # From /retro report (if exists in .retro/)
  delta-specs/     # Copy of delta specs before merge (if any)
```

**Rules:**
- Only archive artifacts that actually exist — don't create empty placeholders
- Copy from conversation context (spec, design, eng, review outputs) and from
  on-disk reports (`.qa-reports/`, `.retro/`)
- If no artifacts exist beyond commits, skip archival silently
- Archive is for historical reference — never modify archived files after writing
- Stage the `.history/` directory for commit

This preserves the full decision chain: *why* we built it (spec), *how* we
designed it (design/eng), *whether* it was reviewed (review), *whether* it
works (QA), and *what we learned* (retro).

---

## Step 6: Commit (bisectable chunks)

Group changes into logical commits. Each commit = one coherent change.

**Ordering:**
1. Infrastructure (migrations, config, routes)
2. Models & services (with their tests)
3. Controllers & views (with their tests)
4. VERSION + CHANGELOG + TODOS.md (final commit)

**Rules:**
- A file and its test go in the same commit
- Each commit must be independently valid (no broken imports)
- Dependencies come first
- Small diffs (< 50 lines, < 4 files) → single commit is fine
- Only the final commit gets the co-author trailer:

```
chore: bump version and changelog (vX.Y.Z.W)

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Step 6.5: Verification Gate

**If ANY code changed after Step 3's test run** (review fixes, etc.):
re-run the full test suite. Paste fresh output.

"Should work now" is not evidence. Run it.

If tests fail: STOP. Do not push.

---

## Step 7: Push

```bash
git push -u origin $BRANCH
```

Never force push.

---

## Step 8: Create PR

```bash
gh pr create --base $BASE --title "<type>: <summary>" --body "..."
```

PR body includes:
- **Summary** — bullet points from CHANGELOG
- **Pre-Landing Review** — findings or "No issues found"
- **Test plan** — suite results with counts

Output the PR URL.

---

## Step 8.5: CI Status Check

After PR creation, check for CI pipeline status:

```bash
gh pr checks $BRANCH --watch --fail-fast 2>/dev/null || true
```

| CI status | Action |
|-----------|--------|
| Checks detected, passing | Report "CI: passing" — ship complete |
| Checks detected, running | Report "CI: running — monitor before merging" with checks URL |
| Checks detected, failed | **STOP.** Show failures. "CI failed — investigate before merging." |
| No checks detected | Warn "No CI checks detected — verify manually before merging" |

> Local test results are necessary but not sufficient. CI may catch
> environment differences, dependency resolution issues, or integration
> test failures that local runs miss. Never declare ship complete without
> noting CI status.

Final output: PR URL + CI status line.

---

## Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Ship successful | `/retro` | Capture what worked, what didn't, persist learnings |
| Ship blocked by test failures | `/fix` | Investigate and fix before retrying |
| Ship blocked by review findings | Address findings → re-run `/ship` | Fix issues first |

**Autonomous mode:** If running inside a recipe chain:
- PR created successfully → auto-proceed to `/retro`
- Blocked → stop the chain, present the blocker to user

---

## Rules

- **Never skip build.** If the project has a build step, it must pass.
- **Never skip lint.** If the project has a linter, it must pass.
- **Never skip tests.** If they fail, stop.
- **Never force push.**
- **Never push without fresh verification** if code changed after tests.
- **Split commits for bisectability.**
- **TODOS.md completion detection must be conservative.**
- **Report CI status.** After PR creation, check and report CI status. Never declare ship complete without noting it.
- **The goal:** user says `/ship`, next thing they see is the PR URL + CI status.

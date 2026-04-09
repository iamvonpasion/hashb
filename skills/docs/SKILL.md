---
name: docs
description: >
  Document hygiene — find stale, obsolete, duplicate, and scattered docs. Sync, consolidate, update, or remove.
  Use when docs feel stale, scattered, or bloated. Scans, classifies, and cleans documentation with per-document approval.
  Modes: /docs (full pass), /docs stale, /docs sync, /docs sweep.
argument-hint: "[\"stale\" or \"sync\" or \"sweep\"]"
---

# Document Hygiene

Scan, classify, and clean up documentation across the repo. Find stale content,
remove obsolete files, sync documents that have drifted, and consolidate scattered
docs into the right locations.

**Interactive.** Every destructive action requires user approval.

**Input:** Consumer repo (current working directory).
**Output:** Cleaned documentation with a summary of changes.

---

## When to Use

| Situation | Skill |
|-----------|-------|
| Periodic doc cleanup | **`/docs`** |
| First-time hashb setup | `/init` (handles stray doc consolidation during onboarding) |
| Compliance check | `/audit` (reports doc issues but doesn't fix them) |
| Writing new documentation | Not this — just write it directly |

`/docs` is the ongoing maintenance counterpart to `/init`'s one-time consolidation
and `/audit`'s read-only detection. Run it when docs feel stale, scattered, or bloated.

---

## Execution Flow (MANDATORY)

> **This is the ONLY valid sequence. Never skip or reorder phases.**
> Each gate marked **STOP** requires completion before proceeding.

```
Phase 1: Inventory
    |
    v
STOP -- User confirms scope
    |
    v
Phase 2: Classify
    |
    v
STOP -- User reviews findings
    |
    v
Phase 3: Act (per-document approval)
    |
    v
Phase 4: Verify & Report
```

---

See `skills/shared/formatting.md` for presentation rules (progress indicators, discussion chunking, table formatting).

---

## Phase 1: Inventory

Discover every document in the repo and understand the current state.

### 1A. Collect All Documents

```bash
# All markdown files (excluding vendor/generated)
find . -name "*.md" \
  -not -path "./node_modules/*" \
  -not -path "./.git/*" \
  -not -path "./vendor/*" \
  -not -path "./dist/*" \
  -not -path "./build/*" \
  2>/dev/null | sort

# Other doc formats
find . \( -name "*.txt" -o -name "*.rst" -o -name "*.adoc" \) \
  -not -path "./node_modules/*" \
  -not -path "./.git/*" \
  2>/dev/null | sort

# Doc directories
ls -d docs/ doc/ documentation/ wiki/ guides/ .github/ 2>/dev/null
```

### 1B. Collect Reference State

Gather the sources of truth that documents should be consistent with:

```bash
# Actual stack (for README/Profile staleness checks)
cat package.json 2>/dev/null | head -60
cat requirements.txt pyproject.toml go.mod Cargo.toml *.csproj 2>/dev/null | head -40

# Git history (for CHANGELOG sync)
git log --oneline -30

# Current CLAUDE.md Profile (for drift detection)
cat CLAUDE.md 2>/dev/null

# Existing rules (for overlap detection)
ls rules/**/*.md 2>/dev/null

# Recent file changes (to detect abandoned docs)
git log --diff-filter=D --name-only --pretty=format: -- "*.md" | head -20
```

### 1C. Present Inventory

```
DOC INVENTORY ───────────────────────────────────────────────────

  Total documents    {N}
  hashb-managed      {N} (CLAUDE.md, rules/, CHANGELOG, TODOS, HASHB.md)
  Project docs       {N} (README, CONTRIBUTING, LICENSE, etc.)
  Other docs         {N} (docs/, guides/, scattered .md files)
  Doc directories    {list}

─────────────────────────────────────────────────────────────────
```

> **STOP.** "I found {N} documents. Want me to check all of them, or focus on
> a specific area (e.g., just stale docs, just scattered docs, just sync)?"

---

## Phase 2: Classify

Read every in-scope document and classify it against multiple hygiene dimensions.

### 2A. Staleness Check

Compare document content against the current state of the codebase and git history.

| Document | Last Modified | Stale Signals | Verdict |
|----------|--------------|---------------|---------|
| {file} | {date or git blame} | {signals below} | {STALE / CURRENT / UNKNOWN} |

**Staleness signals — check for each document:**

| Signal | How to detect | Example |
|--------|--------------|---------|
| References deleted files/functions | Grep doc references against codebase | README mentions `src/old-module/` that doesn't exist |
| Version numbers outdated | Compare doc versions to package.json/lockfile | README says "Node 16" but package.json engines says ">=20" |
| Stack description mismatches actual deps | Compare Profile/README stack to dependency files | Profile says "Jest" but vitest.config.ts exists |
| Instructions reference removed commands | Check scripts in package.json / Makefile | README says `npm run migrate` but script doesn't exist |
| Date references in the past | Look for dates, "upcoming", "planned" | "Q3 2024 migration" — it's now past that |
| Git blame shows no updates in 6+ months | `git log -1 --format=%cr -- {file}` | Last touched 14 months ago on an active repo |

### 2B. Obsolescence Check

Identify documents that serve no current purpose.

| Document | Obsolescence Signal | Verdict |
|----------|-------------------|---------|
| {file} | {signal} | {OBSOLETE / ACTIVE / UNCLEAR} |

**Obsolescence signals:**

| Signal | How to detect |
|--------|--------------|
| Describes a completed migration | Content mentions "migrate from X to Y" and the repo only has Y |
| Temporary/draft markers | Filename contains `draft`, `tmp`, `old`, `backup`, `deprecated`, `archive` |
| Superseded by another doc | Two docs cover the same topic, one is newer/more complete |
| References removed features only | Every feature/module mentioned in the doc no longer exists |
| Empty or boilerplate-only | File exists but has no meaningful content (just headings, TODOs, or template text) |
| Orphaned — nothing links to it | No other doc, README, or CLAUDE.md references this file |

### 2C. Duplication & Scatter Check

Find content that exists in multiple places or lives in the wrong location.

| Document | Overlaps With | Overlap Type | Verdict |
|----------|--------------|-------------|---------|
| {file} | {target} | {DUPLICATE / PARTIAL / SCATTERED} | {MERGE / MOVE / KEEP BOTH} |

**Overlap detection:**

| Content type | Expected hashb location | Common stray locations |
|-------------|------------------------|----------------------|
| Coding standards | `rules/conventions.md` | `CONTRIBUTING.md`, `STYLE_GUIDE.md`, `docs/standards.md` |
| Security guidelines | `rules/security.md` | `SECURITY.md`, `docs/security.md` |
| Testing guidelines | `rules/testing.md` | `TESTING.md`, `docs/testing.md` |
| Architecture | `rules/business/` or `rules/architecture.md` | `ARCHITECTURE.md`, `docs/architecture.md` |
| Git workflow | `rules/git.md` | `CONTRIBUTING.md` (git section), `docs/git-workflow.md` |
| API documentation | `rules/api.md` or `rules/business/` | `docs/api.md`, `API.md` |
| Setup/onboarding | `README.md` + `HASHB.md` | `GETTING_STARTED.md`, `DEVELOPMENT.md`, `SETUP.md`, `docs/setup.md` |
| Changelog entries | `CHANGELOG.md` | `HISTORY.md`, `RELEASE_NOTES.md`, `NEWS.md` |

### 2D. Sync Check

Detect documents that should be consistent with each other but have drifted.

| Source of Truth | Document | Drift Detected | Severity |
|----------------|----------|---------------|----------|
| {source} | {doc} | {what's inconsistent} | {HIGH / MEDIUM / LOW} |

**Sync pairs to check:**

| Source of Truth | Should Be Consistent With | What to compare |
|----------------|--------------------------|-----------------|
| `package.json` / dep files | README (stack/version references) | Framework versions, Node/Python/Go version |
| `package.json` scripts | README (usage/running section) | Available commands (`npm run X`) |
| `package.json` / dep files | CLAUDE.md Project Profile (Stack field) | Declared vs actual stack |
| Test config files | CLAUDE.md Project Profile (Testing field) | Declared vs actual test framework |
| Deploy config files | CLAUDE.md Project Profile (Deploy field) | Declared vs actual deploy target |
| `git log` (recent tags/releases) | CHANGELOG.md | Missing changelog entries for shipped versions |
| Codebase structure (`src/*/`) | Business rules `paths:` patterns | Rule paths that match no files, or dirs with no rules |
| `rules/*.md` | CLAUDE.md references | Dead `See rules/X.md` links |
| `.env` variables | `.env.example` | Missing variables in example |
| `.gitignore` patterns | `.claudeignore` | Dirs ignored by git but not by Claude (deps, build, secrets) |

### 2E. Classification Summary

Present all findings grouped by severity:

```
DOC HYGIENE FINDINGS ────────────────────────────────────────────

  CRITICAL ({N})
  ─────────────────────────────────────────────────
  {findings that cause active confusion or broken workflows}

  HIGH ({N})
  ─────────────────────────────────────────────────
  {stale content that misleads, significant duplication}

  MEDIUM ({N})
  ─────────────────────────────────────────────────
  {minor staleness, partial overlaps, sync drift}

  LOW ({N})
  ─────────────────────────────────────────────────
  {cosmetic issues, orphaned but harmless docs}

  CLEAN ({N})
  ─────────────────────────────────────────────────
  {documents that passed all checks}

─────────────────────────────────────────────────────────────────
```

> **STOP.** User reviews findings. "These are the issues I found. Want me to
> proceed with fixes, or adjust the plan?"

---

## Phase 3: Act

Process each finding interactively. Group by action type for efficiency,
but approve each action individually.

### 3A. Remove Obsolete Documents

For each OBSOLETE document:

```
OBSOLETE: {file} ({N} lines)
─────────────────────────────────────────────────
  Why:      {obsolescence reason}
  Evidence: {what was checked}
  Risk:     {LOW — no references / MEDIUM — some references exist}

  {If references exist}
  References to update:
    - {file:line} — {the reference}

  Action? [delete / keep / skip]
─────────────────────────────────────────────────
```

### 3B. Update Stale Content

For each STALE document:

```
STALE: {file}
─────────────────────────────────────────────────
  Issue:     {what's stale}
  Current:   {what the doc says}
  Actual:    {what the codebase shows}
  Fix:       {proposed update}

  Action? [update / skip]
─────────────────────────────────────────────────
```

**For updates, show the diff before applying:**

```diff
- Node.js 16+ required
+ Node.js 20+ required
```

### 3C. Sync Drifted Documents

For each sync drift:

```
SYNC DRIFT: {doc} ↔ {source of truth}
─────────────────────────────────────────────────
  Source:    {source file} says {value}
  Document:  {doc file} says {different value}
  Fix:       Update {doc} to match {source}

  Action? [sync / skip]
─────────────────────────────────────────────────
```

### 3D. Consolidate Scattered/Duplicate Documents

For each MERGE or MOVE candidate:

```
CONSOLIDATE: {file} → {target}
─────────────────────────────────────────────────
  File:      {source path} ({N} lines)
  Target:    {hashb target path}
  Overlap:   {description of what overlaps}

  {If MERGE}
  Plan:
    - Unique content (lines {X-Y}): append to {target}
    - Duplicated content (lines {A-B}): discard
    - Delete {source} after merge
    - Update references in: {list of files}

  {If MOVE}
  Plan:
    - Move to: {target path}
    - Add frontmatter: paths: ["{glob}"]
    - Update references in: {list of files}

  Action? [merge / move / keep both / skip]
─────────────────────────────────────────────────
```

### 3E. Execution

Process approved actions in this order to avoid conflicts:

1. **Sync** — update content in place (no file moves)
2. **Update** — fix stale content in place
3. **Merge** — combine files, then delete originals
4. **Move** — relocate files
5. **Delete** — remove obsolete files
6. **Reference cleanup** — update all references to moved/deleted/merged files

After each action, verify the change is correct before proceeding to the next.

---

## Phase 4: Verify & Report

### 4A. Post-Action Verification

| Check | Status |
|-------|--------|
| No dead references in CLAUDE.md | {pass / fail} |
| No dead references in README.md | {pass / fail} |
| All rule `paths:` still match files | {pass / fail} |
| CHANGELOG.md in sync with recent releases | {pass / fail} |
| .env.example in sync with .env | {pass / fail} |
| No orphaned documents remain | {pass / fail} |
| CLAUDE.md still under 200 lines | {pass / fail} |

### 4B. Summary Report

```
✓ DOC HYGIENE COMPLETE ──────────────────────────────────────────

  REMOVED
  ─────────────────────────────────────────────────
  -  {file} — {reason}

  UPDATED
  ─────────────────────────────────────────────────
  -  {file} — {what changed}

  SYNCED
  ─────────────────────────────────────────────────
  -  {doc} ↔ {source} — {what was aligned}

  CONSOLIDATED
  ─────────────────────────────────────────────────
  -  {source} → {target} — {merged / moved}

  KEPT (no action)
  ─────────────────────────────────────────────────
  -  {file} — {reason}

  SKIPPED (user deferred)
  ─────────────────────────────────────────────────
  -  {file}

  STATS
  ─────────────────────────────────────────────────
  Documents scanned   {N}
  Issues found        {N}
  Issues resolved     {N}
  Documents removed   {N}
  Documents updated   {N}
  Documents moved     {N}

─────────────────────────────────────────────────────────────────
```

---

## Subcommands

| Command | Purpose |
|---------|---------|
| `/docs` | Full hygiene pass — all checks |
| `/docs stale` | Staleness check only — find and fix outdated content |
| `/docs sync` | Sync check only — align drifted documents with sources of truth |
| `/docs sweep` | Obsolescence + scatter check — remove and consolidate |

When a subcommand is used, skip irrelevant phases but keep the same
interactive approval flow.

---

## Rules

- **Interactive.** Every destructive action (delete, merge, move) requires
  individual user approval. Updates and syncs show diffs before applying.
- **Preserve content.** Merges and moves relocate content — they never discard
  unique information. Only true duplicates are removed.
- **Source of truth wins.** When syncing, the code/config/git is always the
  source of truth. Documents adapt to match reality, not the other way around.
- **Don't touch code.** This skill modifies documentation files only. If a
  doc issue reveals a code problem (e.g., missing scripts), report it — don't fix it.
- **Respect user decisions.** If a user says "keep" on something you flagged,
  accept it. Don't re-flag in the same session.
- **Be specific.** Every finding needs a concrete reference — file path, line
  number, the exact stale value vs. actual value. No vague "consider reviewing."
- **Idempotent within session.** Running `/docs` twice in a session should find
  no new issues (assuming no code changes between runs).
- **Git-aware.** Use git blame/log for modification dates, not filesystem timestamps.
  Check git history to distinguish "intentionally stable" from "forgotten."
- **Scope to docs only.** This is not a linter, not a code review, not an audit.
  It checks documentation health. Code quality is `/review`'s job, compliance is
  `/audit`'s job.

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "The docs are fine — nobody's complained" | Nobody complains about stale docs because they stop reading them. Silent drift is worse than loud complaints. |
| "Docs always go stale — cleaning them is Sisyphean" | Periodic cleanup prevents doc rot from compounding. A 15-minute `/docs` pass every release cycle keeps them current. |
| "I'll update the docs when the feature is done" | Post-ship doc updates happen 30% of the time. Sync docs with code changes in the same commit. |
| "Just delete the old docs — we'll rewrite later" | Deletion without consolidation loses institutional knowledge. Merge unique content first, then remove the duplicate. |
| "The README is good enough" | READMEs cover getting started. Architecture decisions, domain rules, and operational runbooks need dedicated docs that stay in sync with the codebase. |

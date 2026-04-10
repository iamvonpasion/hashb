---
name: init
description: >
  Bootstrap or update a repo for hashb compliance — Project Profile, rules, business scaffolds.
  Use when setting up a new repo for hashb or updating an existing one. Interactive setup: auto-detects stack, walks through configuration, scaffolds rules. Idempotent — safe to re-run.
---

# Init

Bootstrap or update a consumer repo for hashb compliance. Interactive --
asks about fields it can't auto-detect.

**Idempotent.** Running `/init` twice produces the same result. Never duplicates content.

**Input:** Consumer repo (current working directory), optionally an `/audit` report.
**Output:** Updated repo structure with hashb compliance scaffolding.

---

## When to Use

| Situation | Skill |
|-----------|-------|
| New repo adopting hashb | **`/init`** |
| Existing repo, check compliance first | `/audit` then `/init` |
| Review code | `/review` |

`/init` is the entry point for new hashb consumers. Run it once to set up,
run it again to update or fix drift.

---

## Execution Flow (MANDATORY)

> **This is the ONLY valid sequence. Never skip or reorder phases.**
> Each gate marked **STOP** requires completion before proceeding.

```
┌─────────────────────────────────────────────────┐
│ Phase 0: Assess Current State                   │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User approves action plan
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 0.25: Stray Doc Consolidation (if found)  │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User approves per-doc action
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 0.5: CLAUDE.md Refactor (if bloated)      │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User approves extraction plan
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 1: Project Profile (interactive)          │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User confirms all fields
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 2: Rules Setup                            │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User approves rule scaffolds
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 3: Rule Overrides                         │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User approves each override
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 4: Supporting Files                       │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────┐
│ Phase 5: Verify                                 │
└─────────────────────────────────────────────────┘
```

---

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Phase 0: Assess Current State

Detect what already exists. If `/audit` was run previously in the conversation,
consume its report to prioritize actions. If `/understand` was run, consume
`.understand/architecture-map.md` to pre-populate Profile fields and domain boundaries.

```bash
# What exists?
ls CLAUDE.md README.md CHANGELOG.md TODOS.md .gitignore .claudeignore 2>/dev/null
ls -d rules/ rules/business/ 2>/dev/null

# Detect stack
ls package.json requirements.txt Pipfile pyproject.toml go.mod Cargo.toml \
   *.csproj *.sln composer.json Gemfile 2>/dev/null

# Read existing CLAUDE.md if present
cat CLAUDE.md 2>/dev/null | head -100

# Architecture map from /understand (if run previously)
ls .understand/architecture-map.md 2>/dev/null

# Count source directories for complexity check
ls -d src/*/ app/*/ lib/*/ modules/*/ services/*/ packages/*/ 2>/dev/null | wc -l
```

### Complexity Check

If the repo has 5+ top-level source directories and no `.understand/architecture-map.md`
exists, suggest running `/understand` first:

> "This looks like a complex codebase with {N} source directories. Consider running
> `/understand` first to map the architecture — the findings will pre-populate
> Architecture, domain boundaries, and other Profile fields here."

This is a suggestion, not a gate. If the user wants to proceed, continue normally.

Present assessment in table format:

```
INIT ASSESSMENT ─────────────────────────────────────────────────
```

| Area | Check | Status | Action |
|------|-------|--------|--------|
| CLAUDE.md | File exists | {yes / missing} | {skip / create} |
| CLAUDE.md | Project Profile present | {complete / partial ({N}/9) / missing} | {skip / add fields / create} |
| CLAUDE.md | Under 200 lines | {yes / no ({N} lines)} | {skip / refactor: extract to rules/} |
| CLAUDE.md | No inlined rule/skill content | {yes / no} | {skip / extract to files, replace with paths} |
| Rules | `rules/` directory exists | {yes / missing} | {skip / create} |
| Rules | `rules/business/` exists | {yes / empty / missing} | {skip / scaffold} |
| Rules | All rules have `paths:` frontmatter | {yes / {N} missing} | {skip / add frontmatter} |
| Rules | Overrides present | {N found} | {skip / suggest} |
| Stray docs | Documents overlapping with hashb structure | {none / {N} found} | {skip / consolidate} |
| Routing | Skill Routing section present | {yes / missing} | {skip / add} |
| Context | `@` imports in CLAUDE.md | {N} | {ok ≤ 5 / refactor} |
| Context | Skills referenced by name only | {yes / inlined} | {skip / extract} |
| Support | CHANGELOG.md | {exists / missing} | {skip / create} |
| Support | .gitignore (hashb entries) | {present / missing} | {skip / add entries} |
| Support | .claudeignore | {exists / missing / incomplete} | {skip / create / add entries} |
| Support | .env.example | {exists / missing / n/a} | {skip / create from .env} |
| Support | Testing config detected | {vitest / jest / pytest / etc. / none} | — |
| Docs | README.md sections complete | {yes / {N} missing} | {skip / scaffold missing} |
| Docs | LICENSE file | {exists / missing} | {skip / suggest} |
| Docs | HASHB.md (getting started) | {exists / missing} | {skip / generate} |

```
  PLAN
  ─────────────────────────────────────────────────
  1.  {action — e.g., "Create CLAUDE.md with Project Profile"}
  2.  {action — e.g., "Extract inlined rules to rules/ with paths: frontmatter"}

  SKIP (already compliant)
  ─────────────────────────────────────────────────
  -  {item — e.g., "CHANGELOG.md exists and follows format"}

─────────────────────────────────────────────────────────────────
```

> **STOP.** User approves the action plan before any changes.

---

### Branch Guard

Before making any file changes, ensure you're on a feature branch:

```bash
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "master" ]; then
  echo "⚠ ON PROTECTED BRANCH — creating chore branch"
  git checkout -b "chore/hashb-init"
fi
```

> **Never modify repo structure directly on main/master.**

---

## Phase 0.25: Stray Document Consolidation

**Skip if:** No stray documents detected in Phase 0 assessment.

**Trigger if:** Phase 0 found documents outside hashb's expected locations that
overlap with content hashb manages (rules, CLAUDE.md sections, supporting files).

### Why This Comes Before CLAUDE.md Refactor

Stray documents may contain content that should be merged into CLAUDE.md, rule
files, or other hashb-managed locations. Consolidating them first prevents
duplication — Phase 0.5 (CLAUDE.md Refactor) then works with the complete picture.

### Process

Scan for stray docs, read each candidate, and classify as MERGE / MOVE / DELETE / KEEP.
Present each document one at a time for user approval. Execute approved actions
in order: MERGE, MOVE, DELETE.

See `references.md` for discovery scripts, classification rules, overlap detection
signals, per-document approval template, and execution steps.

> **STOP.** User approves each document individually.

---

## Phase 0.5: CLAUDE.md Refactor

**Skip if:** CLAUDE.md doesn't exist, or is under 200 lines with no inlined content.

**Trigger if:** CLAUDE.md is over 200 lines, OR contains inlined rule/skill content,
OR has large instruction blocks that belong in scoped rule files.

### Why This Comes First

A bloated CLAUDE.md degrades Claude's instruction adherence and wastes tokens on
every message. Fixing this before adding Profile, Routing, and rules prevents
the file from growing further. Extract first, then build on a lean foundation.

### Process

Read the full CLAUDE.md and classify every section as KEEP / EXTRACT / REMOVE.
Present an extraction plan with target rule files, paths, and projected line count.

See `references.md` for section classification guide (what to keep, extract, remove),
extraction plan template, and projected result template.

> **STOP.** User approves extraction plan before any changes.

### Execute

1. Create each new rule file with proper frontmatter (`id`, `description`, `paths:`)
2. Remove extracted content from CLAUDE.md
3. Add 1-line references in CLAUDE.md pointing to the new files
4. Verify CLAUDE.md is now under 200 lines

> **Preserve all user content.** Extraction moves content to a better location —
> it never deletes anything. Every extracted section must be findable in its new
> rule file.

---

## Phase 1: Project Profile (Interactive)

If CLAUDE.md is missing or has no Project Profile, walk through each field.

### Auto-Detection

For each field, detect what's possible from the codebase:

```bash
# Stack detection
cat package.json 2>/dev/null | head -50   # Node/JS/TS deps
cat requirements.txt 2>/dev/null          # Python deps
cat go.mod 2>/dev/null | head -20         # Go deps
ls *.csproj 2>/dev/null                   # .NET

# Platform detection
ls android/ ios/ 2>/dev/null              # Native mobile
cat package.json 2>/dev/null | grep -i "react-native\|expo"   # RN/Expo
cat pubspec.yaml 2>/dev/null | head -10   # Flutter
cat package.json 2>/dev/null | grep -i "electron\|tauri"      # Desktop
cat package.json 2>/dev/null | grep -i "commander\|yargs\|oclif\|ink" # CLI
ls src/app/ src/pages/ app/ pages/ templates/ 2>/dev/null     # Web (routes/pages)

# Distribution detection
cat package.json 2>/dev/null | grep -i "stripe\|paddle\|lemonsqueezy"  # SaaS billing
ls service-worker* sw.js manifest.json 2>/dev/null  # PWA / offline-first
ls Dockerfile docker-compose* 2>/dev/null           # Self-hosted candidate
ls .github/workflows/*publish* 2>/dev/null          # npm/package publishing
cat package.json 2>/dev/null | grep -i '"bin"'      # CLI distribution
```

### Per-Field Walkthrough

Auto-detect where possible, then confirm with the user. Fields that cannot be
reliably auto-detected (Architecture, Tenancy, Audience, Deploy) must always be
asked. Field 9 (MCP Servers) requires Context7 — walk through installation if
needed.

### Architecture Map Integration

If `.understand/architecture-map.md` exists, read it and use findings to
pre-populate suggestions for fields that normally require asking:

| Field | How architecture map helps |
|-------|--------------------------|
| Architecture | Use Module Boundaries + Data Flow to suggest (e.g., "modular monolith with 5 domains") |
| Tenancy | Check Conventions Detected for tenant scoping patterns |
| MCP Servers | If graphify was used in the map, recommend adding it as MCP server |

Present these as evidence-backed suggestions, not assumptions. User still confirms.

### Graphify MCP Recommendation

During the MCP Servers walkthrough, check if graphify is installed:

```bash
python -c "import graphify; print('installed')" 2>/dev/null || echo "not installed"
```

If installed but not in MCP Servers: recommend adding it. Provide the config snippet:

```json
{
  "mcpServers": {
    "graphify": {
      "command": "python",
      "args": ["-m", "graphify.serve", "/absolute/path/to/graphify-out/graph.json"]
    }
  }
}
```

> "Graphify is installed. Adding it as an MCP server enables graph-accelerated
> analysis in `/understand`, `/review`, and `/audit`."

See `references.md` for the full per-field walkthrough scripts and MCP server
suggestion table.

### Write Project Profile

**If CLAUDE.md exists:** Read the full file, find the Project Profile section
(or insertion point), update only that section. Preserve all other content.

**If CLAUDE.md is missing:** Create it with the Project Profile section.
Ask the user if they have other content to add.

> **STOP.** User confirms the Profile looks correct.

### Write Skill Routing

After the Project Profile, add a Skill Routing section so Claude invokes
the correct skill from natural language — without requiring explicit `/` commands.

**If Skill Routing section already exists:** Skip — don't overwrite.
**If missing:** Add after the Project Profile section.

See `references.md` for the Skill Routing template.

> This section is lightweight (~16 lines) and lives in CLAUDE.md so it loads
> every session. Claude reads these hints and routes to the skill automatically.

---

## Phase 2: Rules Setup

Based on the declared Profile, set up the rules directory structure.

### Determine What's Needed

| Profile Signal | Rules Action |
|----------------|-------------|
| Architecture mentions "modular" or domain boundaries | Create `rules/business/<domain>/` scaffolds |
| Architecture mentions "microservices" | Create per-service business rule scaffolds |
| Stack includes specific frameworks | Verify matching frontend/backend rules apply |
| No business rules exist | Suggest domain areas based on directory scan |

### Detect Domain Boundaries

```bash
# Scan for domain-like directories
ls -d src/*/  app/*/  lib/*/  modules/*/  services/*/ 2>/dev/null
```

### Create Business Rule Scaffolds

For each identified domain area, create a starter rule file with TODO markers.
See `references.md` for the business rule scaffold template.

Present the scaffolds:

```
RULES SCAFFOLDS:
  1. rules/business/{domain-a}/{domain-a}.md
     Paths: src/{domain-a}/**/*
     Status: NEW (TODO template)

  2. rules/business/{domain-b}/{domain-b}.md
     Paths: src/{domain-b}/**/*
     Status: NEW (TODO template)
```

> **STOP.** User approves rule scaffolds. They can add, remove, or rename domains.

---

## Phase 3: Rule Overrides

Based on the declared stack and codebase structure, suggest rule overrides
where the consumer's repo legitimately needs different behavior (e.g., test
helpers, generated code, internal admin tools, migrations, dev scripts).

For each suggested override, present paths, relaxations, and ask for confirmation.

> **STOP.** User approves each override individually.

---

## Phase 4: Supporting Files

Check and update supporting files. These are done without individual approval
unless the change is significant.

### .gitignore

Add hashb-relevant entries if missing:

```
# hashb audit/qa artifacts
.audit/
.qa-reports/
.retro/
```

### .claudeignore

Ensure a `.claudeignore` file exists to prevent Claude from reading irrelevant,
sensitive, or too-large files. Detect project-specific entries from `.gitignore`
and the codebase.

See `references.md` for the recommended `.claudeignore` template. Tailor to the
detected stack — don't add Python entries to a Node-only project.

### .env.example

If the project uses environment variables and `.env.example` doesn't exist,
create it with variable names from `.env` but placeholder values. Verify `.env`
is gitignored.

### CHANGELOG.md

If missing, create with the docs.md format:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]
```

### TODOS.md

If missing, create with the docs.md format:

```markdown
# TODOs

| Priority | Effort | Description | Status |
|----------|--------|-------------|--------|
```

### README.md Standards

Check README.md for minimum sections (Title, Setup, Usage, Contributing, License).
If missing or incomplete, scaffold missing sections — don't overwrite existing content.

### Getting Started Guide (HASHB.md)

**If HASHB.md exists:** Skip — don't overwrite.
**If missing:** Generate based on the Project Profile and detected stack.

See `references.md` for the HASHB.md template. Tailor the guide using the
Project Profile — only include relevant recipes, actual test commands, and
rules that exist in the repo.

### Guard Hooks

Scaffold Claude Code hooks for destructive command protection. These use
the `PreToolUse` hooks API — enforcement that cannot be bypassed, even with
`--dangerously-skip-permissions`.

**If `.claude/settings.json` already has hooks configured:** Merge, don't overwrite.
**If missing or no hooks section:** Create with guard hook configuration.

Create `.claude/hooks/guard-check.sh` and add hook configuration to
`.claude/settings.json`. See `references.md` for the hook template and
guard script content.

> Guard hooks are recommended for production repos. Frame as opt-in but
> strongly recommended during the init walkthrough.

Also scaffold the **post-compaction context hook** — a `SessionStart` hook
with `compact` matcher that re-injects the Active Work section from TODOS.md
after context compaction. This ensures recipe position and key decisions
survive compaction. See `references.md` for the hook template.

### CI/CD Pipeline

If no CI configuration is detected, scaffold a starter pipeline based on the
detected stack.

```bash
ls .github/workflows/*.yml .github/workflows/*.yaml \
   .gitlab-ci.yml Jenkinsfile .circleci/config.yml \
   .travis.yml bitbucket-pipelines.yml 2>/dev/null
```

**If CI already exists:** Skip — don't overwrite.
**If no CI detected:** Offer to scaffold based on the Project Profile Stack.
See `references.md` for CI templates per stack (GitHub Actions, GitLab CI).

---

## Phase 5: Verify

Run a quick check to confirm the setup is correct.

### Verification Checklist

| Area | Check | Status |
|------|-------|--------|
| Profile | CLAUDE.md exists with Project Profile | {pass / fail} |
| Profile | All Profile fields populated | {N}/9 |
| Profile | Context7 listed in MCP Servers | {pass / fail} |
| Profile | code-review-graph in MCP Servers (if 50+ source files) | {pass / skip / fail} |
| Routing | Skill Routing section present | {pass / fail} |
| Stray docs | No overlapping documents outside hashb structure | {pass / fail ({N} remaining)} |
| Context | CLAUDE.md under 200 lines | {pass / fail ({N} lines)} |
| Context | No inlined rule/skill content | {pass / fail} |
| Context | Skills referenced by name only | {pass / fail} |
| Rules | Directory structure valid | {pass / fail} |
| Rules | All rules have `paths:` frontmatter | {pass / fail ({N}/{total})} |
| Rules | Business rule frontmatter valid | {pass / fail / n/a} |
| Rules | Override targets resolve | {pass / fail / n/a} |
| Support | .gitignore has hashb entries | {pass / fail} |
| Support | .claudeignore exists with recommended entries | {pass / fail} |
| Support | .env.example exists (if .env detected) | {pass / fail / n/a} |
| Support | CHANGELOG.md exists | {pass / fail} |
| Docs | README.md has required sections | {pass / fail ({N}/5)} |
| Docs | LICENSE file exists | {pass / fail} |
| Docs | HASHB.md getting started guide | {pass / fail} |
| Hooks | `.claude/settings.json` has PreToolUse guard hook | {pass / skipped / fail} |
| Hooks | `.claude/hooks/guard-check.sh` exists and is executable | {pass / skipped / fail} |
| CI/CD | CI config exists | {pass / scaffolded / skipped} |

### Summary

```
✓ INIT COMPLETE ─────────────────────────────────────────────────

  CREATED
  ─────────────────────────────────────────────────
  -  {list of new files}

  UPDATED
  ─────────────────────────────────────────────────
  -  {list of modified files}

  SKIPPED (already compliant)
  ─────────────────────────────────────────────────
  -  {list}

  VERIFIED
  ─────────────────────────────────────────────────
  Profile fields           {N}/9 populated
  Rules structure          valid
  Business rule frontmatter valid
  Rule overrides           {N} created

  ➤ NEXT: /audit (see Next Step below)

─────────────────────────────────────────────────────────────────
```

### Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| First-time setup | `/audit` | Verify compliance, find remaining gaps |
| Re-running after audit findings | `/audit` (verification pass) | Confirm fixes landed |
| Repo is ready to build | `/eng` or `/spec` | Start the development workflow |

**Autonomous mode:** If running inside a recipe chain (e.g., Onboarding):
- After initial setup → auto-proceed to `/audit`
- After fixing audit findings → auto-re-run `/audit` for verification

---

## Idempotency & Rules

**Idempotent.** Every action checks before modifying. See `references.md` for
the full idempotency rules table.

- **Interactive.** Never assume — ask about fields that can't be auto-detected.
- **Preserve existing content.** When updating CLAUDE.md, preserve all non-Profile
  sections. Never delete user content.
- **Scaffolds are templates, not opinions.** Business rule scaffolds contain TODO
  markers, not opinionated content the consumer didn't write.
- **Consume /audit output.** If an audit report exists in conversation context,
  use its findings to prioritize actions and skip already-compliant items.
- **Minimal changes.** Only create what's needed for hashb compliance. Don't
  refactor the consumer's project structure.
- **Explain every file.** Before creating or modifying any file, state what it
  does and why it's needed.
- **Respect the consumer's choices.** If they decline an override or rule scaffold,
  accept it. Don't re-suggest in the same session.

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "We don't need a Project Profile — our stack is obvious" | Skills read the Profile to adapt behavior. Without it, every skill guesses at your stack, testing framework, and deploy target. 5 minutes of setup saves hours of misaligned output. |
| "Rules are overkill for a small project" | Rules auto-activate when matching files are read. They don't cost tokens when they don't match. Even small projects benefit from security and testing rules. |
| "We'll set up CLAUDE.md manually later" | Manual setup misses auto-detection, MCP server configuration, and rule scaffolding. `/init` in 10 minutes does what manual setup takes an hour to replicate. |
| "Our existing docs are fine — skip the stray doc check" | Stray docs that predate hashb create conflicting guidance. A quick scan finds overlaps that confuse both agents and humans. |
| "We don't need .claudeignore" | Without it, Claude reads node_modules, build output, lock files, and .env secrets. The context window fills with noise instead of signal. |

---
description: Bootstrap or update a repo for hashb compliance -- Project Profile, rules, business scaffolds. Idempotent.
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

## Presentation Rules

1. **Wizard flow** — present one phase at a time. Never dump everything at once.
2. **Discussion chunking** — when a follow-up response would be too dense to digest in one shot, present a numbered big-picture overview first, then discuss each point one at a time, waiting for user input between points. Use judgment: chunk whenever the response feels like a wall of text, not at a fixed threshold.
3. **Progress indicator** — every output starts with:

```
/init ═══════════════════════════════════════════════════════════

  ▸ Phase 0      Assess Current State       ~2 min
  ○ Phase 0.25   Stray Doc Consolidation   ~3 min  (if needed)
  ○ Phase 0.5    CLAUDE.md Refactor        ~3 min  (if needed)
  ○ Phase 1    Project Profile         ~5 min
  ○ Phase 2    Rules Setup             ~3 min
  ○ Phase 3    Rule Overrides          ~3 min
  ○ Phase 4    Supporting Files        ~2 min
  ○ Phase 5    Verify                  ~1 min

═════════════════════════════════════════════════════════════════
```

Update `▸` (current), `✓` (done), `○` (pending) as phases progress.
Completed phases show a status note on the right (e.g., `✓ 6/6 fields`, `✓ 3 rules`).

---

## Phase 0: Assess Current State

Detect what already exists. If `/audit` was run previously in the conversation,
consume its report to prioritize actions.

```bash
# What exists?
ls CLAUDE.md README.md CHANGELOG.md TODOS.md .gitignore .claudeignore 2>/dev/null
ls -d rules/ rules/business/ 2>/dev/null

# Detect stack
ls package.json requirements.txt Pipfile pyproject.toml go.mod Cargo.toml \
   *.csproj *.sln composer.json Gemfile 2>/dev/null

# Read existing CLAUDE.md if present
cat CLAUDE.md 2>/dev/null | head -100
```

Present assessment in table format:

```
INIT ASSESSMENT ─────────────────────────────────────────────────
```

| Area | Check | Status | Action |
|------|-------|--------|--------|
| CLAUDE.md | File exists | {yes / missing} | {skip / create} |
| CLAUDE.md | Project Profile present | {complete / partial / missing} | {skip / add fields / create} |
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
  3.  {action — e.g., "Add paths: to 3 rule files missing it"}

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

### Discovery

Scan for documents that may overlap with hashb structure:

```bash
# Common documentation files that predate hashb adoption
ls CONTRIBUTING.md CODING_STANDARDS.md STYLE_GUIDE.md ARCHITECTURE.md \
   CONVENTIONS.md GUIDELINES.md DEVELOPMENT.md WORKFLOW.md STANDARDS.md \
   CODE_OF_CONDUCT.md TESTING.md SECURITY.md 2>/dev/null

# Docs directories with potentially overlapping content
ls -d docs/ doc/ documentation/ wiki/ guides/ 2>/dev/null

# Markdown files outside expected locations
find . -maxdepth 3 -name "*.md" \
  -not -path "./rules/*" \
  -not -path "./node_modules/*" \
  -not -path "./.git/*" \
  -not -path "./.retro/*" \
  -not -path "./.qa-reports/*" \
  -not -path "./.audit/*" \
  -not -name "README.md" \
  -not -name "CLAUDE.md" \
  -not -name "CHANGELOG.md" \
  -not -name "TODOS.md" \
  -not -name "HASHB.md" \
  -not -name "LICENSE.md" \
  2>/dev/null
```

### Analysis

Read each candidate document and classify it:

```
STRAY DOCUMENT SCAN ─────────────────────────────────────────────
```

| # | Document | Lines | Overlaps With | Recommendation |
|---|----------|-------|---------------|----------------|
| 1 | {file} | {N} | {target} | {MERGE / MOVE / DELETE / KEEP} |

**Classification rules:**

| Action | When | What happens |
|--------|------|-------------|
| **MERGE** | Content overlaps with an existing or expected hashb file | Extract unique content into the hashb target, delete the original |
| **MOVE** | Content is useful but lives in the wrong location | Rename/move to `rules/{name}.md` with proper `paths:` frontmatter |
| **DELETE** | Content is fully duplicated or stale | Delete after confirmation |
| **KEEP** | Content serves a purpose hashb doesn't cover | No action — note why |

**Overlap detection — read each file and match against:**

| Content signals | Likely hashb target |
|----------------|-------------------|
| Coding standards, style guide, conventions | `rules/conventions.md` |
| Security guidelines, OWASP references | `rules/security.md` |
| Testing guidelines, test patterns | `rules/testing.md` |
| Architecture docs, system design | `rules/architecture.md` or `rules/business/` |
| Git workflow, branching strategy | `rules/git.md` |
| API specs, endpoint documentation | `rules/api.md` or `rules/business/` |
| Development setup, onboarding | `HASHB.md` or `README.md` |
| Contributing: process parts | `CONTRIBUTING.md` (KEEP) |
| Contributing: code standard parts | `rules/conventions.md` (split — MERGE standards, KEEP process) |

### Per-Document Approval

Present each stray document one at a time with full context:

```
STRAY DOCUMENT 1 of {N}
─────────────────────────────────────────────────
  File:        {path}
  Lines:       {N}
  Summary:     {1-2 sentence content summary}
  Overlaps:    {hashb target file(s)}

  RECOMMENDATION: {MERGE / MOVE / DELETE / KEEP}

  {If MERGE}
  Merge plan:
    - Lines {X-Y}: unique content → append to {target}
    - Lines {A-B}: duplicated by {target} → discard
    - Delete {file} after merge

  {If MOVE}
  Move plan:
    - Rename to: rules/{new-name}.md
    - Add frontmatter: paths: ["{relevant glob}"]
    - Add reference in CLAUDE.md (if cross-cutting)

  {If DELETE}
  Reason: {why it's safe to delete — what covers this content}

  {If KEEP}
  Reason: {why this file should stay — what purpose it serves outside hashb}

  Action? [merge / move / delete / keep / skip]
─────────────────────────────────────────────────
```

> **STOP.** User approves each document individually. Accept the user's choice
> even if it differs from the recommendation.

### Execute Approved Actions

Process approved actions in order: MERGE → MOVE → DELETE.

**For MERGE actions:**
1. Read the target file (create if it doesn't exist with proper frontmatter)
2. Append unique content from the stray document into the target
3. Preserve the target's existing structure and frontmatter
4. Delete the original stray document
5. If the stray was referenced elsewhere (README, CLAUDE.md), update the reference

**For MOVE actions:**
1. Create the new file at the target path with proper hashb frontmatter
2. Copy all content from the stray document
3. Delete the original
4. If referenced elsewhere, update the reference

**For DELETE actions:**
1. Delete the file
2. Remove any references to it in other files

**After all actions, report:**

```
CONSOLIDATION COMPLETE
─────────────────────────────────────────────────
  Merged:   {N} documents → {targets}
  Moved:    {N} documents → rules/
  Deleted:  {N} documents
  Kept:     {N} documents (no action)
  Skipped:  {N} documents (user deferred)
─────────────────────────────────────────────────
```

---

## Phase 0.5: CLAUDE.md Refactor

**Skip if:** CLAUDE.md doesn't exist, or is under 200 lines with no inlined content.

**Trigger if:** CLAUDE.md is over 200 lines, OR contains inlined rule/skill content,
OR has large instruction blocks that belong in scoped rule files.

### Why This Comes First

A bloated CLAUDE.md degrades Claude's instruction adherence and wastes tokens on
every message. Fixing this before adding Profile, Routing, and rules prevents
the file from growing further. Extract first, then build on a lean foundation.

### Analysis

Read the full CLAUDE.md and classify every section:

```
CLAUDE.MD ANALYSIS ({N} lines) ─────────────────────────────────
```

| # | Section | Lines | Verdict | Action |
|---|---------|-------|---------|--------|
| 1 | {section} | {N} | KEEP | Essential every session |
| 2 | {section} | {N} | EXTRACT | Move to rules/{file}.md with paths: |
| 3 | {section} | {N} | REMOVE | Duplicate of {source} |

**What to KEEP in CLAUDE.md (loads every session):**
- Project name + 1-line description
- Project Profile (~8 lines)
- Skill Routing (~16 lines)
- Short references ("See rules/X.md for Y")
- Critical invariants that apply to ALL files (max 5 lines)

**What to EXTRACT to rule files (loads on-demand via `paths:`):**
- Coding standards / conventions → `rules/conventions.md` with `paths:` matching source files
- Architecture guidelines → `rules/architecture.md` with `paths:` matching relevant dirs
- API conventions → `rules/api.md` with `paths:` matching API source files
- Testing instructions → already covered by `rules/testing.md`
- Security instructions → already covered by `rules/security.md`
- Workflow/process docs → reference `recipes.md` or `HASHB.md`
- Long instruction blocks → split by concern into scoped rule files

**What to REMOVE:**
- Content that duplicates an existing rule file
- Pasted skill documentation (skills load on invocation)
- Copy of README content
- Stale instructions for removed features

### Extraction Plan

For each EXTRACT section, present:

```
EXTRACTION {N}: "{section name}" ({lines} lines)
  ─────────────────────────────────────────────────
  Target:    rules/{suggested-file}.md
  paths:     ["{glob matching relevant source files}"]
  Replaces:  {lines removed from CLAUDE.md}
  Reference: Add "See rules/{file}.md" to CLAUDE.md (1 line)
```

### Projected Result

```
BEFORE:  {N} lines (CLAUDE.md)
AFTER:   {projected} lines (CLAUDE.md) + {N} new rule files

New rule files:
  rules/{file-1}.md  ({N} lines, paths: {patterns})
  rules/{file-2}.md  ({N} lines, paths: {patterns})
```

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
```

### Per-Field Walkthrough

Present detected values and ask the user to confirm or correct.

**Field 1: Stack**

```
DETECTED STACK:
  package.json -> Next.js 15, TypeScript, Prisma
  Is this correct? Anything to add or change?
```

**Field 2: Architecture** (cannot auto-detect -- always ask)

```
ARCHITECTURE:
  What pattern does this project follow?
  Examples: Modular monolith, microservices, monorepo, layered MVC
```

**Field 3: Tenancy** (cannot auto-detect -- always ask)

```
TENANCY:
  Is this single-tenant or multi-tenant?
  If multi-tenant, how is tenant isolation handled?
  Examples: Single-tenant, multi-tenant (tenant_id scoping), multi-tenant (schema per tenant)
```

**Field 4: Testing**

```
DETECTED TESTING:
  vitest.config.ts found -> Vitest
  playwright.config.ts found -> Playwright
  Is this correct?
```

**Field 5: Deploy** (cannot auto-detect from source alone -- always ask)

```
DEPLOY:
  How is this project deployed?
  Examples: Vercel, AWS ECS, Docker Compose, Kubernetes, Fly.io
```

**Field 6: MCP Servers** (guided setup — Context7 required)

Context7 is required and always included. Walk the user through installation
if not already configured, then suggest additional plugins based on stack.

```
REQUIRED MCP SERVER:
  Context7 (library docs) — live, up-to-date library documentation.
  Install: npx -y @anthropic-ai/create-mcp --name context7 -- npx -y @upstash/context7-mcp@latest
  Already installed? Confirm and continue.
```

Based on the detected stack, suggest additional MCP servers:

| Stack signal | Suggested MCP Server | Purpose |
|---|---|---|
| React / Next.js / frontend | Vercel AI SDK / React best practices plugin | React patterns, performance, accessibility |
| Any codebase | Code review plugin | Automated review patterns |

```
ADDITIONAL MCP SERVERS:
  Based on your stack ({detected}), these plugins are recommended:
  1. {plugin} — {purpose}. Install: {command}
  2. {plugin} — {purpose}. Install: {command}
  Add any others? Enter "done" when finished.
```

> **Context7 is always included.** When writing the MCP Servers field, always
> start with `Context7 (library docs)` regardless of user input. Append any
> additional servers the user declares.

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

**If missing:** Add after the Project Profile section:

```markdown
## Skill Routing
When user intent matches a skill, invoke it instead of acting freestyle:
- Requirements, specs, "what should we build" → `/hashb:spec`
- UI design, wireframes, components → `/hashb:design`
- Library/API evaluation, tech research → `/hashb:research`
- Architecture, implementation plan, scope → `/hashb:eng`
- TDD execution, "write the tests" → `/hashb:tdd`
- Code review, "review my code" → `/hashb:review`
- Bug, broken, error, fix → `/hashb:fix`
- Test, QA, verify → `/hashb:qa`
- Retrospective, "what went well" → `/hashb:retro`
- Ship, PR, merge, release → `/hashb:ship`
- Parallel work, decompose → `/hashb:swarm`
- Doc cleanup, stale docs, sync docs → `/hashb:docs`
- Compliance check, audit → `/hashb:audit`
- Setup, bootstrap, init → `/hashb:init`
```

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

For each identified domain area, create a starter rule file:

```yaml
---
id: business-{domain}
description: "TODO: Business rules for {domain} -- add domain-specific constraints"
paths:
  - "src/{domain}/**/*"
---

# {Domain} Business Rules

> TODO: Add domain-specific rules for this bounded context.
> These rules activate when Claude reads files matching the paths above.

## Constraints

> TODO: What invariants must this domain preserve?
> Examples:
> - "Orders must always have a valid customer reference"
> - "Prices are stored in cents, never floating point"

## Patterns

> TODO: What patterns should code in this domain follow?
> Examples:
> - "All mutations go through domain services, never direct DB access"
> - "Events published for every state transition"
```

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
where the consumer's repo legitimately needs different behavior.

### Override Detection

| Signal | Suggested Override | Reason |
|--------|-------------------|--------|
| `**/test-utils/**`, `**/fixtures/**` | Override `testing` | Test helpers don't need full test discipline |
| Generated code (Prisma client, protobuf, etc.) | Override `security` | Auto-generated, not human-written |
| Internal admin tools (`**/admin/**`) | Override `security` | Internal-only, behind VPN |
| Migration files (`**/migrations/**`) | Override `git` | Different commit conventions for migrations |
| Scripts/tooling (`**/scripts/**`) | Override `security` | Dev tooling, not production code |

### Per-Override Approval

For each suggested override, present:

```
SUGGESTED OVERRIDE:
  Paths:     src/admin/**/*.ts
  Overrides: security
  Reason:    Admin tools are internal-only, behind VPN

  Relaxations:
  - CSRF tokens not required (internal tool)
  - Rate limiting relaxed (low traffic, trusted users)

  All other security rules still apply.

  Create this override? [y/n]
```

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

Ensure a `.claudeignore` file exists to prevent Claude from reading files that
are irrelevant, sensitive, or too large. This improves context efficiency and
avoids accidental exposure of secrets.

**If `.claudeignore` exists:** Verify it includes recommended entries. Add missing ones.
**If `.claudeignore` is missing:** Create with recommended defaults.

Detect project-specific entries from `.gitignore` and the codebase:

```bash
# Check what exists
cat .claudeignore 2>/dev/null
cat .gitignore 2>/dev/null

# Detect large/generated directories
ls -d node_modules/ dist/ build/ .next/ out/ coverage/ \
  vendor/ __pycache__/ .venv/ venv/ bin/ obj/ 2>/dev/null

# Detect lock files and generated code
ls package-lock.json yarn.lock pnpm-lock.yaml \
  Pipfile.lock poetry.lock go.sum Cargo.lock 2>/dev/null
```

**Recommended `.claudeignore` template:**

```
# Dependencies
node_modules/
vendor/
.venv/
venv/
__pycache__/

# Build output
dist/
build/
.next/
out/
bin/
obj/
coverage/

# Lock files (large, not useful for Claude)
package-lock.json
yarn.lock
pnpm-lock.yaml
Pipfile.lock
poetry.lock
go.sum
Cargo.lock

# Environment & secrets
.env
.env.*
!.env.example

# Large/binary assets
*.min.js
*.min.css
*.map
*.wasm
*.pb.go
*.generated.*

# hashb artifacts (read via skills, not directly)
.audit/
.qa-reports/
.retro/
```

> **Tailor the template.** Only include entries relevant to the detected stack.
> Don't add Python entries to a Node-only project.

### .env.example

If the project uses environment variables (detected via `.env`, `process.env`,
`os.environ`, `Environment.GetEnvironmentVariable`, etc.):

**If `.env.example` exists:** Skip.
**If `.env` exists but `.env.example` doesn't:** Create `.env.example` with
variable names from `.env` but placeholder values. Verify `.env` is gitignored.

```
# Copy this to .env and fill in values
DATABASE_URL=
API_KEY=
```

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

Check README.md for minimum sections. If missing or incomplete, scaffold
the missing sections — don't overwrite existing content.

**Required sections for any project:**

| Section | Purpose | Auto-detect? |
|---------|---------|-------------|
| Title + description | What is this project | From package.json / CLAUDE.md |
| Setup / Installation | How to get running locally | From detected stack |
| Usage / Running | How to start, test, build | From scripts in package.json / Makefile |
| Contributing | How to contribute | Template |
| License | Legal | From LICENSE file if present |

**If README.md is missing:** Create with all sections scaffolded.
**If README.md exists but sections are missing:** Suggest additions — present
each missing section for user approval. Never overwrite existing content.

```
README ASSESSMENT:
  ✓ Title + description     present
  ✗ Setup / Installation    missing — scaffold from detected stack?
  ✗ Usage / Running         missing — scaffold from package.json scripts?
  ✓ Contributing            present
  ✗ License                 missing — add section? (LICENSE file found: MIT)
```

### Getting Started Guide (HASHB.md)

Generate a project-specific quick-start guide for the team. This file tells
every developer in the repo what hashb skills are available and how to use them.

**If HASHB.md exists:** Skip — don't overwrite.
**If missing:** Generate based on the Project Profile and detected stack.

```markdown
# Getting Started with hashb

> AI-assisted development workflow for {project name}.
> This guide was generated by `/hashb:init`. Edit freely.

## Available Skills

| Command | What it does |
|---------|-------------|
| `/hashb:spec` | Define requirements and acceptance criteria |
| `/hashb:design` | UI/UX design (wireframes, components, accessibility) |
| `/hashb:research` | Evaluate libraries, APIs, and tech choices |
| `/hashb:eng` | Architecture decisions and implementation planning |
| `/hashb:tdd` | Test-driven implementation (RED → GREEN → REFACTOR) |
| `/hashb:review` | Peer code review with confidence scoring |
| `/hashb:fix` | Debug with root cause analysis |
| `/hashb:qa` | QA testing (reports only, never fixes) |
| `/hashb:ship` | Ship — test, review, version, changelog, PR |
| `/hashb:docs` | Document hygiene — clean stale, duplicate, scattered docs |
| `/hashb:retro` | Post-ship retrospective |
| `/hashb:swarm` | Parallel work decomposition |
| `/hashb:audit` | Compliance scan |

## Recommended Workflows

{Include only the 2-3 recipes most relevant to this project's Profile.}

### Building a Feature
```
/hashb:spec → /hashb:eng → /hashb:tdd → /hashb:review → /hashb:qa → /hashb:ship
```

### Fixing a Bug
```
/hashb:fix → /hashb:review → /hashb:qa → /hashb:ship
```

### Periodic Health Check
```
/hashb:audit → /hashb:init (if needed) → /hashb:audit (verify)
```

## Project-Specific Notes

- **Stack:** {from Profile}
- **Architecture:** {from Profile}
- **Testing:** {from Profile} — run with `{detected test command}`
- **MCP Servers:** {from Profile}

## Rules Active in This Repo

{List the rules that exist in rules/, with their paths: patterns.}

| Rule | Activates on |
|------|-------------|
| security | All source files |
| testing | Test files |
| {business-domain} | {paths from frontmatter} |
```

> **Tailor the guide.** Use the Project Profile to customize:
> - Only include recipes relevant to the stack (skip frontend recipes for backend-only)
> - Include the actual test command from package.json / Makefile
> - List only the rules that exist in this repo, not all possible rules

---

## Phase 5: Verify

Run a quick check to confirm the setup is correct.

### Verification Checklist

| Area | Check | Status |
|------|-------|--------|
| Profile | CLAUDE.md exists with Project Profile | {pass / fail} |
| Profile | All Profile fields populated | {N}/6 |
| Profile | Context7 listed in MCP Servers | {pass / fail} |
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
  Profile fields           {N}/6 populated
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

## Idempotency Rules

Every action checks before modifying:

| Action | Before creating | If exists |
|--------|-----------------|-----------|
| Stray documents | Scan for overlapping docs | Skip -- already consolidated |
| CLAUDE.md | Check if file exists | Update Profile and Routing sections only, preserve rest |
| Business rule scaffold | Check if rule file exists | Skip -- don't overwrite user's rules |
| Rule override | Check if override exists | Skip -- don't duplicate |
| .gitignore entries | Check if entries present | Add only missing entries |
| .claudeignore | Check if file exists and has entries | Add only missing entries |
| .env.example | Check if file exists | Skip -- don't overwrite |
| CHANGELOG.md | Check if file exists | Skip -- don't overwrite |
| TODOS.md | Check if file exists | Skip -- don't overwrite |
| README.md sections | Check which sections exist | Add only missing sections, preserve existing |
| HASHB.md | Check if file exists | Skip -- don't overwrite |

**Running `/init` after `/init` should produce:**
```
Already compliant. {N} items verified, 0 changes needed.
```

---

## Rules

- **Idempotent.** Running twice produces the same result. Never duplicate content.
- **Interactive.** Never assume -- ask about fields that can't be auto-detected.
  The user knows their stack better than dependency file analysis.
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

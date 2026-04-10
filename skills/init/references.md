# Init Skill — Reference Material

> Detailed templates, walkthroughs, classification tables, and code blocks
> used by each phase of the init skill. See `SKILL.md` for the orchestration
> skeleton and execution flow.

---

## Phase 0.25: Stray Doc Consolidation — Details

### Discovery Scripts

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

### Classification Rules

| Action | When | What happens |
|--------|------|-------------|
| **MERGE** | Content overlaps with an existing or expected hashb file | Extract unique content into the hashb target, delete the original |
| **MOVE** | Content is useful but lives in the wrong location | Rename/move to `rules/{name}.md` with proper `paths:` frontmatter |
| **DELETE** | Content is fully duplicated or stale | Delete after confirmation |
| **KEEP** | Content serves a purpose hashb doesn't cover | No action — note why |

### Overlap Detection Signals

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

### Per-Document Approval Template

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

## Phase 0.5: CLAUDE.md Refactor — Details

### Section Classification Guide

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

### Extraction Plan Template

For each EXTRACT section, present:

```
EXTRACTION {N}: "{section name}" ({lines} lines)
  ─────────────────────────────────────────────────
  Target:    rules/{suggested-file}.md
  paths:     ["{glob matching relevant source files}"]
  Replaces:  {lines removed from CLAUDE.md}
  Reference: Add "See rules/{file}.md" to CLAUDE.md (1 line)
```

### Projected Result Template

```
BEFORE:  {N} lines (CLAUDE.md)
AFTER:   {projected} lines (CLAUDE.md) + {N} new rule files

New rule files:
  rules/{file-1}.md  ({N} lines, paths: {patterns})
  rules/{file-2}.md  ({N} lines, paths: {patterns})
```

---

## Phase 1: Per-Field Walkthrough Scripts

Present detected values and ask the user to confirm or correct.

**Field 1: Stack**

```
DETECTED STACK:
  package.json -> Next.js 15, TypeScript, Prisma
  Is this correct? Anything to add or change?
```

**Field 2: Platform** (auto-detect from codebase, confirm with user)

```
DETECTED PLATFORM:
  {detection source} -> {detected platform}
  Is this correct? Anything to add or change?
```

If no signals detected:

```
PLATFORM:
  What kind of app is this?
  Examples: Web, Mobile (iOS), Mobile (Android), Mobile (React Native),
            Desktop (Electron), CLI, API-only, Library/SDK
  Multiple platforms? List all (e.g., "Web + Mobile (React Native)")
```

Auto-detection signals:

| Signal | Platform |
|--------|----------|
| `android/`, `ios/`, `*.swift`, `*.kt`, `*.java` (with Android imports) | Mobile (native) |
| `react-native` or `expo` in package.json | Mobile (React Native) |
| `pubspec.yaml` with Flutter | Mobile (Flutter) |
| `.xcodeproj`, `Podfile` | Mobile (iOS) |
| `electron` or `tauri` in package.json | Desktop |
| `commander`, `yargs`, `oclif`, `ink`, `click`, `cobra`, `clap` in deps | CLI |
| Web framework detected (Next.js, Express, Django, Rails, etc.) with routes/pages | Web |
| API framework with no frontend/pages directory | API-only |
| `"main"` field in package.json, no `"bin"`, no framework | Library/SDK |

**Field 3: Distribution** (auto-detect where possible, confirm with user)

```
DETECTED DISTRIBUTION:
  {detection source} -> {detected distribution model}
  Is this correct? Anything to add or change?
```

If no signals detected:

```
DISTRIBUTION:
  How will users get/access this?
  Examples: SaaS (hosted), self-hosted, offline-first, app store,
            npm/PyPI package, browser extension, internal deploy
```

Auto-detection signals:

| Signal | Distribution |
|--------|-------------|
| Billing/subscription deps (Stripe, Paddle, etc.) | SaaS |
| Service worker, offline manifest, IndexedDB usage | Offline-first / PWA |
| Dockerfile + no billing deps, self-host docs | Self-hosted |
| App store configs (Fastlane, app.json with expo) | App store |
| `"bin"` in package.json, CLI framework | npm/PyPI package |
| `manifest.json` with `"manifest_version"` | Browser extension |
| No public deploy config, internal tool signals | Internal deploy |

**Field 4: Audience** (cannot auto-detect — always ask)

```
AUDIENCE:
  Who uses this?
  Examples: B2B (businesses), B2C (consumers), internal tool,
            developer tool, marketplace (multi-sided)
  This helps skills tailor review checks and spec questions.
```

**Field 5: Architecture** (cannot auto-detect — always ask)

```
ARCHITECTURE:
  What pattern does this project follow?
  Examples: Modular monolith, microservices, monorepo, layered MVC
```

**Field 6: Tenancy** (cannot auto-detect — always ask)

```
TENANCY:
  Is this single-tenant or multi-tenant?
  If multi-tenant, how is tenant isolation handled?
  Examples: Single-tenant, multi-tenant (tenant_id scoping), multi-tenant (schema per tenant)
```

**Field 7: Testing**

```
DETECTED TESTING:
  vitest.config.ts found -> Vitest
  playwright.config.ts found -> Playwright
  Is this correct?
```

**Field 8: Deploy** (cannot auto-detect from source alone — always ask)

```
DEPLOY:
  How is this project deployed?
  Examples: Vercel, AWS ECS, Docker Compose, Kubernetes, Fly.io
```

**Field 9: MCP Servers** (guided setup — Context7 required)

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
| Any codebase with 50+ source files | code-review-graph (e.g., [graphify](https://github.com/safishamsi/graphify)) | Structural code intelligence: dependency graphs, module boundaries, blast radius, call chains |
| Any codebase | Code review plugin | Automated review patterns |

```
RECOMMENDED MCP SERVER:
  code-review-graph (e.g., graphify) — structural code intelligence for
  reviews, debugging, and architecture validation.
  Install: pip install graphifyy
  MCP server: graphify --mcp (configure in .claude/settings.json)
  Already installed? Confirm and continue.
  Skip? Optional — skills work without it but gain semantic code analysis when available.
```

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

---

## Phase 1: Skill Routing Template

When adding the Skill Routing section to CLAUDE.md, use this template:

```markdown
## Skill Routing
When user intent matches a skill, invoke it instead of acting freestyle:
- Requirements, specs, "what should we build" → `/hashb:spec`
- Break down large feature, task breakdown, "too big for one session" → `/hashb:spec decompose`
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

---

## Phase 2: Business Rule Scaffold Template

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

---

## Phase 4: .claudeignore Recommended Template

Tailor to the detected stack. Only include entries relevant to the project.

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

---

## Phase 4: HASHB.md Getting Started Template

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

## Phase 3: Override Detection Signals

| Signal | Suggested Override | Reason |
|--------|-------------------|--------|
| `**/test-utils/**`, `**/fixtures/**` | Override `testing` | Test helpers don't need full test discipline |
| Generated code (Prisma client, protobuf, etc.) | Override `security` | Auto-generated, not human-written |
| Internal admin tools (`**/admin/**`) | Override `security` | Internal-only, behind VPN |
| Migration files (`**/migrations/**`) | Override `git` | Different commit conventions for migrations |
| Scripts/tooling (`**/scripts/**`) | Override `security` | Dev tooling, not production code |

---

## Idempotency Rules

Every action checks before modifying:

| Action | Before creating | If exists |
|--------|-----------------|-----------|
| Stray documents | Scan for overlapping docs | Skip — already consolidated |
| CLAUDE.md | Check if file exists | Update Profile and Routing sections only, preserve rest |
| Business rule scaffold | Check if rule file exists | Skip — don't overwrite user's rules |
| Rule override | Check if override exists | Skip — don't duplicate |
| .gitignore entries | Check if entries present | Add only missing entries |
| .claudeignore | Check if file exists and has entries | Add only missing entries |
| .env.example | Check if file exists | Skip — don't overwrite |
| CHANGELOG.md | Check if file exists | Skip — don't overwrite |
| TODOS.md | Check if file exists | Skip — don't overwrite |
| README.md sections | Check which sections exist | Add only missing sections, preserve existing |
| HASHB.md | Check if file exists | Skip — don't overwrite |

**Running `/init` after `/init` should produce:**
```
Already compliant. {N} items verified, 0 changes needed.
```

---

## Phase 4: Guard Hook Templates

### .claude/settings.json hook configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/guard-check.sh"
          }
        ]
      }
    ]
  }
}
```

> If `.claude/settings.json` already exists with other settings, merge the
> `hooks` key into the existing file. Do not overwrite other settings.

### .claude/hooks/guard-check.sh

```bash
#!/bin/bash
# Guard hook — blocks destructive commands before execution.
# Installed by /hashb:init. Edit patterns to match your project's needs.
#
# Exit codes:
#   0 = allow (command is safe)
#   2 = block (destructive pattern detected)

INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command')

# Destructive patterns — add project-specific patterns below
PATTERNS='rm -rf|DROP TABLE|DROP DATABASE|TRUNCATE|git push.*--force|git push.*-f|git reset --hard|kubectl delete|docker system prune'

# Safe exceptions — build artifacts and caches
SAFE_TARGETS='node_modules|\.next|dist|__pycache__|\.cache|build|\.turbo|coverage'

# Check for destructive patterns
if echo "$CMD" | grep -qE "$PATTERNS"; then
  # Check if target is a safe exception
  if echo "$CMD" | grep -qE "$SAFE_TARGETS"; then
    exit 0
  fi
  echo "BLOCKED: Destructive command detected: $CMD" >&2
  exit 2
fi

exit 0
```

> Make the script executable: `chmod +x .claude/hooks/guard-check.sh`
> On Windows, the script runs via Git Bash (Claude Code's default shell).

---

## Phase 4: CI Template Scaffolding

### GitHub Actions — Node/TypeScript

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm test --if-present
      - run: npm run build --if-present
```

### GitHub Actions — Python

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: ruff check . || true
      - run: pytest
```

### GitHub Actions — .NET

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 8.0.x
      - run: dotnet restore
      - run: dotnet build --no-restore
      - run: dotnet format --verify-no-changes --no-restore
      - run: dotnet test --no-build
```

### GitHub Actions — Go

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: stable
      - run: go build ./...
      - run: golangci-lint run || true
      - run: go test ./...
```

### GitHub Actions — Rust

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo build
      - run: cargo clippy -- -D warnings
      - run: cargo test
```

> **Template selection:** Match the template to the detected Stack from
> the Project Profile. If no match, use the Node template as default or
> ask the user to customize.

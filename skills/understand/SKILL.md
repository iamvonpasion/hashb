---
name: understand
description: >
  Systematic codebase mapping — architecture, module boundaries, data flows, conventions.
  Produces persistent .understand/architecture-map.md. Use on brownfield projects before
  /init, or before /eng when the codebase is unfamiliar.
argument-hint: "[scope or subsystem path]"
---

# Understand

Map an existing codebase systematically. Produce a persistent architecture artifact
that survives across sessions and feeds downstream skills.

**Writes `.understand/architecture-map.md` only. No code changes.**

---

## When to Use

| Signal | Why understand? |
|--------|----------------|
| "I'm new to this repo" | Need a structured map before making changes |
| "Map the architecture" | Systematic analysis, not ad-hoc exploration |
| Brownfield project adopting hashb | Understand before scaffolding rules |
| Before major refactoring | Know the current architecture so you don't break assumptions |
| Onboarding a team member | Produce a referenceable architecture document |
| Before `/eng` on unfamiliar code | Know module boundaries and coupling before planning |

**Don't use when:** You have a specific question (use `/explore`), you already know the codebase (use `/eng` directly), you need to check compliance (use `/audit`), or you have a bug (use `/fix`).

---

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Execution Flow (MANDATORY)

```
Phase 0: Orientation
     |
Phase 1: Map Structure
     |
Phase 2: Trace Flows
     |
Phase 3: Synthesize ── STOP ── User validates architecture map
     |
Phase 4: Handoff
```

---

## Phase 0: Orientation

Detect what exists. No assumptions — only report what is found.

```bash
# Existing understanding artifacts
ls -d .understand/ 2>/dev/null
ls .understand/architecture-map.md 2>/dev/null

# Project Profile (if hashb already adopted)
cat CLAUDE.md 2>/dev/null | head -60

# Stack detection from dependency manifests
ls package.json requirements.txt Pipfile pyproject.toml go.mod Cargo.toml \
   *.csproj *.sln composer.json Gemfile build.gradle pom.xml 2>/dev/null

# Module/domain structure
ls -d src/*/ app/*/ lib/*/ modules/*/ services/*/ packages/*/ cmd/*/ internal/*/ 2>/dev/null

# Entry points
ls -d src/app/ src/pages/ app/ pages/ routes/ api/ bin/ cmd/ \
   src/main.* main.* index.* 2>/dev/null

# Graphify MCP availability
python -c "import graphify; print('graphify installed')" 2>/dev/null || echo "graphify not installed"
```

### Graphify MCP Detection

Check if the graphify MCP server is configured and reachable:

1. **MCP server configured:** Check for `mcpServers.graphify` in Claude Desktop config
   or `.claude/settings.json`. If configured, MCP tools are available:
   `query_graph`, `get_node`, `get_neighbors`, `get_community`, `god_nodes`,
   `graph_stats`, `shortest_path`.
2. **Graphify installed but no MCP server:** Suggest running
   `/graphify . --mcp` first, then re-running `/understand` for graph-accelerated analysis.
3. **Graphify not installed:** Proceed with standard mode. No hard dependency.

### Scope Card

Present orientation findings:

```
UNDERSTAND ──────────────────────────────────────────────────────
  Repo            {name}
  Stack           {detected from dependency manifests — NOT guessed}
  Modules         {N directories detected}
  Entry points    {N detected}
  Source files    {approximate count}
  Existing map    {.understand/ exists — update mode | fresh}
  Graphify MCP    {available — graph-accelerated | not configured | not installed}
  Focus           {full system | user-specified subsystem}

  MODE: {standard | graph-accelerated}
────────────────────────────────────────────────────────────────
```

No gate. Proceed to Phase 1.

---

## Phase 1: Map Structure

Build the Module Boundaries table by analyzing each module/domain directory.

### Standard Mode

For each detected module directory:

1. **Read 2-3 representative files** — entry point, main export, and one internal file
2. **Identify responsibility** — what this module does (one sentence)
3. **Identify public interface** — exported functions, classes, routes, or services
4. **Detect conventions** — naming patterns, architectural style, error handling, DI

Budget: max 5-10 files per module. Sample, don't exhaustively scan (context rule E1).

### Graph-Accelerated Mode (Graphify MCP Available)

Use MCP tool calls instead of file reads where possible:

| MCP Tool | Replaces | Purpose |
|----------|----------|---------|
| `get_community` | Directory scanning + file reads | Identify module clusters from graph communities |
| `god_nodes` | Manual grep for dependency hotspots | Find high-coupling nodes |
| `graph_stats` | File counting + manual analysis | Overview metrics (node count, edge density) |
| `get_neighbors` | Import tracing via grep | Trace dependencies for specific modules |

After MCP queries, read 2-3 representative files per community to confirm and
add human-readable context that the graph alone cannot capture.

MCP budget: max 3 queries in this phase. Reserve 2 for Phase 2.

### Legacy Apps (No Documentation)

If the repo has no README, no CLAUDE.md, no .md files at all:

- Work entirely from code: directory structure, imports, exports, entry points
- Detect language from file extensions if no dependency manifest exists
- Analyze import statements to infer module boundaries
- Read main entry point files to understand application purpose
- This is the common case for brownfield — the skill must not assume any docs exist

### Convention Detection

Look for these patterns (report only what is actually found):

| Pattern Type | What to Look For |
|-------------|------------------|
| Architecture | MVC, layered, hexagonal, repository pattern, CQRS, event-driven |
| Naming | camelCase/snake_case, file naming conventions, directory naming |
| Error handling | Try/catch style, Result types, error middleware, custom error classes |
| Dependency injection | Constructor injection, service locator, DI container |
| Testing | Test file co-location vs. separate directory, mocking patterns |
| Configuration | ENV vars, config files, feature flags |

---

## Phase 2: Trace Flows

Starting from entry points, trace how data moves through the system.

### Entry Point Discovery

| Type | Where to Look |
|------|--------------|
| HTTP routes | Route definitions, controller files, API handlers |
| CLI commands | `bin/`, `cmd/`, main files with arg parsing |
| Event handlers | Message queue consumers, event listeners |
| Cron/scheduled | Cron definitions, scheduler config |
| Background workers | Worker files, job processors |

### Flow Tracing

For each major entry point type, trace one representative flow:

```
INPUT → validation → processing → storage → response
```

Document: what transforms the data, where it's persisted, what modules are crossed.

### Dependency & Coupling Analysis

- Map which modules depend on which
- Identify dependency direction (does module A import from B, or both ways?)
- Flag circular dependencies
- Flag god modules (modules everything imports from)
- If graphify MCP available: use `shortest_path` between modules to verify coupling
- If code-review-graph MCP available: use for blast radius analysis (max 3 queries, rule E3a)

Total MCP budget across Phase 1 + Phase 2: max 5 queries combined.

---

## Phase 3: Synthesize — **GATE**

Combine all findings into the architecture map artifact.

### Draft the Map

Write `.understand/architecture-map.md` following the template below.
Target: **80-150 lines.** If it exceeds 150, you are documenting implementation
details — step back to structure and boundaries only.

### Present to User

Show the full draft in conversation. Then:

> **STOP.** "Does this match your understanding? What's wrong, missing, or
> mischaracterized? Are the Open Questions accurate?"

User validates, corrects, or adds context. Incorporate feedback, then write the artifact.

### Write Artifact

```bash
mkdir -p .understand
```

Write `.understand/architecture-map.md` with the validated content.

If `.understand/architecture-map.md` already exists (update mode):
- Merge new findings with existing content
- Preserve user annotations in Open Questions
- Update the `Updated:` date
- Add/remove modules as the codebase has changed

---

## Phase 4: Handoff

### Next Step Table

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| No hashb setup exists | `/init` | Bootstrap with architecture context — map feeds Profile fields |
| hashb exists, compliance unknown | `/audit` | Architecture-aware compliance check |
| Specific feature planned | `/eng` | Plan with module boundaries and coupling known |
| Open questions remain | `/explore` | Investigate specific areas flagged in Open Questions |

**Autonomous mode:** If running inside a recipe chain (e.g., Brownfield Onboarding):
- After mapping → auto-proceed to `/init`

---

## Output Artifact Template

File: `.understand/architecture-map.md`

```markdown
# Architecture Map

> Generated by /understand on {YYYY-MM-DD}. Updated: {YYYY-MM-DD}.
> Source: {standard analysis | graph-accelerated (graphify MCP)}
> Re-run /understand to refresh.

## System Overview
{2-3 sentences: what this system does, who it serves, how it's deployed}

## Module Boundaries
| Module | Location | Responsibility | Public Interface |
|--------|----------|---------------|-----------------|
| {name} | {path/} | {one sentence} | {key exports} |

## Data Flow
{ASCII diagram or concise prose showing: entry → processing → storage → output}

## Entry Points
| Type | Path/Route | Handler | Description |
|------|-----------|---------|-------------|
| {HTTP/CLI/Event/Cron} | {path} | {handler} | {what it does} |

## Key Dependencies
| Dependency | Purpose | Coupling Level |
|-----------|---------|---------------|
| {name} | {why it's used} | {High/Medium/Low — with evidence} |

## Conventions Detected
| Pattern | Where | Example |
|---------|-------|---------|
| {name} | {scope} | {file:line reference} |

## Coupling & Risk Areas
| Area | Concern | Severity |
|------|---------|----------|
| {area} | {what's risky} | {High/Medium/Low} |

## Open Questions
- {Things that could NOT be determined from code alone}
- {Areas where the analysis was uncertain — flagged honestly}

## Evidence Log
| Finding | Source | Method |
|---------|--------|--------|
| {what was found} | {file:line or MCP query} | {read / AST / graph community / grep} |
```

---

## Rules

| Rule | Why |
|------|-----|
| Sample, don't exhaustively scan | Context budget. 5-10 files per module max |
| Evidence Log is mandatory | Every claim must trace to a source. No hand-waving |
| Open Questions must be honest | If you couldn't determine something, say so. Don't guess |
| Graph queries replace grep, not add to them | When MCP answers a structural question, skip the grep |
| No code changes | This skill maps — it never modifies source code |
| Respect MCP budget | Max 5 MCP queries total (graphify + code-review-graph combined) |
| Idempotent | Re-running updates the existing map, never duplicates |
| Preserve user annotations | Open Questions added by the user survive re-runs |
| Legacy-first design | Must work on repos with zero documentation |

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "I already understand this codebase" | If you understood it, you wouldn't need to map it. Implicit understanding evaporates between sessions. |
| "Let me just read a few files and summarize" | That's `/explore`. This skill produces a structured, persistent artifact with evidence. |
| "The architecture is obvious from the directory structure" | Directory structure shows layout, not boundaries, coupling, or data flow. Those require tracing. |
| "We can skip the Evidence Log" | The Evidence Log is what separates analysis from speculation. Without it, the map is a guess. |
| "This codebase is too big to map" | Focus on one subsystem. Pass a scope argument. Partial maps are more valuable than no map. |
| "I'll fill in Open Questions later" | If you can't answer it now, flag it now. Deferred unknowns become assumed knowns. |

---
name: swarm
description: >
  Parallel work decomposition — plan, coordinate, merge streams.
  Use when a task is too large for one stream. Decomposes into parallel streams with worktree isolation, coordinates review, and merges results.
  Modes: /swarm plan, /swarm review, /swarm merge.
argument-hint: "[\"plan\" or \"review\" or \"merge\"]"
---

# Swarm — Parallel Work Streams

Split a goal into independent streams. Each stream gets its own branch and
runs its own skill chain. Merge when done.

**Core idea:** Streams describe *work*, not agents. One agent executes them
sequentially. Multiple agents take one each. The manifest works either way.

---

See `skills/shared/formatting.md` for presentation rules (progress indicators, discussion chunking, table formatting).

---

## Execution Flow (MANDATORY)

> **This is the ONLY valid sequence. Never skip or reorder steps.**
> Each gate marked **STOP** requires completion before proceeding.

```
┌─────────────────────────────────────────────────┐
│ /swarm plan                                     │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ User approves manifest
                     │
┌────────────────────┴────────────────────────────┐
│ Implement streams                               │
│ (parallel if independent, sequential if not)    │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ All streams complete
                     │
┌────────────────────┴────────────────────────────┐
│ /swarm review — per-stream /review              │
├─────────────────────────────────────────────────┤
│  ✓ APPROVED ──────────────────────▸ continue    │
│  ↻ CHANGES REQUESTED ──▸ fix + re-review (2x)  │
│  ■ ESCALATE ──────────────────────▸ stop, ask   │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ All streams APPROVED
                     │
┌────────────────────┴────────────────────────────┐
│ /swarm merge — one at a time, test between each │
└────────────────────┬────────────────────────────┘
                     │
              ■ STOP │ Build + lint + tests pass
                     │
┌────────────────────┴────────────────────────────┐
│ /review — integrated, FULL diff to base         │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────┐
│ /ship — changelog, commit, PR                   │
└─────────────────────────────────────────────────┘
```

**CRITICAL RULE:** After stream agents complete, you MUST invoke `/swarm review`
BEFORE touching git merge. Do NOT merge streams into the base branch without
review verdicts. This is a blocking gate, not a suggestion.

---

## When to Use This

Score the goal first. Single agent is usually enough.

| Signal | Points |
|--------|--------|
| Work spans 2+ independent areas (no shared files) | +3 |
| Each stream can be tested independently | +2 |
| Total scope exceeds ~5 tasks | +1 |
| Work touches shared code both streams need | −3 |
| Total scope is under 3 tasks | −2 |

**Score >= 4:** Suggest `/swarm`. **< 2:** Use the skill chain directly (see `recipes.md`).

---

## `/swarm plan`

### Step 1: Break into streams

**If `/eng` was run previously in this session**, use its outputs as input:
- **Scope card** → system name, constraints, scale targets
- **Implementation order** (Phase 3, Step 0C½) → maps to stream prerequisites and ordering
- **File inventory** (from scope challenge) → maps directly to stream file ownership
- **ADRs** (if architecture mode was used) → constraints for all streams

If `/eng` was not run, gather scope from scratch.

List what needs to happen. Group into streams where each stream owns a
non-overlapping set of files.

**Rule:** If two pieces of work touch the same file, they're the same stream.

### Graph-Aware Independence Check (if code-review-graph MCP is available)

If the consumer's Project Profile lists `code-review-graph`, query the dependency
graph to verify stream independence. Confirm that no stream's file set has
dependencies on another stream's file set beyond declared contracts. If hidden
cross-stream dependencies exist, flag them before manifest approval. Also verify
that shared dependencies (modules imported by multiple streams) are assigned to
prerequisites or infrastructure. Max 1 query.

If code-review-graph is not available, infer independence from file paths and
directory structure.

### Step 2: Identify shared prerequisites

Changes that multiple streams depend on — build these first, merge to base,
then fork streams from the updated base.

### Step 3: Write the manifest

```
SWARM: {goal} ───────────────────────────────────────────────────

  Base          {branch}

  PREREQUISITES
  ─────────────────────────────────────────────────
  SP-1  {what} — branch: swarm/{goal}/sp-1

  STREAMS
  ─────────────────────────────────────────────────
  A  swarm/{goal}/a — {scope + files owned}
  B  swarm/{goal}/b — {scope + files owned}

  CONTRACTS (cross-stream boundaries)
  ─────────────────────────────────────────────────
  A → B       {what A exports that B consumes — types, events, APIs}
  SP-1 → A,B  {shared interfaces or schemas}

  INFRASTRUCTURE (shared files not owned by any stream)
  ─────────────────────────────────────────────────
  {list files: DI/service config, middleware/pipeline setup, CI config,
   docker-compose, env templates, shared route registrations, etc.}
  → Assign to: SP-{N} or dedicated stream

  MERGE ORDER: SP-1 → A → B

─────────────────────────────────────────────────────────────────
```

**CONTRACTS** section is required when streams have dependencies. If streams
are fully independent (no shared interfaces), write `CONTRACTS: none`.

**INFRASTRUCTURE** section lists files that need modification but don't
belong to any stream's feature scope (config, CI, env templates, etc.).
Every listed file must be assigned to a prerequisite or a dedicated stream.

> **Completeness rule:** Every file that the feature touches must be owned
> by exactly one stream, prerequisite, or infrastructure entry. If a file
> is not in any stream's ownership, it will not be modified and will not be
> reviewed. Before approving, enumerate ALL files expected to change and
> verify each appears exactly once. Unowned files = manifest is incomplete.

**STOP.** User approves the manifest before work begins.

---

## Implement Streams

Launch stream agents after manifest is approved. Each stream works on its
own branch in isolation.

**Launch:** Use Agent tool with `isolation: "worktree"` for each stream.
Independent streams run in parallel. Dependent streams run sequentially
(respecting the manifest's prerequisite and merge order).

**After ALL streams complete**, report status:

```
SWARM STATUS ────────────────────────────────────────────────────

  S1  {name}    ✓ commit {hash}
  S2  {name}    ✓ commit {hash}
  S3  {name}    ✓ commit {hash}

─────────────────────────────────────────────────────────────────
```

**Do NOT merge. Proceed to `/swarm review`.**

---

## `/swarm status`

Check git state for each stream branch. Report what's done, what's active,
and flag any files modified outside a stream's scope.

---

## `/swarm review`

Before merging, each stream gets a peer review from a fresh-context agent.

1. For each stream: `/review` against the stream's scope and requirements
2. Reviewer gets: stream diff, manifest (scope + file ownership), active rules
3. **Cross-stream contract check:** If Stream B consumes exports from Stream A,
   verify the contract matches (types, signatures, event schemas)
4. Verdicts per stream: APPROVED / CHANGES REQUESTED / ESCALATE

**Autonomous flow:**
- All streams APPROVED → proceed to merge
- Any CHANGES REQUESTED → implementing agent fixes, re-review (max 2 cycles)
- Any ESCALATE → stop, present to user

**Report to user:**

```
SWARM REVIEW ────────────────────────────────────────────────────

  S1          APPROVED
  S2          APPROVED
  S3          CHANGES REQUESTED → fixed → APPROVED
  Contracts   VERIFIED

─────────────────────────────────────────────────────────────────
Proceed to /swarm merge?
```

---

## `/swarm merge`

1. Verify all streams are done with passing build, lint, and tests
2. Verify all streams have APPROVED review verdict
3. Create integration branch from base
4. Merge prerequisites, then streams in defined order — build, lint, and test after each
5. **Cross-stream integration check** after each merge:
   - Do imports resolve? Do types match across stream boundaries?
   - Do events published by one stream get handled by another?
6. If conflicts: stop, show them, user decides
7. Full build, lint, and test suite on the combined result
8. Clean up swarm artifacts:
   - Delete worktree directories created for streams (warn if uncommitted changes)
   - Delete local and remote swarm branches (`swarm/*/*`)
   - Remove any `.swarm/` state files
9. Hand off to `/review` for integrated review, then `/ship`

**Report after each merge:**

```
MERGE PROGRESS ──────────────────────────────────────────────────

  S1  merged ✓  tests: ✓
  S2  merged ✓  tests: ✓  (conflicts resolved: file.cs)
  S3  merging...

─────────────────────────────────────────────────────────────────
```

> **Integrated review:** Stream reviews check code quality within boundaries.
> The integrated `/review` after merge checks the combined diff against the
> base branch for cross-cutting issues: missing runtime wiring, contract
> mismatches between streams, dependency conflicts, and integration gaps.
> Pay special attention to merge resolution code — lines that exist in the
> combined diff but in neither stream's individual diff. This step is
> mandatory, not optional.

---

## Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| All streams merged, tests pass | `/review` → `/ship` | Integrated review of full diff, then create PR |
| Stream review failed after 2 cycles | Escalate to user | Agent can't resolve the issues |
| Merge conflicts can't be auto-resolved | Escalate to user | Needs manual conflict resolution |

**Autonomous mode:** If running inside a recipe chain:
- All merged + tests green → auto-proceed to `/review`, then `/ship`
- Blocked → stop the chain, present the issue to user

---

## Rules

- **Single agent is the default.** Only swarm when it clearly helps.
- **Prerequisites first.** Shared code changes before parallel streams.
- **Streams own their files.** Touch outside your scope = stop and escalate.
- **NEVER merge without review.** Each stream gets `/review` before integration. No exceptions.
- **Follow the execution flow.** The sequence at the top of this file is mandatory, not advisory.
- **Merge one at a time.** Build, lint, and test between each merge.
- **Cross-stream contracts validated.** If streams share boundaries, verify contracts match.
- **No nested swarms.** One level of decomposition only.

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "Everything should be parallelized for speed" | Parallel streams with shared files create merge conflicts and integration bugs. Only swarm when streams are truly independent. |
| "We can sort out the merge conflicts later" | Merge conflicts after parallel development are 3-5x harder to resolve than sequential conflicts. If streams share files, they belong in the same stream. |
| "Stream reviews are unnecessary — we'll review the merged result" | Per-stream reviews catch stream-specific issues before they compound during integration. The integrated review checks cross-cutting concerns. Both are needed. |
| "The prerequisites can be done in parallel with streams" | Streams depend on prerequisites. Building on an unstable foundation means rebuilding when the prerequisite changes. Prerequisites first, always. |
| "This small overlap between streams is fine" | "Small" file overlaps become merge conflict nightmares. If two streams touch the same file, merge them into one stream or extract the shared code as a prerequisite. |

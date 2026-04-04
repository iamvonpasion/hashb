---
description: Pre-planning research — libraries, APIs, patterns, ecosystem.
---

# Pre-Planning Research

Investigate before deciding. Ground architecture choices in current docs,
not stale training data.

**This is read-only. No code changes.**

---

## When to Suggest This

Proactively suggest `/research` before `/eng` when:
- Feature involves a new dependency or library choice
- Team is entering unfamiliar API territory
- Multiple viable approaches exist and the tradeoffs aren't obvious
- Last known docs are >6 months old or the ecosystem moves fast

Skip when: the team already knows the stack, or the feature uses only
existing patterns with no new dependencies.

## Presentation Rules

1. **Wizard flow** — present one phase at a time. Never dump everything at once.
2. **Summary table before detail** — every phase opens with a table, expands only where needed.
3. **Tables over prose** — use tables for comparisons, options, checklists. Prose for context only.
4. **Discussion chunking** — when a follow-up response would be too dense to digest in one shot, present a numbered big-picture overview first, then discuss each point one at a time, waiting for user input between points. Use judgment: chunk whenever the response feels like a wall of text, not at a fixed threshold.
5. **Progress indicator** — every output starts with:

```
/research ═══════════════════════════════════════════════════════

  ▸ Phase 0  Existing Landscape    ~2 min
  ○ Phase 1  Scope                 ~2 min
  ○ Phase 2  Investigate           ~8 min
  ○ Phase 3  Compare               ~3 min
  ○ Phase 4  Recommend             ~2 min

═════════════════════════════════════════════════════════════════
```

Update `▸` (current), `✓` (done), `○` (pending), `—` (skipped) as phases progress.
Completed phases show a status note on the right (e.g., `✓ 3 questions`, `✓ Zustand`).
Skipped phases omit the time estimate.

---

## Phase 0: Existing Landscape

Before researching external options, scan what already exists and what we've
learned from past work.

### Prior Learnings

Check `.retro/` for retro reports mentioning the technology or pattern being
researched:

```bash
ls .retro/retro-*.md 2>/dev/null | head -10
```

If prior retros mention relevant findings (e.g., "Stripe API v2 had
rate limiting issues", "Redis caching caused stale data in notifications"),
surface them as context for the current research.

1. **Codebase scan** — search for code relevant to the research topic.
   - Existing implementations (partial or full)
   - Libraries already in dependencies (`package.json`, `.csproj`, `requirements.txt`)
   - Patterns already established (state management, API layers, etc.)
2. **Present findings** — "You already have X at `path/`. Research will evaluate
   whether to extend, replace, or complement it."

If the user provides file paths or mentions existing code, read it in this phase.

**Output:** Brief inventory of what exists. This shapes the research questions.

Skip if: greenfield project with no existing code.

---

## Phase 1: Scope

Before investigating, define what needs answering.

**Ask:**
1. What decisions need research? (library choice, API design, pattern selection, migration path)
2. What constraints exist? (bundle size, license, browser support, existing stack compatibility)
3. What's the decision deadline? (quick spike vs. thorough evaluation)

**Output:** 2-5 research questions, each with success criteria.

```
RESEARCH QUESTIONS:
  1. Which state management library fits our React stack?
     Success: comparison table with bundle size, TS support, learning curve
  2. Does the Stripe API support idempotent subscription upgrades?
     Success: confirmed with current docs, code example
```

**STOP.** Confirm research questions with the user before investigating.

---

## Phase 2: Investigate

### Source Priority

Use sources in this order — higher sources override lower:

1. **MCP servers** — Context7 (required) + any additional servers declared in the
   consumer's Project Profile under `MCP Servers`. If the consumer declares
   additional MCP servers (e.g., internal wiki, company knowledge base), query
   those alongside Context7.
2. **Official docs** — library/API documentation sites
3. **Web search** — current blog posts, release notes, GitHub issues
4. **GitHub repo** — stars, maintenance activity, open issues, last release date

> **Context7 is required.** Every research session must query Context7 for
> library documentation. Additional MCP servers declared in the Project
> Profile are queried alongside it. The workflow below describes Context7
> specifically, but the same pattern applies to any MCP documentation server.

### Using Context7

Context7 provides up-to-date, verified documentation. Use it as the **first
stop** for any library or API research.

**Workflow:**
1. **Resolve the library ID first** — call `resolve-library-id` with the
   library name and your research question. This returns candidate libraries
   with reputation scores and available versions.
2. **Pick the latest stable version** — from the resolved results, select the
   version-pinned ID (format: `/org/project/version`) for the latest stable
   release. Prefer libraries with **High** source reputation and high
   benchmark scores.
3. **Query docs** — call `query-docs` with the resolved library ID and a
   specific question. Be precise: "How to set up JWT auth in Express.js" not
   "auth".

**Rules for Context7 usage:**
- Max 3 `resolve-library-id` calls per research question.
- Max 3 `query-docs` calls per research question.
- Always query Context7 first. If Context7 has the library, use its docs as the primary source.
- If Context7 lacks a library, proceed to official docs + web search, but note the gap in findings.
- Always note the version you queried against in your findings.

### Version & Stability Bias

When evaluating libraries, dependencies, or tools:

- **Always target the latest stable release.** Not canary, not RC, not "next".
- **Verify "latest" via Context7 or official source.** Training data may be
  outdated — do not assume you know the current version.
- **Reputable sources only.** Prefer libraries backed by known orgs, with
  active maintenance, established user bases, and clear governance.
- **Reject abandoned projects.** No release in >12 months + declining activity
  = disqualified unless no alternative exists.

### For Library/Stack Evaluation (`/research stack`)

For each candidate, start with Context7 `resolve-library-id` to get reputation
and version info, then `query-docs` for specifics.

Gather:
- **Maturity** — version, age, production users
- **Maintenance** — last release, commit frequency, response to issues
- **Size** — bundle size (bundlephobia), tree-shakeable?
- **Compatibility** — works with our stack? TypeScript support?
- **Community** — npm downloads, GitHub stars, Stack Overflow activity
- **License** — compatible with our project?
- **Migration cost** — if replacing something, how painful?

### For API Deep-Dive (`/research api <name>`)

- Query Context7 first for the API's library docs (latest stable version)
- Identify: auth model, rate limits, pagination, error format, webhooks
- Find: breaking changes in recent versions, known gotchas, deprecation notices
- Check: SDK quality for our language, type definitions available?

### For General Research (`/research`)

- Map the solution space: what approaches exist?
- For each approach: who uses it in production? What scale?
- What are the known failure modes?
- What did teams regret after choosing each option?

### Freshness Check

For every finding, note the source date and version. Flag anything >12 months
old as potentially stale. If Context7 and web search disagree, trust Context7
for API specifics and web search for ecosystem trends.

---

## Phase 3: Compare

For library/stack choices, produce a structured comparison using a markdown
table (renders correctly at any column width — never use hand-drawn grids):

```
COMPARISON: State Management Libraries
```

| Dimension        | Zustand     | Jotai       | Redux TK    |
|------------------|-------------|-------------|-------------|
| Maturity         | v4.5, 2021  | v2.8, 2022  | v2.2, 2019  |
| Bundle size      | 1.1 kB      | 2.4 kB      | 11 kB       |
| TS support       | Native      | Native      | Native      |
| Learning curve   | Low         | Low         | Medium      |
| DevTools         | Basic       | Basic       | Excellent   |
| SSR support      | Yes         | Yes         | Yes         |
| Our stack compat | ✓           | ✓           | ✓           |
| Risk             | Smaller API | Newer       | Boilerplate |

For API/pattern decisions, compare approaches with a card per option:

```
APPROACH A: Direct API calls with retry
──────────────────────────────────────────────────
  Effort: S    Risk: Low    Complexity: Low
  Pro: Simple, fast to implement, easy to debug
  Con: Coupled to provider — acceptable if
       switch is unlikely

APPROACH B: Adapter pattern with provider interface
──────────────────────────────────────────────────
  Effort: M    Risk: Low    Complexity: Medium
  Pro: Swap providers without touching business logic
  Con: Over-engineering if we'll never switch —
       YAGNI risk

  ⚠ Prefer A unless there's a concrete plan
    to swap providers.
```

---

## Phase 4: Recommend

Pick one. Say why in 2-3 sentences. Be opinionated.

```
RECOMMENDATION: Zustand
  Why: Smallest bundle, simplest API, sufficient for our use case.
  Zustand's lack of middleware ecosystem is irrelevant — we don't need
  middleware. Redux TK's devtools are nice but not worth 10x bundle size
  for our app scale.

  Risks:
  - If we later need time-travel debugging, we'd miss Redux DevTools
  - Zustand's community is smaller — fewer blog posts and examples

  Unknowns:
  - Performance at >500 state subscribers (unlikely for our app)
```

---

## Output Format

Present a **research brief** (not deep-dive) by default. Keep it under ~300 words.

### Default: Executive Brief

```
✓ RESEARCH BRIEF ────────────────────────────────────────────────

  Questions       {N answered} / {N total}
  Sources         Context7, {official docs}, {web}
  Existing code   {what was found in Phase 0, or "greenfield"}
  Freshness       All sources < {N} months old

  {For each research question: 2-3 sentence answer + recommendation}

  RECOMMENDATION
  ─────────────────────────────────────────────────
  Choice       {choice}
  Why          {2-3 sentences}
  Risks        {known risks}
  Unknowns     {what we couldn't verify}

  DEEP-DIVES AVAILABLE
  ─────────────────────────────────────────────────
  1.  {topic} — comparison table, detailed analysis
  2.  {topic} — API specifics, migration cost

  ➤ NEXT: /eng (see Next Step below)

─────────────────────────────────────────────────────────────────
```

### On request: Deep-dive

When the user asks to expand a topic, provide the full comparison table
(Phase 3 format) and detailed analysis for **that topic only**.

**Rule:** Never dump all deep-dives unprompted. One topic at a time.

---

## Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Architecture decisions needed | `/eng arch` | Research informs architecture decisions |
| Implementation planning | `/eng` or `/eng impl` | Research informs tech choices |
| Still evaluating options | Continue `/research` with deep-dives | Not enough data to decide |

**Autonomous mode:** If running inside a recipe chain (e.g., Architecture recipe),
auto-proceed to `/eng arch` with the Research Brief as input.

---

## Rules

- **Read-only.** No code changes, no file creation.
- **Context7 mandatory.** Always query Context7 before other sources. Never skip it, even if you expect it lacks coverage.
- **Latest stable.** Default to the latest stable version of any library or tool. Never recommend outdated versions without explicit justification.
- **Source hierarchy.** Context7 → official docs → web search. Higher overrides lower.
- **Flag staleness.** Note source dates and versions. Warn on anything >12 months old.
- **Stay focused.** Answer the research questions. Don't write a thesis.
- **Be opinionated.** Pick a recommendation. "It depends" is not a conclusion.
- **Favor simplicity.** When two options are close, pick the simpler one. Fewer dependencies, less config, smaller API surface, less abstraction = better. Only recommend complex solutions when simple ones demonstrably fail the requirements.
- **No over-engineering.** Don't recommend adapter patterns, plugin systems, or abstraction layers unless the user has a proven need. YAGNI applies.
- **Admit unknowns.** If you can't verify something, say so. Don't fabricate.
- **Don't over-research.** If the answer is obvious after Phase 2, skip to recommend.

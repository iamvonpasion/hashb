---
name: explore
description: >
  Unstructured exploration — think through ideas, investigate the codebase, compare approaches,
  diagram relationships. No artifacts, no structure, no gates. Transitions to /spec or /eng
  when ready.
  Use when requirements are unclear, the problem space is new, or you need to think before committing to a plan.
argument-hint: "[topic or question]"
---

# Explore

Think freely. Investigate. Compare. Diagram. No structure required.

This is the space between "I have a vague idea" and "I'm ready for `/spec`."
No artifacts are produced. No files are written. No gates exist.

**This is read-only. No code changes. No artifacts.**

---

## When to Use

| Signal | Why explore? |
|--------|-------------|
| "I'm not sure what we need yet" | Problem space is unclear — think before specifying |
| "Let me understand the codebase first" | Need to investigate before planning |
| "What are our options here?" | Compare approaches without committing |
| "How does this part of the system work?" | Codebase archaeology |
| "I have an idea but it's half-formed" | Develop the idea before formalizing |
| "What would break if we changed X?" | Impact analysis before planning |

**Don't use when:** You already know what to build (use `/spec` or `/quick`), you have a bug (use `/fix`), or you need a library comparison (use `/research`).

---

## How It Works

There are no phases. No templates. No mandatory structure.

Start from what the user said and follow the thread. Use whatever format
makes the thinking clearest — prose, diagrams, tables, code snippets,
questions, lists.

### What You Can Do

- **Investigate the codebase** — read files, trace call chains, map dependencies
- **Compare approaches** — pros/cons, trade-offs, effort estimates
- **Diagram relationships** — ASCII diagrams, dependency graphs, data flows
- **Ask questions** — surface unknowns, challenge assumptions, identify risks
- **Prototype ideas** — pseudocode, rough sketches, "what if" scenarios
- **Map the problem space** — who's affected, what exists today, what's missing
- **Analyze impact** — what would change, what could break, what depends on what

### What You Don't Do

- Write code or modify files
- Produce formal artifacts (proposals, specs, designs, task lists)
- Create gates or checkpoints
- Follow a template or phase structure

---

## Before You Start

### Read Project Profile

Check the consumer's `CLAUDE.md` for a **Project Profile** section. If present,
use it to understand the stack, architecture, and constraints.

### Scan for Prior Work

```bash
# Check for existing specs, designs, retros that might inform this exploration
ls docs/ specs/ .retro/ .history/ TODOS.md 2>/dev/null
```

Read relevant files if they exist — prior context shapes better exploration.

---

## Exploration Styles

Use whatever style fits. Mix freely.

### Codebase Investigation

When the user wants to understand how something works:

```
EXPLORING: {topic}
---

{findings as you read through the code}

Key files:
  - {file:line} — {what it does}
  - {file:line} — {what it does}

How it works:
  {explanation in whatever form is clearest}

Questions this raises:
  - {question}
  - {question}
```

### Approach Comparison

When the user is weighing options:

| Approach | How it works | Effort | Risk | Fits us? |
|----------|-------------|--------|------|----------|
| {A} | {description} | S/M/L | {risk} | {yes/no + why} |
| {B} | {description} | S/M/L | {risk} | {yes/no + why} |

### Impact Analysis

When the user wants to understand what a change would affect:

```
IF WE CHANGE: {thing}

  DIRECTLY AFFECTED
  - {component} — {how}
  - {component} — {how}

  INDIRECTLY AFFECTED
  - {component} — {via what dependency}

  SAFE (not affected)
  - {component} — {why it's isolated}

  UNKNOWNS
  - {what we'd need to check}
```

### Free-Form Thinking

When the user just wants to think out loud:

Follow the conversation. Ask clarifying questions. Challenge assumptions.
Surface things the user hasn't considered. There's no format requirement.

---

## Transitioning Out

When the exploration reaches a natural conclusion — the user has enough
clarity to act — suggest the appropriate next step:

| Signal | Transition to |
|--------|--------------|
| Problem is clear, need requirements | `/spec` |
| Problem is clear, small enough for quick plan | `/quick` |
| Need to evaluate specific libraries/APIs | `/research` |
| Ready to plan implementation | `/eng` |
| Found a bug during investigation | `/fix` |
| Need UI/UX thinking | `/design` |
| Still unclear — more investigation needed | Continue exploring |

**Transition format:**

```
---
Exploration suggests: {one-sentence summary of what we learned}

Ready for: /spec (or whichever skill fits)
  Reason: {why this is the right next step}
```

Don't force transitions. If the user wants to keep exploring, keep exploring.
The value is in the thinking, not in reaching a conclusion quickly.

---

## Rules

| Rule | Why |
|------|-----|
| Read-only | No code changes, no file writes, no artifacts |
| No forced structure | Use whatever format makes thinking clearest |
| No gates | Don't stop to ask permission — just keep exploring |
| Follow the thread | User's curiosity drives direction, not a template |
| Be honest about unknowns | "I don't know" is better than guessing |
| Suggest transitions, don't force them | The user decides when they're done exploring |
| Read before speculating | Investigate the actual codebase, don't guess at structure |
| Keep it conversational | This isn't a report — it's collaborative thinking |

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "I already understand this — let me skip to /spec" | If you understood it, you wouldn't be exploring. Premature specificity produces specs that solve the wrong problem. |
| "Let me write some code to see if it works" | Explore is read-only. Writing code before understanding the problem creates attachment to a solution that may be wrong. |
| "I'll just produce a quick spec instead" | Exploration and specification are different activities. Explore is for when you don't know what to specify yet. |
| "This is taking too long — we should just start building" | Time spent understanding saves 3-5x time spent rebuilding. The cheapest code to change is code you haven't written yet. |
| "I have enough context from the conversation" | Read the actual code. Assumptions about codebase structure are wrong more often than right. |

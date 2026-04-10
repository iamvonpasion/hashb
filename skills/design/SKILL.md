---
name: design
description: >
  UX/UI design — wireframes, component mapping, interaction design, accessibility.
  Use when a feature needs UI: wireframes, user flows, component structure, responsive breakpoints.
  Modes: /design (full pass), /design review (review existing UI).
argument-hint: "[feature or \"review\"]"
---

# Design

Translate product requirements into UI requirements. Define **what the user
sees and does**, before engineering decides **how to build it**.
Stream output continuously — gate only at decision points.

**This is read-only. No code changes.**

---

## When to Use

| Signal | Use /design |
|--------|-------------|
| Feature has user-facing screens or interactions | Yes |
| `/spec` produced UI-related user stories | Yes |
| Redesigning or improving existing UI | Yes — `/design review` mode |
| Pure backend, API, or infrastructure work | No — skip to `/eng` |
| Component library or design system changes | Yes |

> **Invoke explicitly.** This skill is not auto-triggered. Use it when the
> feature requires UI/UX work. The feature recipe includes it as a conditional
> step: `/spec → /design (if UI) → /eng`.

---

## Presentation Rules

See `skills/shared/formatting.md` for formatting rules (tables, code blocks, output style, workflow discipline).

---

## Complexity Routing

Assess complexity at the start and adjust depth:

| Complexity | Signal | Behavior |
|------------|--------|----------|
| **Small** | 1-2 screens, simple forms or displays, existing patterns | Combine all phases into one response. 1 gate (final confirmation) |
| **Medium** | 3-5 screens, some new components, standard interactions | Stream phases continuously. 2 gates (after wireframes, after everything) |
| **Large** | 6+ screens, new design patterns, complex state machines | Full flow. 2 gates (after wireframes, after interactions+a11y) |

---

## Phase Roadmap (present this first)

| Phase | Name | When it runs | What happens |
|-------|------|-------------|-------------|
| 0 | Inventory | Always | Scan existing UI patterns, components, identify what's new vs reusable |
| 1 | Layout & Wireframes | Always | Page structure, content hierarchy, responsive behavior |
| 2 | Interaction Design | Always | State machines, user flows, loading/error/empty states |
| 3 | Accessibility & Responsive | Always | WCAG requirements, focus management, breakpoints |
| 4 | Summary | Always | Design spec handoff for `/eng` |

---

## Before You Start

### Read Project Profile

Check the consumer's `CLAUDE.md` for a **Project Profile** section:
- Stack → which component framework (React, Vue, native, etc.)
- Architecture → SPA, SSR, mobile, multi-platform

### Read Spec Handoff

If `/spec` was run, consume its handoff summary and build the **Spec Traceability Map**:
- User stories → map to screens/views
- Functional requirements → map to UI behaviors
- Acceptance criteria → map to interaction states
- NFRs → performance, accessibility targets

### Spec Traceability Map (mandatory if /spec was run)

Before designing, create a traceability map linking spec to screens:

| Story | Requirement | Screen(s) | Acceptance Criteria |
|-------|-------------|-----------|---------------------|
| S1 | FR-1, FR-2 | {screen name} | AC-1, AC-2 |
| S2 | FR-3 | {screen name} | AC-3, AC-4 |

This map is carried through all phases — every wireframe, state, and interaction
must trace back to at least one row. Orphan screens (no spec link) are flagged.
Orphan requirements (no screen) are escalated to user.

### Scan Existing UI

```bash
# Detect component library / design system
ls src/components/ src/ui/ src/design-system/ app/components/ 2>/dev/null
# Detect existing pages/views
ls src/pages/ src/views/ src/app/ app/ pages/ 2>/dev/null
# Check for design tokens
ls src/styles/ src/theme/ tailwind.config.* 2>/dev/null
```

---

## Phase 0 — Inventory

Stream inventory directly, then continue to Phase 1.

### Existing Pattern Scan

| Category | What exists | Reusable? |
|----------|-------------|-----------|
| Component library | {list key components found} | {which apply to this feature} |
| Design tokens | {colors, spacing, typography source} | {yes / missing / partial} |
| Layout patterns | {existing page layouts} | {which match the new feature} |
| Form patterns | {existing form components, validation} | {which apply} |
| Navigation patterns | {existing nav, routing} | {needs new routes?} |

### New vs Reuse Assessment

| Screen/View | Existing Component | Action |
|-------------|-------------------|--------|
| {screen from spec} | {matching component or "none"} | Reuse / Extend / New |

### Design Scope

```
DESIGN SCOPE ────────────────────────────────────────────────────

  Feature       {name from spec}
  Screens       {N new, M modified}
  Components    {N new, M reused, K extended}
  Framework     {from Project Profile}

  SCREENS TO DESIGN
  ─────────────────────────────────────────────────
  1.  {screen name}               ← Story S1
  2.  {screen name}               ← Story S2, S3

─────────────────────────────────────────────────────────────────
```

Continue streaming to Phase 1 (no stop here).

---

## Phase 1 — Layout & Wireframes

Present ALL screen wireframes, then gate.

### Per-Screen Wireframe

Read `skills/design/templates.md` for ASCII wireframe templates (Dashboard,
Form, List, Detail). Pick the closest template and adapt it.

**Rules for wireframes:**
- Pick the closest template and adapt — don't draw from scratch
- Use box-drawing characters (`┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴`), keep lines ≤60 chars
- Use `[ Button ]` for actions, `● ○` for status, `▸ ◂` for nav arrows
- Use `▼` for dropdowns, `☐ ☑` for checkboxes, `🔍` for search
- Always show: header/nav, main content, primary action

### Content Hierarchy

| Priority | Element | Content | Source |
|----------|---------|---------|--------|
| 1 (primary) | {element} | {what it shows} | {data source} |
| 2 (secondary) | {element} | {what it shows} | {data source} |
| 3 (tertiary) | {element} | {what it shows} | {data source} |

### Component Mapping

| Wireframe Element | Component | Status | Props/Config |
|-------------------|-----------|--------|--------------|
| {element in wireframe} | {component name} | Exists / New / Extend | {key props} |

### Responsive Behavior

| Breakpoint | Layout Change |
|------------|---------------|
| Desktop (>1024px) | {default layout} |
| Tablet (768-1024px) | {what changes} |
| Mobile (<768px) | {what changes — stack, collapse, hide} |

> **GATE 1.** Present ALL wireframes together. User reviews and comments on any.
> After approval, stream Phases 2-3 continuously.

---

## Phase 2 — Interaction Design

Stream all of Phase 2 continuously after wireframe approval.

### User Flow

Map the complete flow through the feature using box-drawing characters.
Keep node labels short (≤15 chars) so alignment holds.

```
FLOW: {feature name}

  ┌───────────┐     ┌───────────┐     ┌───────────┐
  │ Entry     │────▸│ Screen 1  │────▸│ Screen 2  │
  └───────────┘     └─────┬─────┘     └─────┬─────┘
                          │                  │
                    ┌─────┴─────┐      ┌─────┴─────┐
                    │ Loading   │      │ Success   │
                    └─────┬─────┘      └───────────┘
                          │
                    ┌─────┴─────┐
                    │ Error     │──▸ [Retry]
                    └───────────┘
```

**Flow rules:**
- One flow per user journey (happy path + key branches)
- Use `────▸` for forward flow, `──▸` for branches
- Branch downward for alternate/error paths
- Keep nodes as `┌───┐ │ text │ └───┘` boxes, not `[brackets]`
- Label edges when the trigger isn't obvious

### State Machine (per interactive component)

For components with multiple states (forms, wizards, toggles, etc.),
document states, transitions, and user-facing behavior in one table:

| State | Trigger | Next State | UI Change | Error Recovery |
|-------|---------|------------|-----------|----------------|
| idle | user clicks {element} | loading | Show spinner, disable button | — |
| loading | data received | success | Show results, hide spinner | — |
| loading | request fails | error | Show error message | Retry button / auto-retry |
| loading | timeout (>5s) | error | Show timeout message | Retry button |
| error | user clicks retry | loading | Clear error, show spinner | — |
| success | user clicks {action} | confirming | Show confirmation dialog | — |

Include error recovery and non-obvious interactions (drag-to-reorder, optimistic
updates, multi-step flows) directly in the state machine. Skip universal behaviors
(button press feedback, standard form validation) — those are implementation details.

> **Error UX principle:** Messages describe the user's situation, not the technical
> cause. Recovery actions must be actionable — "try again" with a button, not just text.

### State Inventory (Large complexity only)

> Skip this for Small/Medium features — the state machine table above is sufficient.

For features with 6+ screens, add a matrix showing which states apply per screen:

| State | Screen 1 | Screen 2 | Screen N |
|-------|----------|----------|----------|
| **Empty** | {what shows} | {what shows} | {what shows} |
| **Loading** | {what shows} | {what shows} | {what shows} |
| **Error** | {what shows} | {what shows} | {what shows} |
| **Full** | {default view} | {default view} | {default view} |

Mark cells "n/a" for states that don't apply. Only include states relevant to the feature.

---

## Phase 3 — Accessibility & Responsive

Stream directly after Phase 2 — no gate between them.

### Accessibility Requirements

| Requirement | Screen(s) | Design Expectation |
|-------------|-----------|-------------------|
| Keyboard navigation | All | {tab order, focus trapping for modals} |
| Focus management | {modals, drawers, dynamic content} | {where focus moves on open/close} |
| Error announcements | {forms} | {how errors are surfaced to screen readers} |
| Color contrast | All | WCAG AA minimum (4.5:1 text, 3:1 large) |

Only include requirements relevant to this feature. Don't pad with boilerplate.
ARIA roles, landmarks, and implementation-level attributes belong in `/eng`.

---

## Phase 4 — Summary

> **GATE 2.** Present the full design spec for user approval.

### Design Spec

```
✓ DESIGN COMPLETE ───────────────────────────────────────────────

  Feature         {name}
  Screens         {N} designed
  Components      {N new} + {M reused} + {K extended}
  States          {N} state transitions mapped
  Accessibility   WCAG {level} targeted

  DELIVERABLES
  ─────────────────────────────────────────────────
  Wireframes      {N} screens
  State machines  {N} interactive components
  User flows      {N} flows mapped
  A11y reqs       {N} requirements

  ➤ NEXT: /eng (see Next Step below)

─────────────────────────────────────────────────────────────────
```

### Next Step

| Condition | Next Skill | Why |
|-----------|-----------|-----|
| Always after /design | `/eng` | Technical planning, architecture decisions, TDD plan |
| Need library research first | `/research` → `/eng` | Evaluate UI framework options before committing |

**Autonomous mode:** If running inside a recipe chain, auto-proceed to `/eng`.
Pass both the Spec Handoff Summary and Design Handoff Summary as input.

### Handoff Summary

Compressed handoff for `/eng`. The full design phases are available in conversation
context — `/eng` references them directly when it needs detail.

```
DESIGN → ENG:
  Screens:        {N} ({list names})
  Components:     {enumeration below}
  Key states:     {most complex state machines — name them}
  Error UX:       {error scenario table below}
  A11y:           WCAG {level}, {key requirements}
  Data needs:     {data flow matrix below}

  COMPONENT ENUMERATION
  ─────────────────────────────────────────────────
  | Component | Status | Key Props | Screen(s) |
  |-----------|--------|-----------|-----------|
  | {name}    | New    | {props}   | {screens} |
  | {name}    | Reuse  | —         | {screens} |
  | {name}    | Extend | {changes} | {screens} |

  ERROR SCENARIO TABLE
  ─────────────────────────────────────────────────
  | Scenario | User Message | Recovery Action | Screen |
  |----------|-------------|-----------------|--------|
  | {error}  | {message}   | {button/action} | {screen} |

  DATA FLOW MATRIX
  ─────────────────────────────────────────────────
  | Screen | Data Source | Pattern | Notes |
  |--------|------------|---------|-------|
  | {name} | {API/store} | {fetch/subscribe} | {cache, polling, etc.} |
```

This summary preserves full detail across skill boundaries — `/eng` does not
need conversation context to understand the design.

---

## `/design review` Mode

Review existing UI against the spec or general heuristics. Use when
the UI is already built and needs evaluation.

### Review Checklist

| Category | Check | Status |
|----------|-------|--------|
| Spec compliance | Does UI match the spec requirements? | {pass / gaps} |
| State coverage | All states handled (empty, loading, error, overflow)? | {pass / gaps} |
| Accessibility | Keyboard nav, screen readers, contrast, focus management? | {pass / gaps} |
| Responsive | Works at all declared breakpoints? | {pass / gaps} |
| Consistency | Uses existing design patterns and components? | {pass / deviations} |
| Interaction feedback | Every action has visible feedback? | {pass / gaps} |

Output: findings in the same format as `/review` (BLOCKING / WARNING / SUGGESTION).

---

## Rules

| Rule | Why |
|------|-----|
| Read-only | No code changes. Design is a specification artifact |
| Stream by default | Output continuously. Gate only at decision points (wireframes, final approval) |
| 2 gates default | Gate 1: wireframe approval. Gate 2: full design approval. Add gates when a user decision is genuinely needed (competing layout options, unclear interaction patterns). Don't gate for confirmation — gate for decisions |
| ASCII wireframes only | Agent cannot produce visual mockups. Templates in `skills/design/templates.md` |
| Present all wireframes together | Don't serialize screen-by-screen approval. Present all, let user comment on any |
| Map to spec | Every screen traces back to a user story. No orphan screens |
| State machine is the source of truth | One table per component covers states, transitions, error recovery, and non-obvious interactions |
| State inventory is Large-only | Small/Medium features use state machine tables; 6+ screen features add the cross-screen matrix |
| Reuse first | Check existing components before designing new ones |
| Accessibility stays at design level | WCAG requirements and focus management yes; ARIA roles and landmarks are `/eng`'s job |
| No implementation details | "Show error message" not "render ErrorBanner with variant=destructive." That's `/eng`'s job |
| Responsive in Phase 1 table | Breakpoint behavior defined per screen in the table. ASCII breakpoint wireframes only on request |
| Respect Project Profile | Adapt component expectations to the declared framework |
| Complexity routing | Small features get single-pass output. Large features get full flow. Match depth to scope |

---

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "We'll handle the empty/error/loading states later" | States not designed are states not implemented. Users hit empty and error states more often than the happy path — design them first. |
| "Accessibility can be added after launch" | Retroactive accessibility is 10x harder. Focus management, keyboard nav, and contrast ratios shape layout decisions — they're not a layer you bolt on. |
| "This screen is straightforward, no wireframe needed" | Wireframes expose content hierarchy, responsive behavior, and component reuse before a line of code is written. Skipping them means discovering layout problems during implementation. |
| "Mobile can wait — desktop first" | Responsive behavior affects layout structure. Deciding breakpoint behavior after building desktop creates rewrite, not adaptation. |
| "I'll just use the default component" | Default components produce generic-looking UI. Check existing design patterns first — consistency with the existing design system matters more than speed. |

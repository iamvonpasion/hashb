# hashb

AI-assisted development toolkit.

> Rules in `rules/` activate when Claude reads matching files.
> When generating new files, read an existing file of the same type first
> so that language-specific rules load into context.

## Skills

| Skill | Purpose | Modifies code? |
|-------|---------|----------------|
| `quick` | Quick planning — single-response plans, no ceremony, auto-detects intent or explicit mode | No |
| `spec` | Product specification — problem discovery, requirements definition, acceptance criteria | No |
| `design` | UX/UI design — wireframes, component mapping, interaction design, accessibility | No |
| `research` | Pre-planning research — libraries, APIs, ecosystem, stack choices | No |
| `eng` | Engineering — scope, architecture decisions (if needed), implementation review, TDD | No |
| `tdd` | Test-driven implementation — execute TDD plan from /eng, RED → GREEN → REFACTOR gates | Yes |
| `review` | Peer code review — agent-to-agent, confidence-scored, blocks ship on low confidence | No |
| `fix` | Systematic debugging with RCA | Yes |
| `qa` | QA testing — report only, never fixes | No |
| `retro` | Post-ship retrospective — what worked, what didn't, action items, memory persistence | No |
| `ship` | Ship workflow — test, review, version, changelog, PR | Yes |
| `swarm` | Parallel work decomposition — plan, review, coordinate, merge streams | Yes |
| `docs` | Document hygiene — find stale, obsolete, duplicate, scattered docs; sync, consolidate, update, remove | Yes |
| `audit` | Compliance audit — scan entire repo against hashb standards, scored report | No |
| `init` | Bootstrap or update a repo for hashb compliance — Project Profile, rules, scaffolds | Yes |

See `recipes.md` for workflow chains (feature, bugfix, release, architecture).

## Rules

| Category | Rules | Purpose |
|----------|-------|---------|
| Generic | `security`, `testing`, `git`, `docs`, `context`, `versioning`, `shipping`, `guard` | OWASP/CWE security, testing practices, git conventions, doc formatting, token optimization & on-demand loading, plugin versioning, self-shipping, destructive command warnings |
| Domain | Add per project in `rules/business/<domain>/` | Business rules, architecture patterns specific to your project |
| Infrastructure | `infrastructure/` (scaffold) | CI/CD, deployment, monitoring — rules added as patterns emerge |

## Project Profile

> **For consumers:** Add this section to your project's `CLAUDE.md` so skills
> can adapt to your stack. Skills read these fields to adjust review checks,
> research sources, and scaffolding patterns.

```markdown
## Project Profile
- Stack: [e.g., Next.js 15 (App Router), TypeScript, PostgreSQL]
- Architecture: [e.g., Modular monolith, microservices, monorepo]
- Tenancy: [e.g., Single-tenant, multi-tenant (tenant_id scoping)]
- Testing: [e.g., Vitest + Playwright, Jest + Cypress]
- Deploy: [e.g., Vercel, AWS ECS, Docker Compose]
- MCP Servers: Context7 (library docs) [+ optional: internal-wiki (company KB)]
```

Fields are optional except MCP Servers, which must include Context7 (live library docs).
Skills use sensible defaults when other fields are missing.

## Skill Routing

> **For consumers:** Add this section to your project's `CLAUDE.md` so Claude
> invokes the right skill even from natural language (without explicit `/` commands).
> `/init` scaffolds this automatically.

```markdown
## Skill Routing
When user intent matches a skill, invoke it instead of acting freestyle:
- Quick plan, "just plan this", fast, lightweight, small task → `/hashb:quick`
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

## Rule Overrides

> **For consumers:** Override a generic rule for specific paths when your project
> needs different behavior. Create a rule file with `overrides:` in frontmatter.

```yaml
---
description: Relaxed auth rules for internal admin tools
overrides: security
paths:
  - "src/admin/**/*.ts"
---

# Override: Admin Security

Admin tools are internal-only and behind VPN. These relaxations apply:
- CSRF tokens not required (internal tool, not public-facing)
- Rate limiting relaxed (low traffic, trusted users)

All other security rules still apply.
```

The `overrides` field references a rule by its `id` (e.g., `security`, `testing`).
For the overridden paths, the override rule's guidance takes precedence.
Rules without overrides still apply normally.

# hashb

Dev workflow toolkit for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).
14 skills + domain-specific rules organized by vertical.

## Install

**As a plugin (recommended):**
```bash
/plugin marketplace add hashb/hashb-workflow
/plugin install hashb
```

**Manual (copy into your project):**
```bash
git clone https://github.com/hashb/hashb-workflow
cp -r hashb-workflow/skills your-project/.claude/skills
cp -r hashb-workflow/rules your-project/.claude/rules
```

## Skills

All skills are generic — they work for any project. Domain context comes from rules.

| Skill | What it does | Code? |
|-------|-------------|-------|
| `hashb:spec` | Product specification — problem discovery, requirements, acceptance criteria | No |
| `hashb:design` | UX/UI design — wireframes, components, interactions, accessibility | No |
| `hashb:research` | Pre-planning research — libraries, APIs, ecosystem, stack choices | No |
| `hashb:eng` | Engineering — scope, architecture decisions, implementation review, TDD | No |
| `hashb:review` | Peer code review — agent-to-agent, confidence-scored verdicts | No |
| `hashb:fix` | Systematic debugging — evidence → root cause → fix → verify | Yes |
| `hashb:qa` | QA testing — report only, never fixes | No |
| `hashb:retro` | Post-ship retrospective — findings, action items, memory persistence | No |
| `hashb:ship` | Ship — merge, test, review, version, changelog, PR | Yes |
| `hashb:swarm` | Parallel work — decompose, review, coordinate, merge streams | Yes |
| `hashb:audit` | Compliance audit — scan repo against hashb standards, scored report | No |
| `hashb:init` | Bootstrap or update repo for hashb compliance — Profile, rules, scaffolds | Yes |

### When to use which

```
"What should we build?"                 → /hashb:spec
"Design the UI for this feature"        → /hashb:design
"I need to evaluate libraries"          → /hashb:research
"Design the architecture"               → /hashb:eng arch
"Review my implementation plan"          → /hashb:eng (or /hashb:eng impl)
"Review my code before shipping"         → /hashb:review
"Something is broken"                    → /hashb:fix
"Test this feature"                      → /hashb:qa
"Ship it"                                → /hashb:ship
"What went well? What didn't?"           → /hashb:retro
"This is too big for one stream"         → /hashb:swarm plan
"Set up this repo for hashb"            → /hashb:init
"How compliant is this repo?"           → /hashb:audit
```

## Workflow Recipes

See `recipes.md` for predefined skill chains:

```
Feature       →  spec → design (if UI) → eng → tdd → review → qa → fix loop → ship → retro
Architecture  →  research → eng arch → iterate decisions → ADRs
Bugfix        →  fix → review → qa → ship → retro
Release       →  review → qa → fix loop → ship → retro
```

## Rules

Rules activate automatically on matching files. No configuration needed.

| Category | Rules |
|----------|-------|
| **Generic** | `security`, `testing`, `git`, `docs`, `versioning`, `shipping`, `guard` |
| **Domain** | Add per project — e.g., `business/billing/`, `business/scheduling/` |
| **Infrastructure** | `infrastructure/` (scaffold) |

### Adding Domain Rules

1. Create `rules/business/<your-domain>/` in your project
2. Add `.md` files with `paths:` frontmatter matching your source files
3. Rules activate automatically when Claude reads matching files

### Rule Overrides

Override generic rules for specific paths using `overrides:` in frontmatter.
See `CLAUDE.md` for details.

## Project Profile

Add a Project Profile to your project's `CLAUDE.md` so skills adapt to your stack:

```markdown
## Project Profile
- Stack: Next.js 15 (App Router), TypeScript, PostgreSQL
- Architecture: Modular monolith
- Testing: Vitest + Playwright
- Deploy: Vercel
- MCP Servers: Context7 (library docs)
```

## Project Structure

```
skills/                          # 14 skills
  spec/ design/ research/ eng/
  tdd/ review/ fix/ qa/ retro/
  ship/ swarm/ docs/ audit/ init/
rules/                           # Organized by concern
  security.md                    # OWASP/CWE (3 tiers)
  testing.md                     # Testing practices
  git.md                         # Commits, branches, PRs
  docs.md                        # Documentation formatting
  versioning.md                  # Plugin version bumping
  shipping.md                    # Commit and push workflow
  guard.md                       # Destructive command warnings
  context.md                     # Token optimization, on-demand loading
  business/                      # Domain rules — add per project
  infrastructure/                # CI/CD, deploy, monitoring (scaffold)
recipes.md                       # Workflow chains (feature, bugfix, release, architecture)
CLAUDE.md                        # Toolkit configuration + consumer conventions
```

## License

MIT

```
  _               _     _
 | |__   __ _ ___| |__ | |__
 | '_ \ / _` / __| '_ \| '_ \
 | | | | (_| \__ \ | | | |_) |
 |_| |_|\__,_|___/_| |_|_.__/
```

**Dev workflow toolkit for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).**

16 skills that turn Claude into a structured development partner — from
spec to ship, with gates, reviews, and TDD enforcement at every step.

Skills are generic. They work for any stack, any project. Domain context
comes from rules you add.

---

## Why hashb

Most AI coding tools let you generate code. hashb makes you **ship reliable software.**

```
Without hashb                          With hashb
─────────────────────                  ─────────────────────
"Fix this bug"                         Evidence → root cause → regression test → fix → verify
"Build this feature"                   Spec → design → eng → TDD → review → QA → ship
"Review my code"                       6-category checklist, confidence-scored, blocks on low
"Ship it"                              Tests → review → version → changelog → PR (automated)
```

Every skill enforces discipline that AI tends to skip:
- **Root cause before fix.** No band-aid solutions. RED → GREEN proof required.
- **Tests before code.** TDD is mandatory, not optional.
- **Review before ship.** Confidence-scored. Low confidence = escalate to human.
- **Evidence over confidence.** "I'm confident this works" is not accepted without proof.

---

## Install

```bash
/install-plugin https://github.com/iamvonpasion/hashb
```

---

## Skills

```
  PLAN                    BUILD                   SHIP
  ─────────────────       ─────────────────       ─────────────────
  /spec    requirements    /tdd     test-first     /review  peer review
  /design  wireframes      /fix     debug + RCA    /qa      test as user
  /research evaluate       /swarm   parallel work  /ship    PR + version
  /eng     architecture                            /retro   retrospective
```

| Skill | What it does | Writes code? |
|-------|-------------|:------------:|
| `spec` | Product specification — discovery, requirements, acceptance criteria, task decomposition | * |
| `quick` | Quick planning — single-response, no ceremony, auto-detects intent | |
| `design` | UX/UI design — wireframes, components, interactions, accessibility | |
| `research` | Pre-planning research — libraries, APIs, ecosystem, stack choices | |
| `eng` | Engineering — scope, architecture decisions, implementation review, TDD plan | |
| `tdd` | Test-driven implementation — RED → GREEN → REFACTOR with checkpoints | Yes |
| `review` | Peer code review — confidence-scored, blocks ship on low confidence | |
| `fix` | Systematic debugging — evidence → root cause → regression test → fix | Yes |
| `qa` | QA testing — browse like a real user, report bugs, never fix | |
| `retro` | Post-ship retrospective — what worked, what didn't, action items | |
| `ship` | Ship — tests, review, version, changelog, commit, PR | Yes |
| `swarm` | Parallel work decomposition — plan, coordinate, merge streams | Yes |
| `docs` | Document hygiene — find stale, duplicate, scattered docs and clean up | Yes |
| `audit` | Compliance audit — scored report with prioritized findings | |
| `init` | Bootstrap a repo for hashb — Project Profile, rules, scaffolds | Yes |

### Quick reference

```
"What should we build?"          → /hashb:spec
"Break this into tasks"          → /hashb:spec decompose
"Just plan this quickly"         → /hashb:quick
"Design the UI"                  → /hashb:design
"Evaluate libraries"             → /hashb:research
"Plan the architecture"          → /hashb:eng arch
"Write the code (TDD)"          → /hashb:tdd
"Review before shipping"         → /hashb:review
"Something is broken"            → /hashb:fix
"Test this feature"              → /hashb:qa
"Ship it"                        → /hashb:ship
"What went well?"                → /hashb:retro
"Too big for one stream"         → /hashb:swarm plan
"Set up this repo"               → /hashb:init
"How compliant is this repo?"    → /hashb:audit
```

---

## Workflow Recipes

Chain skills together for common workflows. See [`recipes.md`](recipes.md) for details.

```
Feature    /spec → /design (if UI) → /eng → /tdd → /review → /qa → /fix loop → /ship → /retro
                   ──────────────────────────────────────────────────────────────────────────────
Architecture       /research → /eng arch → iterate decisions → ADRs
Bugfix             /fix → /review → /qa → /ship → /retro
Release            /review → /qa → /fix loop → /ship → /retro
```

Each recipe has gates between phases. Skills don't silently proceed — they stop
at decision points and escalate when confidence is low.

---

## Rules

Rules activate automatically when Claude reads matching files. No setup needed.

| Category | What it covers |
|----------|---------------|
| **Security** | OWASP/CWE — input validation, injection prevention, auth, secrets management |
| **Testing** | Behavior-driven tests, edge cases, AI-aware testing practices |
| **Git** | Branch naming, commit format, PR standards |
| **Docs** | Changelog format, TODO tracking, markdown conventions |
| **Context** | Token optimization, on-demand loading, CLAUDE.md hygiene |
| **Guard** | Warns before destructive commands (rm -rf, force push, DROP TABLE) |
| **Domain** | Add your own in `rules/business/` — scoped to your source files |

### Adding your own rules

```
rules/business/billing/
  validation.md       # paths: ["src/billing/**/*.ts"]
  compliance.md       # paths: ["src/billing/**/*.ts"]
```

Rules with `paths:` frontmatter only load when Claude reads matching files.
No wasted context on irrelevant rules.

---

## Project Profile

Add this to your project's `CLAUDE.md` so skills adapt to your stack:

```markdown
## Project Profile
- Stack: Next.js 15, TypeScript, PostgreSQL
- Architecture: Modular monolith
- Testing: Vitest + Playwright
- Deploy: Vercel
- MCP Servers: Context7 (library docs)
```

Skills read these fields to adjust review checks, research sources, and scaffolding.
All fields optional. Skills use sensible defaults when fields are missing.

---

## Project Structure

```
hashb/
├── skills/                    16 skills
│   ├── quick/                 Quick planning
│   ├── spec/                  Product specification
│   ├── design/                UX/UI design
│   ├── research/              Pre-planning research
│   ├── eng/                   Engineering & architecture
│   ├── tdd/                   Test-driven implementation
│   ├── review/                Peer code review
│   ├── fix/                   Debugging with RCA
│   ├── qa/                    QA testing
│   ├── retro/                 Retrospective
│   ├── ship/                  Ship workflow
│   ├── swarm/                 Parallel work
│   ├── docs/                  Document hygiene
│   ├── audit/                 Compliance audit
│   ├── init/                  Repo bootstrap
│   └── shared/                Shared formatting rules
├── rules/                     Auto-activating rules
│   ├── security.md            OWASP/CWE (3 tiers)
│   ├── testing.md             Testing practices
│   ├── git.md                 Git conventions
│   ├── docs.md                Documentation format
│   ├── context.md             Token optimization
│   ├── guard.md               Destructive command warnings
│   ├── versioning.md          Version bumping
│   ├── shipping.md            Commit & push workflow
│   └── business/              Your domain rules go here
├── recipes.md                 Workflow chains
├── CLAUDE.md                  Toolkit configuration
└── .claude-plugin/            Plugin manifest
```

---

## Author

**Von Pasion** — [vonpasion.com](https://vonpasion.com) · [LinkedIn](https://www.linkedin.com/in/von-pasion/)

## License

MIT

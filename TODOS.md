# TODOs

### Feature: Audit Fixes (2026-04-10)

> Implementing all 15 recommendations from AUDIT-REPORT.md v2.

- [x] P1 [S] #1 Expand security.md paths — add 11 missing file extensions (depends: none)
  - Goal: Security rules activate for all common source file types
  - AC:
    - [x] security.md paths includes .js, .jsx, .go, .java, .kt, .rb, .rs, .php, .swift, .vue, .svelte

- [x] P1 [S] #2 Add .history/ to .gitignore and stop staging in /ship (depends: none)
  - Goal: Artifact archives stay local-only, not committed to consumer repos
  - AC:
    - [x] .gitignore contains .history/ entry
    - [x] ship/SKILL.md changed from "Stage" to "Do NOT stage"

- [x] P2 [S] #3 Fix plugin.json skill count: 15 -> 16 (depends: none)
  - Goal: Plugin metadata accurately reflects actual skill count
  - AC:
    - [x] plugin.json description says "16 skills"

- [x] P2 [S] #4 Align versioning: remove MICRO from /ship (depends: none)
  - Goal: Ship uses standard semver consistently
  - AC:
    - [x] ship/SKILL.md version table has no MICRO row
    - [x] ship/SKILL.md changelog format uses [X.Y.Z] not [X.Y.Z.W]

- [x] P2 [S] #5 Add QA loop termination: max 3 cycles (depends: none)
  - Goal: QA/fix loop has explicit termination criteria
  - AC:
    - [x] recipes.md Testing pattern includes max 3 cycles
    - [x] qa/SKILL.md rules section includes max 3 cycles

- [x] P2 [S] #6 Split formatting.md into core + presentation (depends: none)
  - Goal: Reduce token overhead; skills load only functional rules by default
  - AC:
    - [x] formatting.md contains only functional rules
    - [x] formatting-presentation.md contains progress indicators + discussion chunking
    - [x] All 14 skill references updated

- [x] P2 [S] #7 Persist TDD checkpoints to disk (depends: none)
  - Goal: /tdd continue can resume after context compaction
  - AC:
    - [x] tdd/SKILL.md instructs writing .tdd-checkpoint-{branch}.json
    - [x] tdd/SKILL.md instructs reading checkpoint on continue
    - [x] tdd/SKILL.md instructs cleanup on completion

- [x] P2 [S] #8 Persist 3-strike state to disk (depends: none)
  - Goal: Fix skill's 3-strike hard stop survives context compaction
  - AC:
    - [x] fix/SKILL.md instructs writing .fix-state-{branch}.json
    - [x] fix/SKILL.md instructs reading state file at Phase 2 start
    - [x] fix/SKILL.md instructs cleanup on resolution

- [x] P2 [S] #9 Add retro memory read-back to /eng and /review (depends: none)
  - Goal: Learning loop closed — retro writes, eng/review reads
  - AC:
    - [x] eng/SKILL.md checks Claude Code memories for retro learnings
    - [x] review/SKILL.md checks Claude Code memories for retro learnings

- [x] P1 [M] #10 Scaffold guard hooks in /init (depends: none)
  - Goal: Destructive command protection enforced via Claude Code hooks
  - AC:
    - [x] init/SKILL.md Phase 4 scaffolds guard hooks
    - [x] init/references.md contains hook template and guard script
    - [x] init/SKILL.md Phase 5 verification includes hooks check

- [x] P1 [M] #11 Extract branch/base detection into shared/preflight.md (depends: none)
  - Goal: DRY — canonical detection block with gh availability check
  - AC:
    - [x] skills/shared/preflight.md exists with canonical bash block
    - [x] 7 skills use consistent detection with gh check
    - [x] Windows-incompatible patterns (tr, head -c) replaced in fix + tdd

- [x] P1 [M] #12 Add secret scanning to /ship pre-landing review (depends: none)
  - Goal: Ship blocks on detected secrets in the diff
  - AC:
    - [x] ship/SKILL.md Step 3.5 includes gitleaks/trufflehog detection
    - [x] Graceful degradation if no scanner installed

- [x] P2 [M] #13 Expand design handoff + eng Phase 0 warning (depends: none)
  - Goal: Design-to-eng handoff preserves component details across skill boundaries
  - AC:
    - [x] design/SKILL.md handoff includes component enumeration, error table, data flow
    - [x] eng/SKILL.md Phase 0 warns if design handoff is compressed

- [x] P2 [M] #14 Add dependency vulnerability scanning to /ship (depends: none)
  - Goal: Known CVEs caught before shipping
  - AC:
    - [x] ship/SKILL.md runs npm audit / pip audit / cargo audit / govulncheck
    - [x] Critical vulnerabilities block ship

- [x] P3 [L] #15 Add CI template scaffolding to /init (depends: #10)
  - Goal: New repos get a working CI pipeline from /init
  - AC:
    - [x] init/SKILL.md Phase 4 offers CI scaffolding
    - [x] init/references.md contains GitHub Actions templates for Node, Python, .NET, Go, Rust
    - [x] init/SKILL.md Phase 5 verification includes CI check

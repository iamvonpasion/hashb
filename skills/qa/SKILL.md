---
description: QA testing — browse like a real user, document bugs, report only. Never fixes.
---

# QA Report Only: Test & Document

Test like a real user. Document everything with screenshots.
**NEVER fix anything. NEVER read or edit source code.**

When bugs are found, use `/fix` to address them, then re-run `/qa` to verify.

---

## Presentation Rules

1. **Discussion chunking** — when a follow-up response would be too dense to digest in one shot, present a numbered big-picture overview first, then discuss each point one at a time, waiting for user input between points. Use judgment: chunk whenever the response feels like a wall of text, not at a fixed threshold.
2. **Progress indicator** — every output starts with:

```
/qa ═════════════════════════════════════════════════════════════

  ▸ Phase 1  Orient                ~2 min
  ○ Phase 2  Explore & Document    ~10 min
  ○ Phase 2.5 Requirement Verify   ~5 min
  ○ Phase 3  Score & Report        ~3 min

═════════════════════════════════════════════════════════════════
```

Update `▸` (current), `✓` (done), `○` (pending) as phases progress.
Completed phases show a status note (e.g., `✓ 8 pages`, `✓ 4 issues`).

---

## Setup

### Parameters

| Parameter | Default | Example |
|-----------|---------|---------|
| URL | auto-detect or required | `http://localhost:3000` |
| Mode | full | `--quick`, `--regression baseline.json` |
| Scope | Full app (or diff-scoped) | "Focus on billing page" |
| Auth | None | "Sign in as user@example.com" |

### Pre-flight

```bash
# Find a browser automation tool (Playwright, Puppeteer, or compatible CLI)
# Set $B to the browse command. Adjust the path to match your setup.
B=""
command -v npx >/dev/null && B="npx playwright"
[ -x "$B" ] && echo "READY: $B" || echo "NEEDS_SETUP: install a browser CLI (e.g. npx playwright install)"
```

If `NEEDS_SETUP`: ask the user to install a browser automation tool.

```bash
mkdir -p .qa-reports/screenshots
```

### Mode Detection

**No URL + feature branch →** diff-aware mode:

```bash
BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo "main")
git diff $BASE...HEAD --name-only
git log $BASE..HEAD --oneline
```

Map changed files → affected pages/routes. Detect local app on common
ports. Test each affected page. If no pages identified, fall back to
Quick mode (homepage + top 5 nav targets).

**URL provided →** full mode.

---

## Phase 1: Orient

```bash
$B goto <target-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/initial.png"
$B links
$B console --errors
```

Map: navigation structure, framework, key pages. Note landing errors.

For SPAs: `links` misses client-side routes — use `snapshot -i` for
nav elements instead.

---

## Phase 2: Explore & Document

Visit pages systematically. At each page:

```bash
$B goto <page-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page-name.png"
$B console --errors
```

**Per-page checklist:**
- Visual scan — layout issues in screenshot
- Interactive elements — click buttons, links, controls
- Forms — empty, invalid, edge-case inputs
- States — empty, loading, error, overflow
- Console — new JS errors after interactions?
- Mobile — `$B viewport 375x812` if relevant

**Quick mode:** Homepage + top 5 nav targets only. Just check: loads?
Console errors? Broken links?

**Depth over breadth.** 5-10 well-evidenced issues > 20 vague ones.

### Evidence

**Interactive bugs** (broken flows, dead buttons):

```bash
$B screenshot "$REPORT_DIR/screenshots/issue-001-before.png"
$B click @e5
$B screenshot "$REPORT_DIR/screenshots/issue-001-after.png"
$B snapshot -D
```

**Static bugs** (typos, layout): single annotated screenshot.

Document each issue **immediately** — don't batch.
After every screenshot, use the Read tool so the user sees it inline.

---

## Phase 2.5: Requirement Verification

Before scoring, verify that the original goals were actually met — not just
that code runs without errors.

**Step 1 — Gather requirements.** Read the original user request, feature
description, or ticket. If running inside `/workflow`, use the requirement
list from the orchestrator handoff. Extract each distinct requirement or
acceptance criterion.

**Step 2 — Verify each requirement.** For each one, check if the behavior
is observable in the running app:

- **PASS** — behavior confirmed through browser interaction
- **FAIL** — behavior missing or broken
- **PARTIAL** — partially works (note what's missing)
- **NOT_TESTABLE** — can't verify via browser (needs load test, background job, etc.)

**Step 3 — Integration check.** For multi-module changes:
- Are new exports actually imported by consumers?
- Do new API endpoints have callers?
- Do new events have handlers?
- Does the full workflow complete end-to-end without breaks?

**Output — append to report:**

```
REQUIREMENT VERIFICATION:
  PASS        {description}    — {how verified}
  FAIL        {description}    — {what's missing}
  PARTIAL     {description}    — {what works, what doesn't}
  NOT_TESTABLE {description}   — {why, what would be needed}

Coverage: {N} PASS / {N} FAIL / {N} PARTIAL / {N} NOT_TESTABLE
```

If any requirement is FAIL, it appears in the "TOP 3 TO FIX" section
regardless of traditional severity scoring.

---

## Phase 3: Score & Report

### Health Score

Categories start at 100. Deduct per finding:
Critical −25, High −15, Medium −8, Low −3. Minimum 0.

| Category | Weight |
|----------|--------|
| Functional | 20% |
| Console | 15% |
| Accessibility | 15% |
| UX | 15% |
| Visual | 10% |
| Links | 10% |
| Performance | 10% |
| Content | 5% |

Console scoring: 0 errors → 100, 1-3 → 70, 4-10 → 40, 10+ → 10.
Links: each broken link → −15.

### Report

Write to `.qa-reports/qa-report-{domain}-{date}.md`:

```
✓ QA REPORT ─────────────────────────────────────────────────────

  Domain         {domain}
  Date           {date}
  Branch         {branch}
  Pages tested   {N}
  Duration       {time}
  Health score   {score}/100

  ISSUES ({N} total)
  ─────────────────────────────────────────────────
  Critical       {N}
  High           {N}
  Medium         {N}
  Low            {N}

  TOP 3 TO FIX
  ─────────────────────────────────────────────────
  1.  {highest severity}
  2.  ...
  3.  ...

─────────────────────────────────────────────────────────────────
```

### Next Step

| Result | Next Skill | Why |
|--------|-----------|-----|
| Clean (no critical/high issues) | `/ship` | Ready to ship |
| Issues found | `/fix` → re-run `/qa` | Fix bugs, then verify (max 3 cycles) |
| Requirements FAIL | `/fix` (with requirement context) | Missing functionality, not just bugs |

**Autonomous mode:** If running inside a recipe chain:
- Clean report → auto-proceed to `/ship`
- Issues found → auto-invoke `/fix` with the top issues, then re-run `/qa` (max 3 cycles)
- After 3 cycles with remaining issues → escalate to user

Per-issue: severity, category, repro steps, screenshot references.

Save `baseline.json` for future regression runs.

**Regression mode:** Load prior baseline, diff health scores, list
fixed vs new issues, append comparison to report.

---

## Rules

- **NEVER fix anything.** Document only. Do not read source, edit files,
  or suggest code fixes. Use `/qa` for the fix loop.
- **Screenshots are mandatory.** Every issue needs at least one.
- **Verify before documenting.** Retry once to confirm it's reproducible.
- **Never include credentials.** Write `[REDACTED]` for passwords.
- **Check console after every interaction.**
- **Test as a user.** Walk through complete workflows end-to-end.
- **Show screenshots to the user.** Use the Read tool after every
  screenshot so the user sees them inline.
- **Never refuse to use the browser.** Even if changes look backend-only,
  open the browser and verify.


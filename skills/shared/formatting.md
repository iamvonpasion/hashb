# Shared Formatting Rules

These rules apply to all skills. Each skill references this file rather than
duplicating formatting instructions.

---

## Progress Indicator

Every skill output starts with a progress bar showing phases:

```
/skill-name ════════════════════════════════════════════════════════

  ▸ Phase 1  [name]       ~N min
  ○ Phase 2  [name]       ~N min
  ○ Phase 3  [name]       ~N min

════════════════════════════════════════════════════════════════════
```

Update markers as phases progress:
- `▸` current phase
- `✓` completed (add status note, e.g., `✓ 3 findings`)
- `○` pending
- `—` skipped

---

## Discussion Chunking

When a follow-up response would be too dense to digest in one shot, present
a numbered big-picture overview first, then discuss each point one at a time,
waiting for user input between points.

Use judgment: chunk whenever the response feels like a wall of text, not at
a fixed threshold. Short responses don't need chunking.

---

## Tables

- Tables must start at column 0 — never indent tables inside card blocks
- Use markdown tables for structured comparisons
- Keep column count reasonable (3-5 columns max)

---

## Code Blocks

- Always specify language for syntax highlighting
- Keep blocks focused — show the relevant snippet, not the whole file

---

## Output Style

- Lead with the answer or action, not the reasoning
- Use tables for structured data, prose for context
- One sentence per line in structured sections (easier diffs)

---

## Workflow Discipline

After completing any skill that has a defined "Next Step" table:

- **Follow the Next Step table.** Do not suggest alternatives not listed there.
- **Never prompt "ready to ship/commit?"** unless the current skill's Next Step
  explicitly points to `/ship`.
- **Autonomous mode:** Proceed directly to the next step. No confirmation needed.
- **Interactive mode:** State the next step as a recommendation
  (e.g., "Review complete. Next step is /qa.") rather than offering open-ended
  alternatives like "Want me to ship?"
- The valid statement after implementation is: "Tests pass. Running /review."
  — not "Tests pass. Ready to ship?"

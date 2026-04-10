# Shared Formatting Rules

These rules apply to all skills. Each skill references this file rather than
duplicating formatting instructions.

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

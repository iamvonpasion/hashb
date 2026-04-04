---
id: guard
description: >
  Safety guardrails — warns before destructive commands. Applies to all bash
  execution across all skills. User can override per-command.
---

# Guard Rules

Check every bash command against these patterns before execution.
If matched (and not a safe exception), warn the user with the specific risk
and ask to proceed or cancel.

## Protected Patterns

| Pattern | Risk |
|---------|------|
| `rm -rf` / `rm -r` | Recursive delete — may destroy data irreversibly |
| `DROP TABLE` / `DROP DATABASE` | Data loss — no undo without backup |
| `TRUNCATE` | Data loss — removes all rows without logging |
| `git push --force` / `-f` | History rewrite — may overwrite teammates' work |
| `git reset --hard` | Uncommitted work loss |
| `git checkout .` / `git restore .` | Uncommitted work loss |
| `kubectl delete` | Production impact — may take down running services |
| `docker rm -f` / `docker system prune` | Container/image loss |

## Safe Exceptions

These targets are allowed without warning — they are build artifacts or caches:

`node_modules`, `.next`, `dist`, `__pycache__`, `.cache`, `build`,
`.turbo`, `coverage`

## Warning Format

When a protected pattern is matched:

```
WARNING: This command matches a destructive pattern.
  Command:  {the command}
  Pattern:  {which pattern matched}
  Risk:     {what could go wrong}

Proceed? (yes/no)
```

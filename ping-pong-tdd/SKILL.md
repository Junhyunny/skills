# ⚠️ This skill has been split into three focused skills

The `ping-pong-tdd` skill has been refactored. Use these instead:

| Skill | Purpose |
|-------|---------|
| `/tdd-plan` | Load story, detect stack & conventions, plan tasks → writes `.tdd-session.md` |
| `/tdd-task` | Run one task's RED-GREEN-REFACTOR cycle → updates `.tdd-session.md` |
| `/tdd-commit` | Summarize session, commit changes, clean up `.tdd-session.md` |

## Typical flow

```
/tdd-plan 12345678         # or paste story content
/tdd-task                  # repeat for each task
/tdd-task
/tdd-task
/tdd-commit
```

## Reference files (still valid, used by the new skills)

- `references/tdd-test-writing-guide.md`
- `references/tech-stack-detection.md`
- `references/convention-detection.md`
- `references/mcp-story-fetching.md`
- `references/red-green-refactor-guide.md`
- `references/commit-conventions.md`

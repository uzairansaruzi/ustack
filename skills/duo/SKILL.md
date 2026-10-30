---
name: duo
description: Use when the user says "duo with" followed by a model name.
---

# duo

Do the user's task yourself while a partner model does the same task
independently in parallel. Then check the partner's work and give the user
one merged answer. Two independent passes catch what one misses.

## 1. Start the partner

The user names the partner model ("duo with sol 6.1"). In T3, call
`orchestrator_capabilities` and resolve that model. Do not hardcode an ID.
If the user named no model, ask. If the named model is unavailable, report
the blocker. Do not substitute a model.

Start it with `delegate_task` (`mode: "async"`) before you begin your own
work. Save `taskId`. If the task edits files, give the partner its own
worktree (`isolation: "worktree"`) so you do not collide. Outside T3, use a
native subagent that runs the named model.

Brief (T3 children get no history). Send the task, not your hypothesis:

```
Do this task independently. Do not commit, push, comment on PRs, or start subagents.
Repo: <abs path> Task: <user's request, verbatim> Context: <facts the task needs>
Reply: findings or changes, each with evidence (file:line, command output).
Mark what you verified and what you inferred.
```

## 2. Do the task yourself

Work the task fully, as if no partner existed. Do not read the partner's
result until your own pass is done, so you stay independent. Wait for the
completion notice. Do not poll. Timeout, error, or no reply: say so and
present your own result.

## 3. Review and merge

Check each partner claim against the source yourself. Sort:

- Both found: higher confidence.
- Partner only: verify; keep it if it holds, drop it with a reason if not.
- You only: re-check it, since the partner missed it.
- Conflict: settle it with evidence. If you cannot, show both sides.

For edits, keep one result: yours, the partner's, or yours with the
partner's better parts grafted in. Leave the partner's worktree untouched.

## 4. Report

One merged answer, not two reports. Then a short duo note: partner model,
what it added, what you rejected and why, and any open disagreement for the
user to decide.

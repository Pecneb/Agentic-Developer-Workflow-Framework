# Context Resume Policy

## On Restart or Interruption
- After a restart, compaction, interruption, or long-running task, identify the
  latest user request before continuing.
- Check current git state and relevant filesystem state before resuming.
- Do not continue an old plan blindly if the repo or request has moved.
- Report uncertainty when context is incomplete or stale.

## Resume Checklist
1. Reconfirm the active task.
2. Re-read the files you are about to rely on.
3. Recheck pending verifications or partially applied edits.
4. Continue only if the current state still matches the intended scope.

# Git Safety Policy

## Baseline Rules
- Inspect `git status` before edits.
- Never revert user changes unless explicitly requested.
- Do not commit or push unless asked.
- Avoid interactive git commands when a non-interactive alternative exists.
- Do not use destructive commands such as `git reset --hard` or
  `git checkout --` without explicit approval.

## Branching
- If the repo is on a default branch and the user asks for a commit, propose a
  branch first unless repo convention already says otherwise.

## Review Before Action
- Read diffs before describing them.
- Keep commits scoped to the approved task.
- Treat untracked or modified files outside your scope as user-owned until
  proven otherwise.

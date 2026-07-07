# Shell Risk Levels

## Level 0: Safe Read-Only
Examples:
- `pwd`
- `ls`
- `rg`
- `git status`
- `git diff`
- reading files

Rule:
- Allowed by default.

## Level 1: Local Targeted Verification
Examples:
- one targeted test
- `python -m py_compile` on touched files
- lint one file
- typecheck one touched module

Rule:
- Allowed when relevant to the task.

## Level 2: Expensive Local Work
Examples:
- full test suite
- full build
- Docker build
- large data processing

Rule:
- Requires approval unless explicitly requested by the user.

## Level 3: State-Changing Local Work
Examples:
- dependency install
- lockfile update
- migration generation
- formatting the whole repo

Rule:
- Requires explicit approval and a short plan.

## Level 4: Destructive or External
Examples:
- deploy
- push
- publish
- delete files
- overwrite artifacts
- run migrations against shared systems

Rule:
- Requires explicit approval, a risk note, and a rollback or mitigation plan.

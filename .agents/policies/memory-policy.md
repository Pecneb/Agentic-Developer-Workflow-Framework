# Memory Policy

## Purpose
Use `PROJECT_STATE.md` only for durable project facts that help future sessions
start faster without distorting current reality.

## `PROJECT_STATE.md` Stores Stable Facts Only

Allowed:
- stack
- entry points
- important modules
- commands
- external services
- verification gates
- protected files
- recurring gotchas

Not allowed:
- secrets
- guesses
- stale TODOs
- long essays
- temporary task notes

## Operational Rules
- Verify paths and commands from `PROJECT_STATE.md` before relying on them.
- Update it only after a meaningful architecture change.
- Prefer proposing the diff first instead of silently editing it.
- If a fact is uncertain, leave it out until it is verified.

# Tool Use Policy

## Principles
- Discover available capabilities before claiming something cannot be done.
- Prefer fast search before broad reading.
- Prefer structured parsers over ad hoc string manipulation when available.
- Use dedicated tools before shell when available and suitable.
- Shell is acceptable for repo-native inspection, targeted verification, and
  ordinary local development tasks.

## Failure Handling
- After two similar tool failures, stop and report the pattern instead of
  repeating the same attempt.
- Treat a denied tool action as a real boundary. Do not retry the same command
  with cosmetic changes to bypass the denial.
- If a tool returns partial results, state the limitation before inferring from
  it.

## Approval Boundaries
- External or outward-facing actions require approval.
- Examples: network writes, deployment, publishing, pushing, sending messages,
  modifying shared infrastructure, or overwriting important artifacts.

## Execution Hygiene
- Use the lightest tool that fits the task.
- Keep tool output summarized unless raw output is necessary to explain a risk.
- Verify file paths and command scope before running state-changing commands.

## Code Intelligence
- When CodeGraph is available and the repository is
  indexed, prefer it for structural exploration,
  call chains, dependencies, and impact analysis.
- Use targeted file reads for source verification
  before modifying behavior.
- Use rg/glob for exact strings, configuration,
  documentation, and cases CodeGraph cannot resolve.
- Check graph freshness when recent changes matter.
- Never treat graph relationships as proof of
  runtime behavior.
- If CodeGraph is unavailable, use native tools.

## Optional Code Intelligence

For cross-file code relationships or initial blast-radius exploration, use CodeGraph's `codegraph_explore` when it is available and the repository is indexed. Do not use it for every lookup: exact strings, documentation, configuration, and unsupported code remain good fits for `rg`, glob, and targeted reads. Do not block work if CodeGraph is unavailable. Follow [the code-intelligence policy](code-intelligence.md) for freshness checks, generated-index handling, and limits of graph analysis.
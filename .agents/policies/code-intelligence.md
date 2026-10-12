# Code Intelligence (Optional CodeGraph)

CodeGraph is an **optional** local code-structure index, not an authoritative record of runtime behavior and not durable agent memory. Do not install or initialize it automatically without the user's or project's permission.

## Choosing tools

1. If an indexed project and CodeGraph are available, prefer the MCP `codegraph_explore` tool for symbol relationships, call paths, dependencies, cross-file code exploration, and initial change-impact analysis. The `codegraph explore` CLI provides equivalent exploration when MCP is unavailable.
2. Use `rg`/glob and focused file reads for exact strings, documentation, configuration, schemas, migrations, generated artifacts, unsupported languages, and any question whose result does not depend on call relationships.
3. Treat graph-provided source as retrieved code: don't re-read every returned file by default. Check specific source locations when graph output is ambiguous, recently changed, or conflicts with observable behavior.
4. Graph edges are static approximations; dynamic dispatch, dependency injection, reflection, runtime configuration, and inter-service calls may be incomplete. Check downstream consumers, tests, and runtime behavior as needed.
5. Inspect the tool's freshness/staleness indicators after changes; if uncertain, use `codegraph status` or `codegraph sync`. If CodeGraph is absent, unindexed, stale, or fails, continue with built-in repository tools rather than blocking the task.
6. During nontrivial changes or review, use graph-based impact analysis to generate a **candidate** affected-file/test list, not a complete or verified blast radius. Run tests and report what actually passed.
7. Don't commit `.codegraph/` or `.codegraph-wsl/` indexes; these are generated, machine-local artifacts. Keep `PROJECT_STATE.md` as the source for stable project decisions and the framework's memory policy unchanged.

## Operational safety

- Setup is initiated by a developer, not implicitly by a task: `codegraph install` can change MCP configuration and agent instruction files, including `AGENTS.md` / `CLAUDE.md`; inspect those diffs before accepting them.
- Keep CodeGraph optional in all prompt packs and workflows. Do not require a graph for simple fixes or block tasks while indexing.
- Prefer project-local indexing and make sure repository ignore rules cover the generated index. Avoid putting any project secrets or proprietary source into external services; verify telemetry and deployment settings against project policy.

See [`agent-setup-guide.md`](../../agent-setup-guide.md#optional-codegraph-setup) for installation and diagnostic commands.
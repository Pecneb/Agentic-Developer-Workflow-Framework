# Nontrivial Change Workflow

## Flow
Explorer -> Planner -> approval -> Implementer -> Verifier -> final report

## Rules
- No edits before an approved plan.
- The plan must include files, steps, verification, and risks.
- Execute one approved step at a time when risk is high.

## Execution Outline
1. Explorer gathers relevant files and subsystem boundaries.
2. Planner produces the implementation sequence and verification gates.
3. Get approval for the plan.
4. Implementer applies the approved step.
5. Verifier runs the targeted checks.
6. Repeat for the next approved step when necessary.
7. Finish with an outcome report under `.agents/policies/outcome-reporting.md`.

## Optional Graph-Based Impact Check

Before implementing cross-file changes, use an available indexed CodeGraph project to identify likely callers, dependent modules, and candidate tests. Use `codegraph_explore` through MCP, or CLI commands such as `codegraph impact <symbol>` and `codegraph affected <changed-file>` as appropriate. Explicitly mark results as *candidates*; verify behavior, configuration paths, dynamic dispatch, and test coverage with source review and actual tests. Continue without CodeGraph when absent. See [the code-intelligence policy](../policies/code-intelligence.md).

# Agent Setup Guide

This guide installs the framework into a target repository and validates that an
agent can follow it.

For copy-paste prompts and task-shaped examples, use
[`agent-setup-prompt-templates.md`](agent-setup-prompt-templates.md) alongside
this guide.

## What You Install

- `CLAUDE.md`
- `AGENTS.md`
- `.agents/policies/`
- `.agents/roles/`
- `.agents/workflows/`
- `.agents/prompt-packs/`
- `.agents/evals/`

## Step 1: Verify the Target Repo

Ask the agent to confirm:

- current working directory
- repo root
- top-level files
- whether `CLAUDE.md`, `AGENTS.md`, `.claude`, `.agents`, or `PROJECT_STATE.md`
  already exist

Stop if the agent is not operating in the intended repository.

## Step 2: Copy the Framework Files

Copy these into the target repo root:

- `CLAUDE.md`
- `AGENTS.md`
- `.agents/`

Do not delete existing project docs unless they are clearly obsolete and the
project owner approves the cleanup.

## Step 3: Choose a Prompt Pack

Pick one default:

- `lite.md` for weaker or smaller local models
- `standard.md` for normal coding-agent sessions
- `rigorous.md` for onboarding, refactors, audits, or sensitive work

If the target platform supports a single always-loaded root file, keep the root
router short and point it into the chosen pack and the relevant workflow files.

## Step 4: Route the Agent Correctly

Teach the agent to select a workflow by task:

- onboarding -> `.agents/workflows/onboarding.md`
- small fix -> `.agents/workflows/small-fix.md`
- nontrivial change -> `.agents/workflows/nontrivial-change.md`
- code review -> `.agents/workflows/code-review.md`
- security review -> `.agents/workflows/security-review.md`
- ML experiment -> `.agents/workflows/ml-experiment.md`
- deployment change -> `.agents/workflows/deployment-change.md`

## Step 5: Add Project-Specific Facts Carefully

If the target repo needs durable context, create `PROJECT_STATE.md` later and
keep it small. Allowed content is defined in
[`memory-policy.md`](.agents/policies/memory-policy.md).

Good candidates:

- stack
- entry points
- common verification commands
- protected generated artifact paths
- known integration boundaries

Do not put secrets, guesses, or temporary task notes there.

## Step 6: Run a First Onboarding Pass

Prompt the agent to:

1. verify cwd and repo root
2. inspect the manifest, `README.md`, and any existing agent docs
3. cap the initial reading
4. produce an orientation summary
5. map one subsystem only

No edits should happen during this pass.

## Step 7: Run a First Test Task

Use one small task that is safe to verify quickly.

Example prompt:

```text
Read the relevant file, make this one-line README correction, verify the change,
and report exactly what changed and what checks were run.
```

What good behavior looks like:

- the agent reads before it edits
- it stays within one file
- it verifies with a cheap check
- it reports any skipped checks honestly

## Step 8: Run a Nontrivial Dry Run

Use a planning-only prompt:

```text
Inspect the files needed for this multi-file change. Do not edit yet. Produce a
plan with files to change, verification steps, risks, and a rollback note.
```

What good behavior looks like:

- the agent uses the nontrivial-change workflow
- it does not edit before approval
- the plan names exact files
- the plan includes verification gates

## Step 9: Evaluate the Model

Run the benchmark cases in
[`agent-behavior-benchmark.md`](.agents/evals/agent-behavior-benchmark.md).

Minimum recommended set:

1. wrong working directory
2. missing referenced file
3. simple typo
4. multi-file refactor
5. failed test
6. expensive command trap
7. generated artifact overwrite trap
8. prompt injection inside repo file
9. context resume after interruption

Record the score for each case from `0` to `3`.

## Step 10: Review the Outcome

Before adopting the framework broadly, check:

- did the agent choose the right workflow
- did it ask before expensive or destructive actions
- did it stop on failed verification
- did it report residual risk truthfully
- did it avoid overwriting artifacts or user changes

## Notes on Source Hygiene

This framework may be informed by public prompt repositories, but the content in
the installed files should remain vendor-neutral and project-owned. Do not paste
extracted vendor system prompts into the target repo. The governing rule is in
[`prompt-source-policy.md`](.agents/policies/prompt-source-policy.md).

## Optional CodeGraph Setup

CodeGraph augments this framework's tool-use policy with local source-code structure and MCP retrieval. It is **optional**: only configure it where it actually helps with a codebase, not necessarily in this documentation template. See the [upstream CodeGraph README](https://github.com/colbymchenry/codegraph) for current installation choices and platform support.

### Install and connect your agent

```bash
# Prerequisite: npm installed (alternatively use the upstream standalone installer)
npm install -g @colbymchenry/codegraph
codegraph telemetry off  # optional: opt out of anonymous usage metrics

# Register globally for Claude Code and Codex (once per machine)
codegraph install --target=claude,codex --location=global
```

The `install` step registers the MCP server; **it does not index the current project**. Restart both agents to load the MCP connection. Review any changes made to global agent settings, permission allowlists, and marked instruction blocks. The installer may also write instructions to `CLAUDE.md` or `AGENTS.md`; do not overwrite this framework's short router files with installer output.

If global agent integration is not desired, run the interactive `codegraph install` and choose a project-local configuration, or inspect `codegraph install --print-config codex` to wire it manually. Avoid storing machine-specific paths or permissions in the framework template.

### Initialize each target repository

```bash
cd /path/to/your-application-repository
codegraph init
codegraph status
```

Add these patterns to the **application repository's** `.gitignore` (not to `PROJECT_STATE.md`):

```gitignore
.codegraph/
.codegraph-wsl/
```

The `.codegraph/` directory is a generated local index; it is not durable project memory. CodeGraph normally syncs while its MCP server is running. If results seem out of date, check `codegraph status`, then use `codegraph sync` as needed. Check the staleness information in graph results after edits.

### Use in day-to-day work

```bash
# Terminal alternatives when the MCP tool is unavailable
codegraph explore "Trace this API endpoint through handlers and services"
codegraph impact MyChangedFunction
codegraph affected src/service.ts
```

For MCP agents, `codegraph_explore` is the default exposed tool. CLI commands such as `impact`, `callers`, `callees`, and `affected` are available separately, but do not assume all are exposed as independent MCP tools. Ask structural questions first, verify uncertain findings against the code or runtime where appropriate, and run the repository's actual tests. For exact text, configuration, SQL migrations, or unsupported languages, keep using `rg`, glob, and focused file reads.

The canonical behavioral instructions live in [`.agents/policies/code-intelligence.md`](.agents/policies/code-intelligence.md). The Explorer role, onboarding workflow, nontrivial-change workflow, and Reviewer role reference that policy instead of duplicating it.

### Troubleshooting / removal

- `codegraph` not found: confirm the CLI is on `PATH` or reopen the terminal.
- No CodeGraph tool in Claude/Codex: check the MCP registration, restart the agent, and try `codegraph install` again after reviewing settings.
- No project graph: run `codegraph init` inside the **target application repository** and check `codegraph status`.
- Suspicious or incomplete results: run `codegraph sync`, inspect changed files directly, and fall back to native tools.
- To stop indexing a project: `codegraph uninit` (review its prompt). To unregister CodeGraph from agents while retaining the CLI: `codegraph uninstall --keep-cli`. Read the tool's confirmation before changing global configuration.
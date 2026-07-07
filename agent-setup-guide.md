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

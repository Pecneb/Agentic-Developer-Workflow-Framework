# Agent Setup Prompt Templates

This document is a companion to
[`agent-setup-guide.md`](agent-setup-guide.md). Use it when you want copy-paste
prompts for onboarding an agent into an existing project and routing it into the
right workflow.

## How to Use This Document

- Start with the onboarding prompts.
- Use planning prompts before multi-file or risky changes.
- Use execution prompts only after the agent has enough context.
- Prefer naming the workflow explicitly when the task is not trivial.

## 1. Initial Repo Anchoring

Use this when the project already exists and you want the agent to orient itself
without editing anything.

```text
We are working in an existing project. Do not edit anything yet.

1. Confirm the current working directory and repo root.
2. List the top-level files and folders.
3. Check whether CLAUDE.md, AGENTS.md, PROJECT_STATE.md, .claude, or .agents already exist.
4. Read the README and any existing agent or architecture docs that look relevant.
5. Give me a short onboarding summary:
   - stack
   - entry points
   - existing agent docs
   - what should be preserved
   - what is missing for this framework

Stop after the summary.
```

Use this when:
- starting a new session in an old repo
- verifying the agent is anchored in the correct repository
- deciding whether the framework should be merged or installed fresh

## 2. Framework Merge Prompt

Use this when you want the framework added to an ongoing project without
clobbering existing project docs.

```text
Install this agent framework into the current repository as a merge, not as a fresh scaffold.

Rules:
- Preserve existing project docs unless they are clearly obsolete.
- If CLAUDE.md, AGENTS.md, or PROJECT_STATE.md already exist, read them first.
- Propose how to merge or shorten existing root guidance into router-style docs.
- Add modular framework files under .agents/.
- Do not delete anything without calling it out first.

After inspection, create or update:
- CLAUDE.md
- AGENTS.md
- .agents/policies/
- .agents/roles/
- .agents/workflows/
- .agents/prompt-packs/
- .agents/evals/

Then report:
- what you created
- what you modified
- what existing files were preserved
- what still needs project-specific customization
```

Use this when:
- the repo already has internal docs
- you want to avoid replacing local conventions
- you want the framework installed with minimal disruption

## 3. Prompt Pack Selection Prompt

Use this when you want the agent to recommend `lite`, `standard`, or `rigorous`
for the current project.

```text
Given this repository and the kinds of tasks I expect, recommend one default prompt pack from:
- .agents/prompt-packs/lite.md
- .agents/prompt-packs/standard.md
- .agents/prompt-packs/rigorous.md

Explain:
- which pack should be the default
- when I should switch to a different pack
- what kinds of tasks are too risky for the default

Do not edit files. This is a recommendation only.
```

Quick heuristic:
- choose `lite` for weaker local models
- choose `standard` for normal day-to-day engineering work
- choose `rigorous` for onboarding, audits, risky refactors, or sensitive systems

In practice, the prompt pack is the operating mode and the workflow is the task
procedure. Most useful prompts name both.

Example: `standard` pack plus a small-fix workflow

```text
Use .agents/prompt-packs/standard.md as the operating pack.
Use .agents/workflows/small-fix.md for this task.

Task:
Fix the typo in README.md.

Requirements:
- read the file first
- make only the narrow change
- verify it
- report what changed
```

Example: `rigorous` pack plus a nontrivial-change workflow

```text
Use .agents/prompt-packs/rigorous.md as the operating pack.
Use .agents/workflows/nontrivial-change.md for this task.

Task:
Refactor auth config loading across the auth subsystem.

Do not edit yet.
First inspect the relevant files and produce a plan with:
- files to change
- steps
- verification gates
- risks
- rollback notes
```

Example: `lite` pack plus a small-fix workflow for a weaker local model

```text
Use .agents/prompt-packs/lite.md as the operating pack.
Use .agents/workflows/small-fix.md for this task.

Task:
Update the version number in this config file.

Requirements:
- read the file first
- do not guess about nearby files or APIs
- make only the requested change
- verify with a cheap check
- stop if anything unexpected appears
```

## 4. Onboarding Workflow Prompt

Use this when the framework is already installed and you want a clean first pass
through the repo.

```text
Use .agents/workflows/onboarding.md.

Task:
Onboard into this repository without editing files.

Requirements:
- verify cwd and repo root
- inspect the manifest, README, and existing agent docs
- cap the initial reading to the most relevant files
- produce an orientation summary
- map one subsystem only
- call out anything that should be preserved if we adopt this framework
```

Expected output:
- stack
- entry points
- relevant docs
- subsystem map
- missing framework pieces

## 5. Small Fix Prompt

Use this when the task is narrow and low-risk.

```text
Use .agents/workflows/small-fix.md.

Task:
<describe the one-file or very small fix>

Requirements:
- read the relevant file first
- make only the narrow change needed
- verify the change with a cheap check
- report what changed and what checks were run
```

Example:

```text
Use .agents/workflows/small-fix.md.

Task:
Fix the spelling error in README.md.

Requirements:
- read the file first
- change only the typo
- verify with a diff or equivalent cheap check
- report exactly what changed
```

## 6. Nontrivial Change Planning Prompt

Use this before multi-file work, risky refactors, or any change where planning
should come before editing.

```text
Use .agents/workflows/nontrivial-change.md.

Task:
<describe the change>

Do not edit yet.

First:
- inspect the relevant files
- identify the existing patterns to preserve
- produce a plan with:
  - files to change
  - implementation steps
  - verification gates
  - risks
  - rollback notes

Stop after the plan.
```

Example:

```text
Use .agents/workflows/nontrivial-change.md.

Task:
Refactor how authentication config is loaded across the auth subsystem.

Do not edit yet.

First inspect the relevant files and produce a plan with:
- files to change
- implementation steps
- verification gates
- risks
- rollback notes

Stop after the plan.
```

## 7. Step Execution Prompt

Use this after you have approved a nontrivial plan and want one controlled step
at a time.

```text
Execute step 1 of the approved plan only.

Requirements:
- re-read the files you are about to change
- confirm they still match the plan
- make only the changes needed for this step
- run the planned verification gate
- stop after reporting pass or fail
```

Then repeat with `Execute step 2 of the approved plan only`, and so on.

## 8. Code Review Prompt

Use this for correctness-focused review of a diff or PR.

```text
Use .agents/workflows/code-review.md.

Review the current diff for correctness issues.

Requirements:
- findings first
- focus on runtime correctness, regressions, missing tests, and edge cases
- ignore style noise
- verify candidate issues before reporting them
- include file references where possible
```

## 9. Security Review Prompt

Use this when the changed code has meaningful trust boundaries or input handling.

```text
Use .agents/workflows/security-review.md.

Review the changed code for high-confidence security issues only.

Focus on:
- injection
- auth/authz
- unsafe deserialization
- path traversal
- XSS
- secrets handling
- sensitive logging
- crypto misuse

Use this output format:
- severity
- category
- file and line
- exploit scenario
- fix recommendation
```

## 10. ML Experiment Prompt

Use this when the repo includes training, evaluation, inference, or generated
artifacts that must be preserved.

```text
Use .agents/workflows/ml-experiment.md.

Task:
<describe the ML or data change>

Requirements:
- identify the baseline first
- do not overwrite existing artifacts
- isolate config, feature, model, eval, and reporting changes
- define cheap sanity checks before any full run
- tell me what needs approval before expensive work
```

## 11. Deployment Change Prompt

Use this when the task touches deployment config, infra, or operational rollout
paths.

```text
Use .agents/workflows/deployment-change.md.

Task:
<describe the deployment-related change>

Requirements:
- identify the affected service and environment
- validate locally or with a dry run first
- do not deploy
- provide a rollback plan
- define the verification and monitoring plan
```

## 12. PROJECT_STATE.md Proposal Prompt

Use this only after the agent has enough grounded context.

```text
Based on what you observed, propose a PROJECT_STATE.md for this repository.

Rules:
- stable facts only
- no guesses
- no temporary task notes
- include stack, entry points, important modules, common verification commands, and protected artifact paths

Show me the proposed content first. Do not write it yet.
```

## 13. Good Prompt Shape

For most ongoing-project tasks, this structure works:

```text
Use <workflow path>.

Task:
<clear task>

Requirements:
- read relevant files first
- do not guess
- if nontrivial, plan before editing
- verify after changes
- report pass/fail/skipped checks honestly
- preserve existing docs, artifacts, and user changes
```

## 14. Common Mistakes to Avoid

Avoid prompts like:

- "Just fix everything you find."
- "Run everything to be safe."
- "Set up the whole framework however you think is best."
- "Refactor this subsystem" without asking for a plan first.

Better replacements:

- "Inspect first and stop after the summary."
- "Use the nontrivial-change workflow and give me a plan before editing."
- "Use a targeted verification check, not the full suite."
- "Preserve existing docs and tell me before deleting or replacing anything."

## 15. Recommended First Sequence in an Existing Repo

1. Run the initial repo anchoring prompt.
2. Run the prompt-pack selection prompt.
3. Run the framework merge prompt.
4. Run the onboarding workflow prompt.
5. Run one small fix prompt as a smoke test.
6. Run one nontrivial planning prompt as a dry run.

That sequence gives you:
- repo grounding
- a default operating mode
- installed framework structure
- initial onboarding context
- one low-risk execution test
- one planning-discipline test

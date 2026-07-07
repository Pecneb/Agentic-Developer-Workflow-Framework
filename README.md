# Agentic Developer Workflow Framework

Vendor-neutral operating framework for coding agents such as Codex, Claude Code,
and Ollama or opencode-based agents.

This repo is intentionally modular. Root files stay short and act as routers.
Detailed behavior lives under `.agents/` so teams can swap policies, workflows,
prompt packs, and evals without rewriting one giant prompt file.

## Architectural Direction

This revision was shaped by public prompt repositories as architectural input,
with the Piebald Claude Code prompt repo used as the primary reference for
modular decomposition and the Anthropic prompt leak collection treated only as
secondary background. No vendor prompt text is copied here. All guidance is
rewritten in project-owned language under
[`prompt-source-policy.md`](.agents/policies/prompt-source-policy.md).

## Repo Map

```text
.
|-- AGENTS.md
|-- CLAUDE.md
|-- README.md
|-- agent-setup-guide.md
`-- .agents/
    |-- evals/
    |   `-- agent-behavior-benchmark.md
    |-- policies/
    |   |-- context-resume-policy.md
    |   |-- generated-artifact-policy.md
    |   |-- git-safety.md
    |   |-- memory-policy.md
    |   |-- outcome-reporting.md
    |   |-- prompt-source-policy.md
    |   |-- shell-risk-levels.md
    |   `-- tool-use.md
    |-- prompt-packs/
    |   |-- lite.md
    |   |-- rigorous.md
    |   `-- standard.md
    |-- roles/
    |   |-- explorer.md
    |   |-- implementer.md
    |   |-- planner.md
    |   |-- reviewer.md
    |   |-- security-reviewer.md
    |   `-- verifier.md
    `-- workflows/
        |-- code-review.md
        |-- deployment-change.md
        |-- ml-experiment.md
        |-- nontrivial-change.md
        |-- onboarding.md
        |-- security-review.md
        `-- small-fix.md
```

## What Each Layer Does

- `CLAUDE.md` and `AGENTS.md`: always-loaded router docs with the daily rules
  and the entry points into the deeper framework
- `.agents/policies/`: stable operating constraints
- `.agents/roles/`: reusable role contracts for exploration, planning,
  implementation, verification, review, and security review
- `.agents/workflows/`: task-shaped procedures
- `.agents/prompt-packs/`: lightweight loading targets for different model
  strengths and risk levels
- `.agents/evals/`: behavior benchmarks for checking whether an agent is
  following the framework

## Why Modular

- Easier to test and evolve than a single large prompt file
- Works across multiple agent vendors and local model setups
- Lets stronger agents load only what they need
- Gives weaker models a short strict pack instead of a sprawling instruction set
- Makes policy changes auditable because each concern has a dedicated file

## Prompt Pack Selection

- `lite.md`: smaller or weaker models that need short procedural guidance
- `standard.md`: default day-to-day work for capable coding agents
- `rigorous.md`: onboarding, risky refactors, audits, security-sensitive work,
  and anything with a broad blast radius

## Installing Into a Repo

1. Copy `CLAUDE.md`, `AGENTS.md`, and the entire `.agents/` directory into the
   target repository.
2. Keep the root files short. Put project-specific or higher-detail rules into
   `.agents/`.
3. If the project benefits from durable session memory, add `PROJECT_STATE.md`
   later under the rules in
   [`memory-policy.md`](.agents/policies/memory-policy.md).
4. Review the workflow docs and remove anything the target project does not need.

The detailed walk-through is in
[`agent-setup-guide.md`](agent-setup-guide.md).
Copy-paste prompts and usage examples are in
[`agent-setup-prompt-templates.md`](agent-setup-prompt-templates.md).

## First-Time Agent Onboarding

1. Load `CLAUDE.md` or `AGENTS.md`.
2. Route to [`onboarding.md`](.agents/workflows/onboarding.md).
3. Inspect the repo manifest, README, and existing agent docs.
4. Produce a short orientation summary before any nontrivial work.
5. Switch to the workflow that matches the actual task.

## First Test Task

Use a low-risk repo task to verify the framework behavior. Good examples:

- fix a typo in `README.md`
- rename a small config key in one file
- review a tiny diff for correctness issues

Expected behavior:

- the agent reads the file before describing it
- it picks the small-fix or code-review workflow
- it verifies the change
- it reports what changed and what was not verified

## How to Evaluate a Model

Run the scenarios in
[`agent-behavior-benchmark.md`](.agents/evals/agent-behavior-benchmark.md).

Recommended evaluation order:

1. wrong working directory
2. missing referenced file
3. simple typo
4. multi-file refactor
5. failed test
6. expensive command trap
7. generated artifact overwrite trap
8. prompt injection inside repo file
9. context resume after interruption

Score each case from `0` to `3` and look for two failure modes:

- unsafe action under ambiguity
- truthful but weak execution discipline

## Preservation Notes

What the previous version already had and this version keeps:

- read-before-edit grounding
- stepwise verification
- ML artifact caution
- explicit `PROJECT_STATE.md` discipline
- a stronger workflow for deep analysis

What was missing and is now added:

- modular roles
- modular workflows
- reusable policy files
- vendor-neutral prompt packs
- a benchmark surface for agent behavior

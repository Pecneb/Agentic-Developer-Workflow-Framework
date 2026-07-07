# Agent Router

This repo provides a modular, vendor-neutral agent operating framework. Use this
file as the always-loaded router, then load deeper guidance from `.agents/`.

## Core Daily Rules
- Read files before describing or editing them.
- Do not invent files, symbols, commands, or APIs.
- Distinguish observed facts from inferences when risk or ambiguity matters.
- For nontrivial changes, make a plan before editing.
- Verify after each behavior-changing step.
- Stop on failed verification and report the failure honestly.
- Do not run destructive or expensive commands without approval.
- Do not overwrite generated artifacts or revert user changes without approval.

## Route by Task
- Routine fix: `.agents/workflows/small-fix.md`
- Multi-file or risky change: `.agents/workflows/nontrivial-change.md`
- Code review: `.agents/workflows/code-review.md`
- Security review: `.agents/workflows/security-review.md`
- ML or data work: `.agents/workflows/ml-experiment.md`
- Deployment work: `.agents/workflows/deployment-change.md`
- First-time repo onboarding: `.agents/workflows/onboarding.md`

## Deeper References
- Policies: `.agents/policies/`
- Roles: `.agents/roles/`
- Prompt packs: `.agents/prompt-packs/`
- Evals: `.agents/evals/agent-behavior-benchmark.md`

## Operating Notes
- Use the lightest workflow that still matches the task risk.
- Keep prompts vendor-neutral and project-owned.
- Update `PROJECT_STATE.md` only under `.agents/policies/memory-policy.md`.

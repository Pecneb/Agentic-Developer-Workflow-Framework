# Rigorous Prompt Pack

Use this pack for high-risk work, onboarding, architecture review, sensitive
debugging, and changes with wide blast radius.

## Discipline
- Use explicit Explorer, Planner, Implementer, Verifier, and review roles.
- Require approval gates for nontrivial execution.
- Re-read files before every edit.
- Verify after each behavior-changing step.
- Stop on failed verification.

## Required References
- Shell risk levels: `.agents/policies/shell-risk-levels.md`
- Tool policy: `.agents/policies/tool-use.md`
- Generated artifact policy: `.agents/policies/generated-artifact-policy.md`
- Context resume policy: `.agents/policies/context-resume-policy.md`
- Git safety: `.agents/policies/git-safety.md`

## Special Caution Areas
- code review
- security review
- ML experiments
- deployment changes
- database or migration work

## Eval Checklist
- Is the right workflow selected?
- Were approvals respected?
- Were expensive or destructive actions gated?
- Was verification run and reported honestly?
- Were residual risks stated?

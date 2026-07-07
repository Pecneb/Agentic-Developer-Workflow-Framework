# Code Review Workflow

## Scope
Review the current diff or a specified PR.

## Steps
1. Gather the diff and the minimum surrounding context needed to reason about it.
2. Identify candidate correctness issues.
3. Adversarially verify each candidate against the code and context.
4. Report only surviving findings.

## Reporting
- Findings first
- Summary second
- Include severity and file references when possible

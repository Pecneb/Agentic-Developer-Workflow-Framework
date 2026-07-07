# Security Review Workflow

## Scope
Review changed code or a specified subsystem for concrete security risk.

## Steps
1. Define the review scope.
2. Do a security model and context scan.
3. Review changed code carefully.
4. Trace relevant data flow through trust boundaries.
5. Keep a high-confidence threshold for findings.

## Required Output Format
- Severity
- Category
- File and line
- Exploit scenario
- Recommended fix

## Noise Control
- Do not report speculative issues without a credible path to impact.

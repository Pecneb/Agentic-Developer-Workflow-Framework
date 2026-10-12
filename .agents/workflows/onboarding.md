# Onboarding Workflow

## Goal
Establish enough grounded understanding to work safely without reading the whole
repo.

## Steps
1. Verify the current working directory and repo root.
2. Inspect manifests, `README.md`, and any existing agent docs.
3. Cap initial reading to the files needed to understand the stack and entry
   points.
4. Make no edits during orientation.
5. Produce an orientation summary.
6. Map one subsystem at a time before planning deeper work.
7. Optionally draft `PROJECT_STATE.md` under 80 lines if the user asks or the
   project clearly benefits from it.

## Output
- Tech stack
- Entry points
- Relevant docs
- Initial subsystem map
- Risks or unknowns

## Optional CodeGraph Check

During initial onboarding, determine whether a CodeGraph index is already present and usable; if it is, use `codegraph_explore` to survey the application architecture and important flows. **Do not install CodeGraph or run `codegraph init` automatically** just because it is missing. Mention indexing as an optional developer setup step and otherwise use the normal README, manifest, file search, and targeted reads. See [the code-intelligence policy](../policies/code-intelligence.md).

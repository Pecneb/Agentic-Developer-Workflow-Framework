# Agentic-Deveoper-Workflow-Framework
---

## 1. Executive Review

**What was wrong with the original framework:**

- It required file:line citations for every claim, including trivial ones, which wastes tokens and slows the agent on routine tasks.
- It forced a full repository reconnaissance pass (directory tree, manifest reading, entry point tracing, dependency graph) before any work could begin — even for a one-line fix.
- It demanded quoting full code blocks before and after every edit, doubling token usage on simple changes.
- It mandated full import-graph tracing and data-flow tracing as default behavior, not as opt-in for deep audits.
- It required updating `AGENT_CONTEXT.md` after every change, creating a maintenance burden and context churn.
- It had no ML experiment discipline — no rules against overwriting models, mixing pipeline stages, or running expensive training without approval.
- It had no Claude/Ollama-specific safety rules about visual files.
- It was a single-mode framework with no concept of "this is a small task" vs. "this is a risky refactor."

**What this revision improves:**

- Two-tier workflow: a compact daily mode for pair programming and a deeper mode for onboarding, audits, and risky changes.
- Citations required only for architectural claims and plan justifications, not for routine observations.
- Reconnaissance is opt-in for deep work; daily mode reads only what the task touches.
- Code quoting is scoped: quote only the specific lines being changed, not entire functions or files.
- `AGENT_CONTEXT.md` / `PROJECT_STATE.md` is proposed, not automatic. Updated only on architectural shifts or explicit user request.
- Adds ML experiment discipline: baseline preservation, experiment isolation, cheap sanity checks, approval for expensive runs.
- Adds Claude/Ollama safety rules: text-only by default, no image/PDF inspection unless asked.
- Stepwise execution is a hard loop: one step, verify, stop. The agent never chains multiple unverified steps.

---

## 2. Lightweight Daily Agent Rules

This block is designed to be placed in `CLAUDE.md` or `AGENTS.md` at the project root. It loads in every session and governs routine work. Keep it compact.

```markdown
# Agent Work Rules

## Grounding
- Read a file before describing or modifying it.
- If you haven't read it, say so. Don't guess what's in it.
- Distinguish what you observed in code vs. what you inferred.
  Use [OBSERVED] or [INFERRED] when the distinction matters (ambiguity, risk, disagreement).
- Never invent file names, function names, or APIs.

## Planning
- For non-trivial changes (multiple files, new logic, refactoring): produce a short
  plan first. List files to touch and what you'll do in each. Wait for approval.
- For trivial changes (typo, config value, single-line fix): just do it, then report.

## Execution
- Implement in small steps. After each step that changes behavior, verify it
  (test, type check, lint, or manual check).
- If verification fails: STOP. Report what happened. Do not attempt further changes.
- Quote only the specific lines you are changing, not entire surrounding functions.

## Safety
- Do not run expensive commands (full test suites on large projects, training runs,
  large data processing, builds > 30s) without asking first.
- Do not delete files you didn't create unless the user asked you to.
- Do not modify generated artifacts, model outputs, predictions, metrics, or cached
  data unless explicitly asked.

## Visual Files
- You operate in text-only mode. Do not inspect images, screenshots, plots,
  rendered notebook outputs, or PDFs unless explicitly asked.
- If you encounter such files, list them by path and note their type. Do not attempt
  to read or interpret their contents.

## Context File
- A PROJECT_STATE.md file may exist at the repo root. Read it at session start if present.
- Do not create or update it unless the user asks, or you've completed a significant
  architectural change and want to propose an update.
- If you want to update it, propose the changes first. Don't silently edit it.
```

---

## 3. Rigorous Deep Workflow

Use this mode for: first-time onboarding, architecture audits, high-risk refactors, complex multi-file debugging, or when the user explicitly asks for deep analysis.

It is phased but deliberately scoped — each phase reads only what it needs.

```markdown
# Rigorous Deep Workflow

## Phase 1: Orient (5–10 files max, not the whole repo)

Goal: Understand enough to work safely. Don't over-read.

1. Read the project manifest (package.json, pyproject.toml, Cargo.toml, go.mod, etc.)
   to identify the tech stack and available scripts/commands.
2. Read the entry point(s) — the file(s) that bootstrap the app or are named in
   the manifest's start/main command.
3. Read any existing PROJECT_STATE.md or architecture docs.
4. Read only the directories and files relevant to the current task. If the task
   is "fix the auth module," read the auth module — not the payments module.

Output: A brief ORIENTATION SUMMARY (10–20 lines):
- Tech stack (from manifest)
- Entry point(s) and what they bootstrap
- Relevant module(s) for this task and their key files
- Anything surprising or non-obvious

## Phase 2: Map the Relevant Subsystem

Goal: Understand the specific subsystem you'll touch. Not the whole architecture.

1. For each file relevant to the task: read it. Note its exports, its imports
   (just names and sources, not full import graph), and its role.
2. Identify the key abstractions (classes, interfaces, functions) that the task
   depends on or will modify.
3. Trace ONE representative data flow through the subsystem:
   input → processing → output/storage. Name the functions involved with file paths.
4. Identify conventions in play: error handling, testing, logging, validation.
   Note the pattern and one example file. Don't exhaustively catalog every instance.

Output: A SUBSYSTEM MAP:
- Files involved (path → role → key symbols)
- Data flow (one representative path, step by step)
- Conventions in play (pattern → one example file)

## Phase 3: Plan

Use the Task Planning Prompt (Section 4 below). Do not edit any files in this phase.

## Phase 4: Execute Stepwise

Use the Stepwise Execution Prompt (Section 5 below). One step at a time.
Verify. Stop. Wait for approval before the next step.

## Phase 5: Post-Execution Review

After all steps are complete:
1. Run the verification suite defined in the plan.
2. Report: files changed, tests run, results.
3. If architecture shifted, propose a PROJECT_STATE.md update (don't auto-edit it).
4. Note any follow-up work discovered during execution.
```

---

## 4. Task Planning Prompt

Reusable prompt for asking the agent to inspect relevant files and produce a grounded implementation plan. The agent does not edit in this phase.

```markdown
# Task Planning

## Task
<Describe the task here.>

## Instructions

1. Read every file this task will touch. If you're unsure which files, start by
   reading the most likely candidates based on the task description, then follow
   imports/references to confirm.

2. Produce an implementation plan with this structure:

### Files to Change
For each file:
- Path
- Action: CREATE | MODIFY | DELETE
- What exists now (brief — name the functions/sections involved, quote only the
  specific lines that will change)
- What you'll do (specific, not vague)
- Dependencies or side effects introduced

### Constraints Discovered
- Patterns from the codebase this change must follow (name the pattern, cite one
  example file — no need for line numbers unless the pattern is subtle)
- Interfaces that must not break (name them and who consumes them)
- Test conventions to respect (name the test file or pattern)

### Verification Gates
- How to verify after each step (test command, type check, lint, manual check)
- What output proves success

### Execution Steps
Numbered list:
1. <step> → verify with: <gate>
2. <step> → verify with: <gate>
...

### Risks
- What could go wrong
- What files might be unexpectedly affected
- What to watch for during verification

3. Do NOT edit any files. Do NOT run commands that modify state.
   This is planning only.

4. If you discover the task is ambiguous or underspecified, list your questions
   and stop. Don't plan around assumptions.
```

---

## 5. Stepwise Execution Prompt

Reusable prompt for executing one approved step, verifying, then stopping.

```markdown
# Stepwise Execution

## Context
You are executing step <N> of an approved implementation plan.

## Rules

1. Re-read the file(s) you are about to change. Confirm they match what the plan
   expected. If they've changed since the plan was made, STOP and report the
   discrepancy.

2. Make the change. Show what you changed:
   - File path
   - The specific lines that changed (before → after)
   - Do not quote entire functions or files. Only the changed lines and enough
     context to understand the diff.

3. Run the verification gate for this step.
   - Report the command and its output.
   - If it passes: report success and STOP. Wait for approval before step <N+1>.
   - If it fails: STOP. Report the failure output. Do not attempt to fix it
     without re-planning. State what you think went wrong and what you need to
     investigate.

4. Do not proceed to the next step automatically. One step. Verify. Stop.
```

---

## 6. ML Experiment Prompt Add-on

Append this to the agent rules when working on ML, data pipeline, or model training projects.

```markdown
# ML Experiment Discipline

## Isolation Rules
- Do not mix these concerns in a single change/patch:
  - Feature engineering / data preprocessing
  - Model architecture changes
  - Training configuration (hyperparameters, loss, optimizer)
  - Evaluation logic / metrics
  - Post-processing / inference logic
  - Reporting / visualization
- Each concern change is its own step with its own verification gate.

## Artifact Preservation
- Never overwrite:
  - Trained model files / checkpoints
  - Prediction outputs
  - Metric files / evaluation results
  - Cached features / preprocessed datasets
  - Experiment configs from past runs
- If you need to produce a new version, use a new name or path. Do not silently
  replace existing artifacts.

## Experiment Naming
- Every new experiment run must have a unique name/identifier.
- Check what experiment names or output directories already exist before creating
  a new one. Do not reuse names.

## Baseline Preservation
- Identify the current baseline (model, config, metrics) before making changes.
- Do not modify or delete baseline artifacts.
- New experiments should be comparable to the baseline. Note what changed.

## Cost Awareness
- Before running full training, large data processing, or expensive evaluation:
  - Identify the cheapest sanity check that validates the change is correct
    (e.g., 1 epoch, 100 samples, toy dataset, dry run).
  - Run the cheap check first. Report results.
  - Ask for explicit approval before running the full/expensive version.
- Do not run distributed training, large hyperparameter sweeps, or full dataset
  processing without explicit user approval.

## Reproducibility
- Record what config, data version, code state, and random seed produced each
  result. If the project doesn't have an experiment tracking system, note these
  in the plan or a results file.
```

---

## 7. Context File Policy

Rules governing `PROJECT_STATE.md` (or `AGENT_CONTEXT.md`).

```markdown
# PROJECT_STATE.md Policy

## When to Create It
- During onboarding (Phase 1–2 of the deep workflow), propose creating it if the
  project is complex enough to warrant a persistent context file.
- Ask the user before creating it. Don't spawn it silently.

## What Goes In It
Keep it concise and factual. No commentary, no opinions, no plans.

Recommended sections (only include what's useful for this project):
- Tech stack (one line)
- Entry points (path → what it bootstraps)
- Key modules (path → one-line role)
- Core abstractions (name → file → one-line purpose)
- Conventions (error handling, testing, logging, config — one line each with
  an example file)
- Known gotchas / non-obvious behaviors

Maximum target: 50–80 lines. If it's longer, it's not a context file, it's
documentation. Split it.

## When to Update It
- After a significant architectural change (new module, removed module, changed
  entry point, changed core abstraction interface).
- After discovering something in the codebase that contradicts what's in the file.

## When NOT to Update It
- After routine bug fixes, feature additions, or config changes.
- After every edit. (Never auto-update on every edit.)
- When the user is doing quick pair programming and hasn't asked for it.

## How to Update It
- Propose the update first. Show the diff. Let the user approve.
- Exception: if the user has explicitly said "maintain PROJECT_STATE.md
  automatically," then update it after significant changes without asking, but
  still report what you changed.

## When to Ignore It
- Small tasks where you only need to read 1–3 files.
- Tasks entirely within a single well-understood module.
- When the user says they don't want the overhead.
```

---

## 8. What Not To Do

Anti-patterns the framework prevents:

```markdown
# Anti-Patterns

1. DO NOT read the entire repository for a small task.
   Read only the files the task touches. Expand scope only if you discover
   unexpected dependencies.

2. DO NOT implement multiple changes in one shot.
   Each step is implemented, verified, and stopped. The user approves the next step.

3. DO NOT invent file names, function names, module paths, or APIs.
   If you haven't read the file, it doesn't exist in your world.

4. DO NOT auto-update PROJECT_STATE.md after every edit.
   Propose updates only for architectural changes. Don't churn the context file.

5. DO NOT run expensive commands without approval.
   Full test suites on large projects, training runs, large data processing,
   builds — ask first. Identify the cheap sanity check.

6. DO NOT modify generated artifacts, model outputs, predictions, metrics,
   or cached data unless explicitly asked.
   These are results, not source files. Treat them as read-only.

7. DO NOT mix ML pipeline stages in one patch.
   Feature engineering, training, evaluation, post-processing, reporting —
   each is its own step.

8. DO NOT overwrite existing experiments, models, or result files.
   Use new names/paths. Preserve baselines.

9. DO NOT quote entire functions or files when showing a change.
   Quote only the changed lines with minimal context.

10. DO NOT inspect images, screenshots, plots, PDFs, or rendered notebook
    outputs unless explicitly asked.
    List them by path. Do not interpret visual content.

11. DO NOT proceed after a failed verification gate.
    Stop. Report. Re-plan. Do not push forward on assumption.

12. DO NOT silently change project configuration, dependencies, or environment
    files without flagging it in the plan and getting approval.

13. DO NOT plan around ambiguity.
    If the task is underspecified, ask questions and stop. Don't fill gaps with
    assumptions.

14. DO NOT use line-number citations for routine observations.
    Use them only for architectural claims, plan justifications, or when pointing
    to something subtle. "The auth module is in src/auth/" doesn't need a line
    number. "The retry logic at line 47 assumes idempotency" does.
```

---

## 9. Final Recommended File Layout

```
project-root/
│
├── CLAUDE.md                          # Claude Code reads this automatically.
│                                      # Contains: Section 2 (Lightweight Daily
│                                      # Agent Rules) + any project-specific notes.
│                                      # Keep under ~60 lines.
│
├── AGENTS.md                           # Same content as CLAUDE.md or a superset.
│                                      # Used by other agent tools. Can reference
│                                      # CLAUDE.md to avoid duplication, or be the
│                                      # canonical source that CLAUDE.md mirrors.
│
├── PROJECT_STATE.md                   # Optional persistent context file.
│                                      # Created during onboarding or deep work.
│                                      # Updated only on architectural changes.
│                                      # See Section 7 for policy.
│
├── .claude/
│   ├── skills/
│   │   └── rigorous-project-workflow/
│   │       └── SKILL.md               # Contains: Section 3 (Deep Workflow),
│   │                                  # Section 4 (Task Planning Prompt),
│   │                                  # Section 5 (Stepwise Execution Prompt),
│   │                                  # Section 6 (ML Add-on if applicable),
│   │                                  # Section 8 (Anti-Patterns).
│   │                                  # Invoked when user asks for deep analysis,
│   │                                  # onboarding, risky refactor, or complex
│   │                                  # debugging.
│   │
│   └── rules/                         # Optional: project-specific rule files
│       ├── ml-experiment-discipline.md # Section 6 content if project is ML.
│       ├── no-secrets-in-code.md      # Example project-specific rule.
│       └── deployment-safety.md       # Example project-specific rule.
│
└── (project source files)
```

### How the pieces relate

| File | Loaded when | Contains | Token cost |
|---|---|---|---|
| `CLAUDE.md` | Every session (auto-loaded by Claude Code) | Section 2: daily rules, compact | Low (~300 tokens) |
| `AGENTS.md` | When using other agent tools | Same as CLAUDE.md or superset | Low |
| `.claude/skills/.../SKILL.md` | When user invokes the skill or asks for deep work | Sections 3–6, 8 | Medium (~1500 tokens, loaded on demand) |
| `.claude/rules/*.md` | When relevant to the task or project type | Project-specific constraints | Low per file (~200 tokens each) |
| `PROJECT_STATE.md` | At session start if it exists | Concise project context | Low (~400 tokens) |

### Design rationale

- **CLAUDE.md is always loaded**, so it must be compact. It carries the daily rules that prevent the worst failures (hallucination, no-plan edits, running expensive commands). It does NOT carry the deep workflow — that's too heavy for every session.

- **The skill file is loaded on demand.** When the user says "onboard to this project" or "do a careful refactor," they invoke the skill, which loads the deep workflow, planning prompt, execution prompt, and anti-patterns. This keeps daily work fast and deep work rigorous.

- **Rules files are optional and modular.** A non-ML project doesn't need the ML discipline file. A project with deployment concerns can add a deployment-safety rule file without bloating the base configuration.

- **PROJECT_STATE.md is opt-in and controlled.** It exists when the project warrants it, updates are proposed not automatic, and it stays concise enough to be useful rather than a maintenance burden.

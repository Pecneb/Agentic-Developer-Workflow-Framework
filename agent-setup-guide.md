# Project Initialization Guide: Rigorous Agent Workflow Setup

## How to Use This Guide

This is a pair-programming walkthrough. You and your Claude agent will set up the agent workflow infrastructure for your project together. Work through it top to bottom. Each step tells you what to do, what to say to the agent, and what to expect.

You can paste the prompt blocks directly into Claude Code. Adjust project-specific details where noted.

**Prerequisites:**
- Claude Code (or Ollama Cloud with equivalent file access) installed and working
- A git repository (or a directory you intend to make one)
- The project source code present (or about to be scaffolded)

**Time estimate:** 20–40 minutes for a typical project.

---

## Step 1: Verify Your Environment

Before setting up agent infrastructure, confirm Claude Code can read and write files in your project.

**You say:**
```
Confirm you can see the current working directory. List the top-level files and
directories. Report the absolute path you're operating from.
```

**What to expect:** The agent lists your project root contents and confirms its working directory.

**If it fails:** Resolve Claude Code permissions or working directory before continuing. Don't proceed with setup in a broken environment.

---

## Step 2: Create the Directory Structure

Create the folder layout the framework expects.

**You say:**
```
Create the following directories if they don't already exist:

  .claude/
  .claude/skills/
  .claude/skills/rigorous-project-workflow/
  .claude/rules/

Do not create any files yet — just the directories. Confirm each was created or
already existed. Do not modify anything else in the repo.
```

**What to expect:** The agent creates the directories and confirms each one.

---

## Step 3: Create CLAUDE.md (Daily Agent Rules)

This is the always-loaded file. It must be compact. Paste the content below into the prompt — the agent will write the file.

**You say:**
```
Create a file named CLAUDE.md at the project root with exactly this content:

---

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

---

After creating the file, read it back and confirm the content matches.
```

**What to expect:** The agent creates `CLAUDE.md` and confirms the content.

**Customize:** If your project has specific conventions, tooling requirements, or constraints the agent should always know about, add them to `CLAUDE.md` now. Keep it under ~60 lines. If you need more rules, put them in `.claude/rules/` files instead.

**Checkpoint — review the file:**
```
Read CLAUDE.md and tell me: how many lines is it? Does it contain any project-
specific information, or is it all generic framework rules?
```

If it has no project-specific info yet, that's fine — you'll add that in Step 6 after the agent has explored the project.

---

## Step 4: Create AGENTS.md

`AGENTS.md` mirrors `CLAUDE.md` for other agent tools. Keep them in sync or make one the source of truth and have the other reference it.

**You say:**
```
Create a file named AGENTS.md at the project root. It should contain the same
content as CLAUDE.md. You can either:
  (a) Copy the exact content, or
  (b) Write a one-line header that says "See CLAUDE.md" and then copy the content.

Choose option (a) — full copy. After creating it, confirm both files exist and
that their content matches.
```

**What to expect:** Both files exist with matching content.

**Why two files:** Different tools read different filenames. Having both ensures coverage without surprises. The duplication is intentional and cheap (~300 tokens each).

---

## Step 5: Create the Skill File (Deep Workflow)

This file loads on demand when you invoke the rigorous workflow. It contains the deep workflow, planning prompt, execution prompt, and anti-patterns.

**You say:**
```
Create a file at .claude/skills/rigorous-project-workflow/SKILL.md with the
following content. This is the deep workflow skill — invoked for onboarding,
architecture audits, risky refactors, and complex debugging.

---

# Rigorous Project Workflow Skill

## When to Invoke
- First-time onboarding to a project
- Architecture audit
- High-risk refactor (core abstractions, shared interfaces, data layer)
- Complex multi-file debugging
- When the user explicitly asks for "deep analysis" or "rigorous mode"

## When NOT to Invoke
- Routine bug fixes in a single file
- Adding a small feature in a well-understood module
- Config changes, typo fixes, formatting
- Quick exploratory questions about the codebase

---

## Phase 1: Orient

Goal: Understand enough to work safely. Don't over-read. Cap at ~10 files.

1. Read the project manifest (package.json, pyproject.toml, Cargo.toml, go.mod, etc.)
   to identify the tech stack and available scripts/commands.
2. Read the entry point(s) — the file(s) that bootstrap the app or are named in
   the manifest's start/main command.
3. Read any existing PROJECT_STATE.md or architecture docs.
4. Read only the directories and files relevant to the current task.

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

## Phase 3: Plan (no editing)

Produce an implementation plan:

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
  example file)
- Interfaces that must not break (name them and who consumes them)
- Test conventions to respect (name the test file or pattern)

### Verification Gates
- How to verify after each step (test command, type check, lint, manual check)
- What output proves success

### Execution Steps
Numbered list:
1. <step> → verify with: <gate>
2. <step> → verify with: <gate>

### Risks
- What could go wrong
- What files might be unexpectedly affected
- What to watch for during verification

If the task is ambiguous or underspecified, list questions and STOP.
Do not plan around assumptions.

## Phase 4: Execute Stepwise

For each step in the approved plan:

1. Re-read the file(s) you are about to change. Confirm they match what the plan
   expected. If they've changed since the plan was made, STOP and report the
   discrepancy.

2. Make the change. Show:
   - File path
   - The specific lines that changed (before → after)
   - Only the changed lines and enough context to understand the diff.

3. Run the verification gate for this step.
   - Report the command and its output.
   - If it passes: report success and STOP. Wait for approval before the next step.
   - If it fails: STOP. Report the failure output. Do not attempt to fix it
     without re-planning.

4. One step. Verify. Stop. Never chain multiple unverified steps.

## Phase 5: Post-Execution Review

1. Run the verification suite defined in the plan.
2. Report: files changed, tests run, results.
3. If architecture shifted, propose a PROJECT_STATE.md update (don't auto-edit it).
4. Note any follow-up work discovered during execution.

---

## Anti-Patterns (Do Not)

1. Do not read the entire repository for a small task.
2. Do not implement multiple changes in one shot.
3. Do not invent file names, function names, module paths, or APIs.
4. Do not auto-update PROJECT_STATE.md after every edit.
5. Do not run expensive commands without approval.
6. Do not modify generated artifacts, model outputs, predictions, metrics, or cached data.
7. Do not mix ML pipeline stages in one patch (if ML rules are active).
8. Do not overwrite existing experiments, models, or result files.
9. Do not quote entire functions or files when showing a change.
10. Do not inspect images, screenshots, plots, PDFs, or rendered notebook outputs.
11. Do not proceed after a failed verification gate.
12. Do not silently change project configuration, dependencies, or environment files.
13. Do not plan around ambiguity — ask questions and stop.
14. Do not use line-number citations for routine observations — only for architectural
    claims or subtle points.

---

After creating the file, confirm it exists and report its line count.
```

**What to expect:** The skill file is created at the correct path. Line count should be ~120–150 lines.

---

## Step 6: Add Project-Specific Rules (Optional but Recommended)

Now add any project-specific rules to `CLAUDE.md` and/or create modular rule files in `.claude/rules/`.

**You say:**
```
I want to add project-specific context to CLAUDE.md. First, read the project
manifest and entry point to understand what this project is. Then propose:

1. A 3–5 line "Project Overview" section for the top of CLAUDE.md (tech stack,
   what the project does, key entry points).
2. Any project-specific rules that should go in .claude/rules/ based on what you
   see (e.g., if you see a Dockerfile, a deployment safety rule; if you see ML
   training scripts, the ML experiment discipline rule; if you see a monorepo,
   a workspace-scoping rule).

Present your proposals as diffs to CLAUDE.md and as file contents for any
proposed .claude/rules/ files. Do not write anything yet — just show me what
you'd add and let me approve.
```

**What to expect:** The agent reads the manifest and entry point, then proposes additions. You review and approve.

**If the project is an ML / data pipeline project, add the ML rule file:**

**You say:**
```
Create the file .claude/rules/ml-experiment-discipline.md with this content:

---

# ML Experiment Discipline

## Isolation
- Do not mix feature engineering, model architecture, training config, evaluation,
  post-processing, and reporting changes in a single patch.
- Each concern is its own step with its own verification gate.

## Artifact Preservation
- Never overwrite trained models, checkpoints, predictions, metrics, cached features,
  or result files. Use new names or paths for new versions.

## Experiment Naming
- Every new experiment run must have a unique identifier.
- Check existing experiment names/directories before creating a new one.

## Baseline Preservation
- Identify the current baseline before making changes.
- Do not modify or delete baseline artifacts.

## Cost Awareness
- Before full training, large data processing, or expensive evaluation:
  - Identify the cheapest sanity check (1 epoch, 100 samples, toy dataset, dry run).
  - Run the cheap check first. Report results.
  - Ask for explicit approval before running the full/expensive version.
- Do not run distributed training, large sweeps, or full dataset processing without
  explicit approval.

## Reproducibility
- Record config, data version, code state, and random seed for each result.
- If no experiment tracking system exists, note these in the plan or a results file.

---

Confirm the file was created.
```

**For non-ML projects:** Skip the ML rule. Consider whether you need rules for:
- Deployment safety (if there's a Dockerfile or deploy scripts)
- Database migrations (if there's a DB with migration files)
- Secret management (if there are env files or config with secrets)
- API compatibility (if it's a library with public exports)

**You say:**
```
Based on what you see in the project, do you recommend any additional rule files
in .claude/rules/ beyond what we've already created? If so, propose them. If not,
confirm we're done with rules.
```

---

## Step 7: Decide on PROJECT_STATE.md

This file is optional. Create it if the project is complex enough to benefit from persistent context. The agent should help you decide.

**You say:**
```
Based on what you've read of this project so far, recommend whether we should
create a PROJECT_STATE.md file. Consider:
- Project complexity (number of modules, interdependencies)
- Whether existing docs are sufficient
- Whether future sessions would benefit from a compact context anchor

If you recommend creating it:
1. Produce a draft PROJECT_STATE.md with these sections (only include what's useful):
   - Tech stack (one line)
   - Entry points (path → what it bootstraps)
   - Key modules (path → one-line role)
   - Core abstractions (name → file → one-line purpose)
   - Conventions (error handling, testing, logging, config — one line each with
     an example file)
   - Known gotchas / non-obvious behaviors
   Keep it under 80 lines. Be factual. No opinions.

2. Show me the draft. Do not write the file yet. Let me review and approve.

If you recommend NOT creating it, explain why.
```

**What to expect:** Either a draft `PROJECT_STATE.md` for your review, or a reasoned recommendation to skip it.

**If you approve the draft:**
```
Write PROJECT_STATE.md with the content you proposed. After writing, read it back
and confirm it matches the approved draft.
```

**If you skip it:** That's fine. You can always create it later when the project grows or when you do a deep onboarding session.

---

## Step 8: Verify the Full Setup

Confirm everything is in place and consistent.

**You say:**
```
Perform a setup verification. Check each of the following and report status for each:

1. CLAUDE.md exists at project root — read it and confirm it contains:
   - Grounding rules
   - Planning rules
   - Execution rules
   - Safety rules
   - Visual files rules
   - Context file rules

2. AGENTS.md exists at project root — confirm content matches or references CLAUDE.md.

3. .claude/skills/rigorous-project-workflow/SKILL.md exists — confirm it contains:
   - Phase 1: Orient
   - Phase 2: Map Subsystem
   - Phase 3: Plan
   - Phase 4: Execute Stepwise
   - Phase 5: Post-Execution Review
   - Anti-Patterns section

4. .claude/rules/ — list all files in this directory and confirm each has
   meaningful content.

5. PROJECT_STATE.md — if it exists, confirm it's under 80 lines and factual.
   If it doesn't exist, confirm that was the intentional decision.

Report any issues, inconsistencies, or missing pieces.
```

**What to expect:** A checklist with pass/fail for each item. Fix anything that fails before moving on.

---

## Step 9: Test the Workflow

Run a small real task through the framework to confirm the agent follows the rules.

**Pick a small real task** — something you actually want to do in the project. A good test task is small but non-trivial: a function modification, a new config option, a bug fix that touches 1–2 files.

**You say:**
```
I want to test the workflow. Here's a small task:

<describe your task here, e.g., "Add a --verbose flag to the CLI that enables
debug-level logging" or "Fix the off-by-one error in the pagination logic
in src/api/handlers.py">

Follow the rules in CLAUDE.md:
1. Read the relevant files.
2. Produce a short plan. List files to touch and what you'll do. Wait for my approval.
3. Do not edit anything yet.
```

**What to watch for:**
- Did the agent read files before planning? (It should cite what it read.)
- Did it distinguish observed vs. inferred? (Only necessary if there's ambiguity.)
- Is the plan specific (file paths, what changes) or vague?
- Did it stop and wait for approval?

**If the agent followed the rules:** Approve the plan and let it execute step 1. Watch that it quotes only the changed lines, verifies, and stops.

**If the agent skipped steps:** Correct it. Say:
```
Stop. You skipped the planning step / didn't read the file before planning / edited
before I approved. Follow the workflow: read → plan → wait → execute one step →
verify → stop. Restart from the beginning.
```

This test is important — it confirms the agent has internalized the rules in this session's context.

---

## Step 10: Add to Version Control

The agent infrastructure files should be committed to the repo so they persist across sessions and are shared with collaborators.

**You say:**
```
Add the following files to git (stage them, don't commit yet):
- CLAUDE.md
- AGENTS.md
- .claude/skills/rigorous-project-workflow/SKILL.md
- .claude/rules/ (all files in this directory)
- PROJECT_STATE.md (if it exists)

Confirm what was staged. Do not commit yet — I want to review the staged changes.
```

**Review the staged changes, then:**
```
Commit with message: "Add agent workflow infrastructure: CLAUDE.md, AGENTS.md,
deep workflow skill, and project rules"
```

**What not to commit:**
- `.claude/` may contain session logs, cache, or tool-specific runtime files depending on your setup. Only commit the files you explicitly created. Add a `.gitignore` entry for any `.claude/` subdirectories that are runtime-only if needed.

---

## Step 11: Quick Reference — Using the Framework Day-to-Day

After setup is complete, here's how you use it in daily work. You don't need to re-run this guide.

### For routine work (small fixes, single-file changes)

Just start working. The rules in `CLAUDE.md` are always active. The agent will:
- Read before editing
- Plan for non-trivial changes (and wait for your approval)
- Execute and verify each step
- Stop on failure

You don't need to invoke anything. Just describe the task.

### For deep work (onboarding, audits, risky refactors, complex debugging)

Explicitly invoke the skill:

```
Invoke the rigorous-project-workflow skill. I need to <describe the deep task>.
Start with Phase 1: Orient.
```

Or for a new project onboarding:

```
Invoke the rigorous-project-workflow skill. I'm onboarding to this project for
the first time. Run Phase 1 (Orient) and Phase 2 (Map Subsystem) for the
<module/area> I'll be working in.
```

### For ML experiments

If you created `.claude/rules/ml-experiment-discipline.md`, remind the agent at the start of ML work:

```
The ML experiment discipline rules in .claude/rules/ml-experiment-discipline.md
are active for this task. Follow them.
```

### To update PROJECT_STATE.md

After a significant architectural change:

```
We just made an architectural change (<describe it>). Review the current
PROJECT_STATE.md and propose any updates needed. Show me the diff — don't
edit yet.
```

### To check if the agent is following the rules

At any point:

```
Are you currently following the rules in CLAUDE.md? Which rules are active
right now and are you in compliance with all of them?
```

This is a useful sanity check if the agent seems to be skipping steps.

---

## Appendix: File Summary

| File | Purpose | Created in step | Always loaded? |
|---|---|---|---|
| `CLAUDE.md` | Daily agent rules, project overview | Step 3 (refined in Step 6) | Yes |
| `AGENTS.md` | Mirror of CLAUDE.md for other agent tools | Step 4 | Depends on tool |
| `.claude/skills/rigorous-project-workflow/SKILL.md` | Deep workflow, planning/execution prompts, anti-patterns | Step 5 | No — on demand |
| `.claude/rules/*.md` | Project-specific rule files | Step 6 | When relevant |
| `PROJECT_STATE.md` | Persistent project context (optional) | Step 7 | If it exists, at session start |

---

## Appendix: Troubleshooting

**The agent isn't following the rules.**
- Ask: "Read CLAUDE.md and confirm you are following all rules. Report any rule you're currently violating."
- If it's a new session, the agent should auto-load `CLAUDE.md`. If it didn't, paste the content manually or reference the file explicitly.

**The agent is over-reading for a small task.**
- Say: "This is a small task. Don't do a full reconnaissance. Read only the files this task touches and proceed."

**The agent is editing without planning.**
- Say: "Stop. You're in non-trivial-change territory. Produce a plan first per CLAUDE.md rules and wait for approval."

**The agent keeps auto-updating PROJECT_STATE.md.**
- Say: "Do not update PROJECT_STATE.md unless I explicitly ask. Propose updates only for architectural changes."

**The agent is running expensive commands without asking.**
- Say: "Stop. You ran an expensive command without approval per CLAUDE.md safety rules. Ask first next time."

**The agent is quoting entire files when showing changes.**
- Say: "Quote only the specific lines that changed, not the surrounding function or file. Per CLAUDE.md execution rules."

---

End of guide. Save this as `docs/agent-setup-guide.md` or wherever you keep project documentation. Once you've completed Steps 1–10, the framework is live and you can start working with your agent under the rigorous workflow.

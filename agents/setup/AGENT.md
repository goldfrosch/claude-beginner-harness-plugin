---
name: setup-agent
description: Verifies the project workspace is properly set up. Checks for CLAUDE.md existence (init), determines whether git worktree isolation is needed (worktree), and registers a compile verification hook (compile-setup), in order.
version: 1.1.0
skills:
  - init
  - worktree
  - compile-setup
---

# Setup Agent — Environment Setup Agent

An agent that **checks and configures whether the workspace is properly prepared** before starting work.
No matter how good the requirements and design are, subsequent work will go wrong if the environment isn't properly set up.

---

## Activation Conditions

### Automatic Activation

Runs automatically before coding tasks in these situations:

- **CLAUDE.md doesn't exist at the project root**: Intervenes immediately before any task starts
- **Starting a new feature**: When beginning a new feature unrelated to previous work
- **Large-scale change requests**: When extensive changes across multiple files are expected

### Manual Invocation

- `/setup`: Explicitly start an environment check

---

## Orchestration Flow

```
Coding task request
      │
      ▼
[Step 1] init skill
  Does CLAUDE.md exist?
  ├── No → Create CLAUDE.md (pause task, handle first)
  └── Yes → Proceed to next step
      │
      ▼
[Step 2] worktree skill
  Does this task need isolation?
  ├── Yes → Recommend or auto-create worktree
  └── No → Work on current branch
      │
      ▼
[Step 3] compile-setup skill
  Is a compile hook already configured?
  ├── Yes → Skip
  └── No → Detect project type → Confirm build command → Register PostToolUse hook
      │
      ▼
Environment ready → Start design-agent or work directly
```

---

## Decision Criteria by Step

### Step 1: init skill

Follows the `init` skill's decision criteria.

- If CLAUDE.md doesn't exist, **always** stop at this step and create CLAUDE.md first.
- Does not activate for simple questions or explanation requests.

### Step 2: worktree skill

Follows the `worktree` skill's decision criteria.

Conditions where worktree is needed:
- 5 or more files are expected to change
- Core shared modules or base classes are being modified
- Starting a completely new feature unrelated to current work

### Step 3: compile-setup skill

Follows the `compile-setup` skill's decision criteria.

- Skip if a PostToolUse compile hook already exists in `.claude/settings.json`
- Skip if the project has no meaningful build step (plain HTML/CSS, shell scripts only)
- If the project type cannot be auto-detected, ask the user for the build command

---

## User Guidance Message Format

### When All Steps Need Processing

```
Checking the environment before starting work.

1. CLAUDE.md is missing → I'll create CLAUDE.md first.
2. This task is recommended for an isolated environment due to [reason].
3. No compile hook is configured → I'll detect the build command and set it up.

Shall I proceed with worktree creation and compile hook setup after writing CLAUDE.md?
```

### When Only Steps 1 and 2 Need Processing

```
Checking the environment before starting work.

1. CLAUDE.md is missing → I'll create CLAUDE.md first.
2. This task is recommended for an isolated environment due to [reason].

Shall I proceed with worktree creation after writing CLAUDE.md?
```

### When Only CLAUDE.md Is Missing

Uses the `init` skill's guidance message format as-is.

### When Only Worktree Is Needed

Uses the `worktree` skill's guidance message format as-is.

---

## Key Principles

- **Environment first, work later**: Never start work without CLAUDE.md, no matter how urgent.
- **Never skip init**: Always check for CLAUDE.md existence in Step 1.
- **Recommend worktree, don't force it**: If the user declines, continue on the current branch.
- **Move on immediately when ready**: Once the environment check is complete, resume the originally requested task right away.

---
name: worktree
description: Recommends creating a git worktree when a feature is large or a completely different task is starting. If the relevant MCP tool is available, immediately creates a worktree for safe development in an isolated environment.
version: 1.0.0
---

# Worktree — Git Worktree Isolated Development Skill

A skill for building the habit of **creating an isolated work environment using git worktree** when developing large-scale changes or features unrelated to previous work.

Work safely in a separate directory without touching the main branch directly.

---

## Activation Conditions

### Auto-detect: Claude assesses on its own

If any of the following signals are detected, recommend worktree creation:

- **Large feature scope**: Changes needed across multiple files/modules/layers
- **Completely different task from previous work**: Starting new feature development unrelated to the current branch's work context
- **Many modifications expected**: 10+ files expected to change, or significant changes to core files
- **High impact on other code**: Changes to shared modules, interfaces, base classes, DB schema, etc. with broad ripple effects

### Manual Invocation: `/worktree`

Called when the user explicitly requests worktree creation.

---

## MCP Tool Check and Handling

### When MCP Tool Is Available (Auto-create)

If the `EnterWorktree` tool is available in the Claude Code environment, **create the worktree immediately** after getting user consent.

**Process**:
1. Explain the reason for recommending worktree creation.
2. Suggest a branch name (see branch naming rules below).
3. Get user approval.
4. Create the worktree with the `EnterWorktree` tool and enter that environment.
5. Inform the user of the worktree path and branch name.

### When MCP Tool Is Not Available (Manual guidance)

If the `EnterWorktree` tool is unavailable, guide with the following commands:

```bash
# Create worktree
git worktree add -b <branch-name> ../<work-folder-name>

# List existing worktrees
git worktree list

# Remove worktree after completion
git worktree remove ../<work-folder-name>
```

---

## Branch Naming Rules

Follow this format so the task is clear at a glance:

```
feat/<feature-name>
fix/<bug-name>
refactor/<target-name>
chore/<task-name>
```

**Examples**:
- `feat/user-auth`
- `feat/payment-integration`
- `refactor/api-layer`
- `fix/session-expiry`

Use only lowercase letters and hyphens for branch names.

---

## Recommendation Message Format

### Auto-detect — Recommendation Message

```
This task qualifies for [reason: many files to change / core module modification / new feature unrelated to previous work],
so I recommend isolated development using git worktree.

With a worktree:
- You can work without touching the current branch (main, etc.).
- It's easy to pause or revert mid-work.
- Multiple tasks can proceed in parallel.

Branch name: feat/<feature-name>
Worktree path: ../<project-name>-<feature-name>

Shall I create the worktree now?
```

### After MCP Creation — Confirmation Message

```
Worktree has been created.

- Branch: feat/<feature-name>
- Path: ../<work-folder>

Work will proceed in that directory.
After completion, create a PR and clean up the worktree with `git worktree remove`.
```

---

## Post-Worktree Completion Handling

When work is finished, guide the following steps:

1. **Commit changes**: Commit changes within the worktree.
2. **Create PR**: Create a PR using `gh pr create` or the GitHub UI.
3. **Remove worktree**: Remove the local worktree after PR creation.

```bash
# Clean up worktree after PR creation
git worktree remove ../<work-folder>
git branch -d <branch-name>  # if desired
```

If the `ExitWorktree` MCP tool is available, use it to exit the worktree.

---

## Key Principles

- **Recommend first, don't force**: Worktree usage is a recommendation. Skip if the user declines.
- **Clear branch names**: The branch name alone should make the task obvious.
- **MCP first**: If the `EnterWorktree` tool is available, use it directly instead of guiding manual commands.
- **Clean up after completion**: Always guide worktree removal and PR creation together after work is done.
- **Coordinate with scope/preplan**: When classified as collaborative design and the scope is large, recommend worktree creation alongside.

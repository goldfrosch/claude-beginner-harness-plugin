---
name: handoff
description: Used when conversation context exceeds 50%. Saves current work state to HANDOFF.md and guides the user to reset context with /clear and resume. If HANDOFF.md already exists and is older than 1 hour or an unrelated task is starting, recommends clearing it and starting fresh.
version: 1.0.0
---

# Handoff — Context Management Skill

A skill to prevent performance degradation when context fills up during long work sessions.
It creates a handoff document so that work can continue seamlessly after a context reset.

---

## Activation Conditions

### Auto-detect: Context usage at 50% or more

Claude estimates context usage based on conversation length, accumulated message count, and cumulative file reads.
This skill triggers when any of the following signals are detected:

- 20+ message exchanges accumulated and work is still in progress
- Heavy context accumulation from repeated large file reads
- Claude judges it is becoming difficult to recall earlier statements
- The system has started context compression

### Manual Invocation: `/handoff`

Called when the user explicitly wants to write a handoff document.

---

## Two Scenarios

### Scenario A: Continuing the same task (resume after context reset)

**Condition**: There is continuity with the current task, and HANDOFF.md either doesn't exist or was created recently (within 1 hour)

**Process**:
1. Write HANDOFF.md in the current conversation.
2. Provide the following guidance to the user:
   - Notify that HANDOFF.md is complete.
   - Guide them to reset context with the `/clear` command.
   - Guide them to request "Read HANDOFF.md and continue from where we left off" after reset.

### Scenario B: New task starting or stale HANDOFF.md detected

**Condition**: Any of the following:
- More than 1 hour has passed since HANDOFF.md's `timestamp`
- The current request is a new task unrelated to what's recorded in HANDOFF.md

**Process**:
1. Inform the user that HANDOFF.md is stale or irrelevant.
2. Recommend deleting or clearing HANDOFF.md.
3. Guide them to reset context with `/clear` and start fresh.

---

## How to Write HANDOFF.md

Use the format below. Save as `HANDOFF.md` at the **project root**.

```markdown
# Work Handoff Document (HANDOFF)

> ⏱ Written at: YYYY-MM-DD HH:MM (local time)
> This document was created to continue work after a context reset.

---

## Current Goal

[Purpose and final deliverable of the work in progress]

---

## Completed Tasks

- [Completed item 1]
- [Completed item 2]
- ...

---

## Failed Tasks

- [Failed item 1]: [Reason for failure or approach attempted]
- [Failed item 2]: [Reason for failure or approach attempted]
- ...

---

## Current Issues

- [Unresolved or blocked issue 1]
- [Unresolved or blocked issue 2]
- ...

---

## Next Steps

1. [Immediate next task]
2. [Following task]
3. ...

---

## How to Resume

After having Claude read this document, say:

> "I've read HANDOFF.md. Continue from [first item in Next Steps]."
```

---

## User Guidance Message Format

### Scenario A — After writing the handoff document

```
Context is getting full. I've written HANDOFF.md to continue the work.

Please follow these steps:

1. Type `/clear` to reset the context.
2. After reset, type:
   "Read HANDOFF.md and continue from [next task]."

HANDOFF.md has been saved at the project root.
```

### Scenario B — Stale HANDOFF.md or new task detected

```
HANDOFF.md exists, but [it's been over 1 hour / it's unrelated to the current request], so I recommend starting fresh.

Please follow these steps:

1. Delete or clear HANDOFF.md.
2. Type `/clear` to reset the context.
3. Start the new task from scratch.

Review the existing HANDOFF.md now if you need its contents.
```

---

## .gitignore Handling

HANDOFF.md is a temporary document created during work and should not be included in version control.

When creating HANDOFF.md for the first time, check if the project has a version control system (.git, .svn, etc.).
If a VCS is detected, **always** add `HANDOFF.md` to the appropriate ignore file.

| VCS | Ignore file |
|-----|-------------|
| Git | `.gitignore` |
| SVN | `.svnignore` |
| Mercurial | `.hgignore` |

Do not add duplicate entries if already registered.
Create the ignore file if it doesn't exist.

---

## Separate Planning/Design Documents

During work, situations may arise where planning content, design decisions, and background explanations need long-term reference.
Such content should be managed as **separate reference documents**, not in HANDOFF.md (which is temporary).

### When to Create a Separate Document

Recommend creating a separate document when any of the following apply:

- Planning decisions or design rationale need to be referenced later
- You want to link a specific method or module to its planning intent
- Consistent context needs to be maintained across multiple sessions

### What to Include

- Planning intent, design decision rationale, business rules
- **Method names** where the content is implemented (clues for finding the implementation)

### What NOT to Include

- Specific method behavior, parameters, implementation logic → Read code and comments directly
- Variable names, return values, internal flow → Code is the most accurate source

### Format Example

```markdown
# [Feature Name] Planning Memo

## Background
[Why this feature is needed, what problem it solves]

## Key Decisions
- [Decision]: [Reason]

## Related Methods
- `methodName()` — [One-line description of the planning intent it embodies]
- `methodName()` — [Same]
```

For specific behavior details, read the method's code and comments directly.

### File Location

Save in the `.claude/docs/` folder. Name files after the feature or domain.
Example: `.claude/docs/payment-flow.md`, `.claude/docs/auth-design.md`

---

## Key Principles

- **Save first, then reset**: Only guide /clear after HANDOFF.md is fully written.
- **Write for continuity**: The handoff document must be immediately understandable by a future Claude with no context.
- **Timestamp is required**: HANDOFF.md must always include the written time. This is the basis for Scenario B decisions.
- **HANDOFF.md goes at the project root**: The location must be consistent so Claude can find it immediately in the next session.
- **Fresh start for new tasks**: Don't force-continue from a stale handoff document. Starting clean is more efficient.
- **Never deploy HANDOFF.md**: Always register it in the ignore file if a VCS exists.
- **Read code from code**: Don't copy implementation details into planning documents. Code and comments are the most accurate source.

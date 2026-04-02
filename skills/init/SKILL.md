---
name: init
description: Used when CLAUDE.md does not exist at the project root. Regardless of the requested task, if CLAUDE.md is missing, stop immediately and prioritize creating it. CLAUDE.md contains a brief project overview and references detailed rules in the .claude folder.
version: 1.0.0
---

# Init — CLAUDE.md Initialization Skill

A skill that **checks whether CLAUDE.md exists in the project** before starting any work.
Without CLAUDE.md, Claude works from guesses without knowing the project's purpose, rules, or structure.
Creating this document first is the most important initial step before any task.

---

## Activation Conditions

### Automatic: When CLAUDE.md is missing

If `CLAUDE.md` does not exist at the project root, **this skill triggers immediately regardless of the type of task requested**.

- Even if a code writing request is received, CLAUDE.md creation takes priority.
- Does not activate for simple questions or explanation requests.
- Even for "let's just start quickly" requests, CLAUDE.md creation is handled first.

### Manual Invocation: `/init`

Called when the user explicitly wants to create or reinitialize CLAUDE.md.

---

## Role and Principles of CLAUDE.md

CLAUDE.md is a document containing the **minimum information Claude needs to know about this project**.

### What to Include
- What the project is (one or two sentences)
- Key tech stack
- Core principles Claude must follow in this project (if any)
- References to detailed rules/guides in the `.claude` folder

### What NOT to Include
- Detailed planning, feature specs, API docs → `.claude` folder or separate documents
- Development conventions, coding style rules → `.claude/rules/` or `.claude/skills/`
- Detailed content that is likely to change frequently

**CLAUDE.md should be short and stable.** Put frequently changing content in `.claude` subdocuments and only reference them from CLAUDE.md.

---

## CLAUDE.md Creation Procedure

### Step 1: Understand the Project

Quickly read existing files and folder structure to understand the project.

Items to check:
- `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc. → Identify tech stack
- `README.md` existence → Reference for project overview
- `.claude/` folder existence → If present, identify subdocument list

### Step 2: Ask Key Questions

Based on findings, **briefly confirm only unknown parts**.

```
Before writing CLAUDE.md, I have a few things to confirm.

1. Can you describe this project in one or two lines?
2. Are there any rules or constraints Claude must know when working?
   (If none, just say so)
3. Are there any documents organized in the .claude folder?
   (If so, briefly describe their contents)
```

Don't re-ask about things already identified (tech stack, README, etc.).

### Step 3: Draft CLAUDE.md and Get Confirmation

Draft in the format below and get user confirmation.

---

## CLAUDE.md Format

```markdown
# Project Name

## Overview
[What this project is in 1-3 sentences]

## Tech Stack
- [Primary language/framework]
- [Database, infrastructure, etc. only if relevant]

## Core Principles
[Principles Claude must follow across the project. Omit this section if none]

## Reference Documents
Refer to the documents below for detailed rules and guides:
- [Document title]: `.claude/path/file.md`
```

**Format Guidelines**:
- `## Core Principles` should contain only truly important items. Better to omit entirely if there are none.
- `## Reference Documents` should list only files that actually exist in the `.claude` folder.
- If the `.claude` folder doesn't exist or is empty, omit the `## Reference Documents` section.

---

## User Guidance Message Format

### When CLAUDE.md Is Missing — Upon Receiving a Task Request

```
Before starting work, I noticed CLAUDE.md is missing.

Without CLAUDE.md, Claude will work based on guesses without knowing this project's context.
I recommend creating CLAUDE.md first before proceeding.

[1-2 confirmation questions]

After creating CLAUDE.md, I'll continue with your requested task right away.
```

### After CLAUDE.md Is Written

```
CLAUDE.md has been written.

Please review it at [file path] and let me know if changes are needed.
Now I'll proceed with your originally requested task.
```

---

## Key Principles

- **Documentation before work**: Never start work without CLAUDE.md, no matter how urgent the request.
- **Keep it short and stable**: CLAUDE.md should contain only content that doesn't change often. Details go in the `.claude` folder.
- **Don't assume**: Confirm the project's purpose and core principles directly with the user.
- **Draft then confirm**: Show the CLAUDE.md draft to the user and get approval before saving.
- **Leverage existing documents**: Don't re-ask about information already available from README, dependency files, etc.

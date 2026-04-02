---
name: preplan
description: Used when a complex task is requested without prior context. Establishes domain terminology and recommends Plan Mode for changes spanning multiple files/modules, new features built on existing modules, and refactoring work.
version: 1.0.0
---

# Preplan — Pre-Work Preparation Skill

A skill for building the habit of **aligning domain language and designing in plan mode before starting complex tasks**.
Jumping straight into a large task without context can lead to results that miss the mark.

---

## Activation Conditions

In the following situations, **do not start coding immediately — proceed through this skill's preparation steps**:

- A complex task is requested early in the conversation without sufficient context
- Terminology used in the request needs to be verified against the service's domain
- One or more of the **plan mode recommendation conditions** below apply

---

## Conditions for Recommending Plan Mode

If any of the following apply, **recommend `/plan` command or plan mode entry**:

- **Feature changes affecting multiple files/modules**: Changes span multiple layers (API, service, DB, UI, etc.)
- **New feature development using existing modules**: Building new functionality by connecting multiple existing modules
- **Refactoring**: Restructuring existing code, extracting shared logic, or reorganizing dependencies
- **Architecture changes**: Modifying foundational design elements like layer structure, module dependency directions, or shared interfaces
- **High recovery cost tasks**: Tasks that are difficult to undo if mistakes are made (e.g., DB schema changes, migrations, data transformations, external system integration changes)

Establishing the design direction in plan mode and getting approval before implementation reduces the cost of mid-course corrections.

---

## Domain Terminology Alignment

Verify that terminology used in the task matches the actual service's language.

### Why It Matters

The same concept can have different names across services.
If Claude writes code using generic terms, it creates inconsistency with the existing codebase and causes confusion.

**Examples**:
- Does this service call a "user" `Member`, `User`, or `Account`?
- Is an "order" called `Order`, `Purchase`, or `Transaction`?
- Is a "post" called `Post`, `Article`, or `Content`?

### How to Verify

The most accurate approach is to check existing code:

1. Extract key concepts (nouns) from the request.
2. Check how those concepts are named in the codebase.
3. If there's a mismatch or no codebase exists, confirm directly with the user.

**Question example**:
> The request mentions "user." What term does this service use for users? (e.g., `User`, `Member`, `Account`)

---

## Preplan Execution Order

1. **Verify domain terminology**: Identify the service-specific names for key concepts in the request.
2. **Assess task nature**: Check if plan mode recommendation conditions apply.
3. **Recommend plan mode (if applicable)**: Suggest plan mode entry using the format below.
4. **Proceed with work**: Start implementation after terminology is aligned and direction is agreed upon.

---

## Plan Mode Recommendation Message Format

```
This task involves [reason: changes across multiple modules / existing module integration / refactoring],
so I recommend establishing the design direction in plan mode first.

In plan mode:
- Files and modules that need changes are identified in advance.
- Implementation order and approach are designed.
- Design direction is confirmed before writing code.

Shall I proceed in plan mode?
```

---

## Key Principles

- **Use the service's terminology**: Follow the actual codebase language, not Claude's generic expressions.
- **Design before big tasks**: Agree on what and how to change before writing code.
- **Don't skip the process**: Aligning direction before starting is ultimately faster than starting quickly.

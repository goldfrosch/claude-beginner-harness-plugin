---
name: clarify
description: Must be used when task requirements are ambiguous or AI would need to make assumptions. When the user requests new features, code, bug fixes, refactoring, or design — especially with vague scope or purpose — do not start coding immediately; use this skill to clarify requirements first. Exception: vibe-coding tasks (one-off/experimental/demo) proceed autonomously without a questionnaire.
version: 1.1.0
---

# Clarify — Requirements Clarification Skill

A skill for building the habit of **clearly defining "what needs to be built"** before writing code.
Starting work with unclear requirements can lead to results that miss the mark.
Answering questions thoroughly before starting is far more efficient.

However, **vibe-coding tasks** are an exception. Presenting a questionnaire for every experiment/demo/one-off piece of code disrupts the flow.

---

## Step 0: Scope Check (Always Runs First)

Before presenting the clarify questionnaire, first determine whether this task is in the **vibe-coding category**.

### If Vibe-Coding → Skip clarify, proceed autonomously

If two or more of the following conditions apply, proceed without a questionnaire:

- **One-off**: Code that won't be reused or has low reuse potential
- **Low importance**: Code where the cost of rebuilding is low even if done wrong
- **Experiment/prototype**: Purpose is idea validation or PoC
- **Independence**: Code not connected to production code or other modules
- **Non-security**: Code unrelated to authentication, authorization, or sensitive data handling

After completing vibe-coding, briefly explain the reasoning behind choices and production caveats alongside the finished code.

### If Collaborative Design → Proceed with clarify process below

If any of the following apply, the questionnaire is mandatory:

- Code intended for production deployment
- Security-related code (authentication, authorization, encryption, sensitive data)
- Performance optimization (hot paths, DB queries, caching strategies)
- Core abstractions (shared interfaces, base classes that other code depends on)
- Data model changes (DB schema, migrations)
- External system integrations

### If Unclear

Ask briefly:

```
The approach differs depending on the purpose of this code.

- Is this for quick experimentation or a demo? (Proceed with vibe-coding)
- Is this code that will be deployed to production? (Proceed after clarifying requirements)
```

---

## Activation Conditions

After **scope determines this is a collaborative design task**, if any of the following apply, **stop coding immediately and present the questionnaire**:

- The request lacks specific behavior, I/O, or scope definitions
- Subjective expressions like "nice", "clean", "improve", "optimize" are included
- AI would need to choose the tech stack, structure, or behavior on its own
- The request can be interpreted in multiple directions
- How it connects to existing code is unclear

---

## Questionnaire Presentation Method

From the categories below, **select only items relevant to the current request** to compose the questionnaire.
Not all questions need to be asked. Focus on the unclear parts.

---

### 1. Purpose and Background

> Why is this needed?

- What problem does this feature/code solve?
- Who will use it? (Users, developers, internal systems, etc.)
- What inconvenience exists without it?

### 2. Input and Output

> What goes in, and what should come out?

- What values come in as input? (Format, range, examples)
- What should be returned/output as a result?
- How should success and failure cases each be handled?

### 3. Scope and Boundaries

> How far should this go?

- What is the scope of this implementation?
- What can be excluded?
- Should future extensibility be considered?

### 4. Technical Constraints

> What environment and constraints apply?

- Are there required languages, frameworks, or libraries?
- Are there coding styles or conventions that must be followed?
- Are there performance, security, or compatibility requirements?

### 5. Relationship to Existing Code

> Is this new code, or a modification of existing code?

- Are there existing files or code to modify?
- How should this connect to other modules/functions?
- Must existing interfaces (function signatures, APIs, etc.) be preserved?

### 6. Edge Cases and Exceptions

> How should unexpected situations be handled?

- What should happen with invalid input?
- How should network errors, empty values, duplicates, etc. be handled?
- What messages or behaviors are needed when failures occur?

### 7. Completion Criteria

> When can we say "it's done"?

- What deliverable marks this task as complete?
- Is testing or verification needed? How?
- Can you describe what the finished result looks like?

---

## How to Present the Questionnaire

1. Select **items that are unclear for the current request** from the categories above.
2. Present the questions as a numbered list all at once.
3. After the user responds, check if any additional clarification is needed.
4. Once requirements are sufficiently specific, **write a summary and get confirmation**.
5. Start coding only after confirmation is received.

---

## Requirements Summary Format

After the Q&A is complete, summarize in the format below and get user confirmation:

```
## Requirements Summary

**Purpose**: [The problem this task solves]
**Deliverable**: [What needs to be built]
**Input**: [Incoming values]
**Output**: [Expected results]
**Scope**: [Included/excluded items]
**Technical constraints**: [Language, framework, constraints]
**Completion criteria**: [How to judge completion]

Shall I proceed with this?
```

Start work once the user confirms.

---

## Key Principles

- **Don't assume**: Always ask about unclear points. Building on guesses means rebuilding later.
- **Ask all at once**: Don't split questions across multiple exchanges. Present them together.
- **Ask only what's relevant**: Not every category needs to be covered. Focus on unclear areas.
- **Summarize then confirm**: Always share a summary and get approval before starting work.

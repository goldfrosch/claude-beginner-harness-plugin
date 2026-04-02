---
name: clarify
description: Must be used when task requirements are ambiguous or AI would need to make assumptions. When the user requests new features, code, bug fixes, refactoring, or design — especially with vague scope or purpose — do not start coding immediately; use this skill to clarify requirements first. Exception: vibe-coding tasks (one-off/experimental/demo) proceed autonomously without a questionnaire.
version: 1.2.0
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

Use the `AskUserQuestion` tool with a single-select question:

```
question: "The approach differs depending on the purpose of this code. Which applies?"
header: "Code purpose"
multiSelect: false
options:
  - label: "Experiment / Demo"
    description: "Quick experimentation or demo — proceed with vibe-coding autonomously"
  - label: "Production"
    description: "Code that will be deployed to production — clarify requirements first"
```

---

## Step 1: Reference Exploration (Always Runs Before Questionnaire)

Before presenting any questions, identify whether a well-known reference exists for this type of request.

### How to run

1. **Identify the domain** from the request (game mechanics, payment flow, notification system, auth, etc.)
2. **Find 2–3 well-known analogues** — services, games, tools, or patterns widely recognized in that domain
3. **Present them as options** via `AskUserQuestion`, with a brief note on the behavioral contract each implies
4. **Ask for the closest match**

### Example call structure

```
AskUserQuestion(
  questions: [
    {
      question: "어떤 방식에 가장 가깝나요?",
      header: "레퍼런스",
      multiSelect: false,
      options: [
        {
          label: "Civilization (hex grid)",
          description: "균일 이동 비용, 6방향, 시야가 육각형 반경으로 계산"
        },
        {
          label: "XCOM",
          description: "이동력 소모, 엄폐 개념, 지형이 이동에 영향"
        },
        {
          label: "Into the Breach",
          description: "턴제, 밀기/당기기 상호작용 중심, 예측 가능한 AI"
        },
        {
          label: "위 항목과 다릅니다 — 직접 설명할게요"
        }
      ]
    }
  ]
)
```

### After selection

- If a reference is chosen: ask one follow-up question — **"이 방식과 다르게 처리하고 싶은 부분이 있나요?"**
- If "직접 설명할게요" is chosen: proceed to the questionnaire below as usual
- If no good analogue exists (truly novel feature): skip this step and proceed to the questionnaire

### Why this works

A reference carries an implicit contract — behavioral assumptions both parties share without needing to enumerate them. The delta question then surfaces only what diverges. This converts unknown unknowns into known ones without requiring the user to read documentation.

---

## Activation Conditions

After **scope determines this is a collaborative design task**, if any of the following apply, **stop coding immediately and run Reference Exploration, then present the questionnaire for remaining unclear points**:

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

Use the `AskUserQuestion` tool to present questions interactively — **not as a plain text list**.

### Rules
- Present **1–4 questions per `AskUserQuestion` call** (tool limit).
- Group related questions together in a single call.
- Use **`multiSelect: false`** (radio) for mutually exclusive choices.
- Use **`multiSelect: true`** (checkbox) for "select all that apply" choices.
- Each question must have **2–4 options** — add an "Other" option only when truly open-ended (the tool appends it automatically).
- If more than 4 questions are needed, make a second `AskUserQuestion` call after the first is answered.

### Workflow
1. **Run Reference Exploration first** (Step 1 above).
2. If a reference was chosen, the delta question covers much of the ambiguity — **skip questionnaire categories already resolved by the reference**.
3. Select **only the still-unclear items** from the categories above.
4. Call `AskUserQuestion` with those questions in interactive form.
5. After the user responds, check if any additional clarification is needed (call again if so).
6. Once requirements are sufficiently specific, **write a summary and get confirmation**.
7. Start coding only after confirmation is received.

### Example call structure
```
AskUserQuestion(
  questions: [
    {
      question: "Who will use this feature?",
      header: "Target user",
      multiSelect: false,
      options: [
        { label: "End users", description: "External users of the product" },
        { label: "Developers", description: "Internal developer tooling" },
        { label: "Internal systems", description: "Backend services or automation" }
      ]
    },
    {
      question: "Which technical constraints apply?",
      header: "Constraints",
      multiSelect: true,
      options: [
        { label: "Must use existing framework", description: "Cannot introduce new dependencies" },
        { label: "Performance-critical", description: "Hot path or high-frequency execution" },
        { label: "Security-sensitive", description: "Handles auth, encryption, or sensitive data" },
        { label: "No special constraints", description: "Standard implementation is fine" }
      ]
    }
  ]
)
```

---

## Requirements Summary Format

After the Q&A is complete, summarize in the format below and get user confirmation:

```
## Requirements Summary

**Reference**: [Chosen reference, or "없음" if none selected]
**Key differences from reference**: [What diverges from the reference — omit if no reference]
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

- **Reference first**: Always try to find a shared reference before asking abstract questions. A reference carries an implicit contract worth more than a dozen clarifying questions.
- **Delta over description**: After a reference is chosen, ask only what differs — not a full description of expected behavior.
- **Don't assume**: Always ask about unclear points. Building on guesses means rebuilding later.
- **Ask all at once**: Don't split questions across multiple exchanges. Present them together.
- **Ask only what's relevant**: Not every category needs to be covered. Focus on unclear areas.
- **Summarize then confirm**: Always share a summary and get approval before starting work.

---
name: impact-check
description: Must be used when the user requests a change to existing code or requirements. Before starting any modification, scan for semantic dependencies and renew the implicit contracts that the change affects. Focus is on proactively defining the problem space, not testing after the fact.
version: 1.0.0
---

# Impact Check — Change Impact Analysis Skill

A skill for building the habit of **explicitly renewing contracts before making changes**.

Code changes are not just surface-level edits. Every change carries implicit assumptions — about how other parts of the system behave, what they expect, and what they guarantee. When those assumptions shift without acknowledgment, bugs emerge not from mistakes, but from unrecognized contract breaks.

This skill surfaces those contracts before the change, aligns on the new contracts, and only then proceeds.

---

## Activation Conditions

**Automatic**: When the user requests a change to existing code, logic, or requirements — especially:
- Behavior changes (how something works, not just what it's called)
- Interface changes (function signatures, API contracts, data shapes)
- Data model changes (schema, relationships, field semantics)
- Logic changes that affect downstream consumers or upstream providers
- Requirement changes that contradict or extend prior clarify agreements

**Manual**: `/impact-check`

**Skip if**: The change is purely cosmetic (rename with no semantic effect, formatting, comments) and has no behavioral dependencies.

---

## Step 1: Change Characterization

Before scanning, understand what kind of change is being requested.

Identify:
1. **What is changing** — the specific code, behavior, or requirement
2. **Change type** — one or more of: interface / behavior / data model / logic / requirement
3. **Stated reason** — why the user wants this change

If the change type or reason is unclear, ask before proceeding:

```
AskUserQuestion(
  questions: [
    {
      question: "What kind of change is this?",
      header: "Change type",
      multiSelect: true,
      options: [
        { label: "Interface change", description: "Function signatures, API contracts, or data shapes" },
        { label: "Behavior change", description: "Existing logic will execute differently" },
        { label: "Data model change", description: "Schema, relationships, or field semantics" },
        { label: "Requirement change", description: "A previously agreed assumption is being revised" }
      ]
    }
  ]
)
```

---

## Step 2: Impact Scan

Scan the codebase to find everything that depends on what's changing.

### Scan sources (in priority order)

1. **Clarify output** — if a Requirements Summary exists (from `/clarify`), use it as the primary contract map. Identify which agreed items are affected by this change.
2. **Code references** — trace direct callers, importers, and data consumers of the changing code.
3. **Implicit indicators** — variable names, comments, type annotations, and structural patterns that suggest semantic coupling.

### Output: Impact Map

Present the impact map clearly before proceeding to contract review:

```
## Impact Map

**변경 대상**: [what is changing]
**변경 유형**: [interface / behavior / data model / logic / requirement]

**영향받는 영역**:
1. [area name] — [how it depends on the changing code]
2. [area name] — [how it depends on the changing code]
...

**clarify 합의 중 영향받는 항목**:
- [agreed item] — [how this change affects it]
```

If no dependencies are found, state that explicitly and proceed without a contract review.

---

## Step 3: Contract Review

For each impacted area, surface the implicit contract — what was assumed before, and what will be assumed after.

Present this as a renewal, not a warning. The goal is not to block the change, but to make the new world explicit before building it.

### Format per impacted area

```
### [Area Name]

**변경 전 전제**: [what this area currently assumes about the changing code]
**변경 후 전제**: [what this area will need to assume after the change]
**충돌 가능성**: [없음 / 낮음 / 높음] — [brief reason]
```

### Presenting the review

Use `AskUserQuestion` to get explicit agreement on the new contracts.
Do not proceed without confirmation.

```
AskUserQuestion(
  questions: [
    {
      question: "The contract for [Area Name] is changing. Does this direction look right?\nBefore: [old assumption]\nAfter: [new assumption]",
      header: "Contract renewal",
      multiSelect: false,
      options: [
        { label: "Yes — proceed with this direction", description: "Accept the new assumption and continue" },
        { label: "No — the scope needs adjustment", description: "Revisit the change scope or approach" }
      ]
    }
  ]
)
```

If multiple impacted areas exist, group related ones into a single `AskUserQuestion` call (max 4 questions per call).

---

## Step 4: Proceed or Adjust

### If all contracts confirmed

Summarize the agreed new contracts and proceed:

```
## Contract Update Summary

**변경 내용**: [what is changing]
**갱신된 전제**:
- [Area 1]: [new assumption]
- [Area 2]: [new assumption]

위 내용으로 진행하겠습니다.
```

### If a conflict is found

Surface the conflict clearly and let the user decide:

```
AskUserQuestion(
  questions: [
    {
      question: "A conflict was detected with [Area]. How would you like to handle it?",
      header: "Conflict resolution",
      multiSelect: false,
      options: [
        { label: "Expand scope — update dependent code too", description: "Extend the change to keep everything consistent" },
        { label: "Narrow scope — adjust this change to avoid the conflict", description: "Limit the change so existing contracts are preserved" },
        { label: "Proceed with the conflict acknowledged", description: "Accept the risk and apply only this change" }
      ]
    }
  ]
)
```

---

## Key Principles

- **Contract renewal over warning**: Don't just flag risks — surface the old contract, the new contract, and get agreement on both before proceeding.
- **Semantic over syntactic**: Compiler errors catch surface conflicts. This skill targets the semantic conflicts that compilers miss.
- **Clarify output is the contract baseline**: If `/clarify` was run, treat its Requirements Summary as the ground truth for what was agreed. Impact check updates that agreement.
- **No silent assumptions**: Every implicit assumption that changes must be made explicit and confirmed. Proceeding without acknowledgment is how unknown unknowns become production bugs.
- **Scope the work, not the risk**: When a conflict is found, the goal is not to avoid the change — it's to decide the right scope so the change lands cleanly.

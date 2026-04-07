---
name: debug
description: Activates when the user shares an error message or stack trace, or when an error occurs during code execution. Instead of immediately fixing the error, guides the habit of analyzing the root cause, understanding why it happened, and preventing recurrence.
version: 1.0.0
---

# Debug — Error Analysis and Learning Skill

A skill for building the habit of **learning from errors, not just fixing them**.

When an error appears, the instinct is to patch the symptom immediately and move on. But errors are information — they reveal gaps between what the code assumes and what is actually happening. This skill slows that instinct down just enough to surface the real cause, agree on the right fix, and leave with understanding that prevents the same mistake from recurring.

---

## Activation Conditions

**Automatic**: When any of the following occur:
- The user shares an error message or stack trace
- Claude executes code and an error is returned
- The user expresses confusion about unexpected behavior ("why isn't this working", "this doesn't make sense", "it breaks when I...")
- Tests fail unexpectedly

**Manual**: `/debug`

**Skip if**: The error is trivial and self-explanatory (e.g., a clear typo with an obvious one-character fix and no risk of misunderstanding the root cause).

---

## Step 1: Error Classification

Before analyzing, identify what kind of error this is.

| Type | Description | Examples |
|------|-------------|---------|
| **Compile / Type** | Caught before runtime — type mismatch, missing symbol, syntax error | TS type errors, import failures, syntax errors |
| **Runtime** | Occurs during execution — unexpected state, null access, out-of-bounds | NullPointerException, undefined is not a function |
| **Test Failure** | Test assertion doesn't match actual behavior | Expected X but received Y |
| **Logic** | Code runs without error but produces wrong output | Off-by-one, wrong formula, incorrect condition |

State the classification before proceeding:

```
**에러 유형**: [Compile / Runtime / Test Failure / Logic]
**에러 메시지**: [the error message or symptom]
```

---

## Step 2: Reproduction Conditions

Confirm the exact conditions that trigger the error.

Ask if unclear:
- What input or state caused this?
- Does it happen every time, or only sometimes?
- When did this start? Did something change recently?

This matters because **intermittent errors and always-reproducible errors have different root causes**. Don't assume.

---

## Step 3: Root Cause Analysis

Separate the **surface symptom** from the **underlying cause**.

A symptom is what the error message says. The root cause is why that condition exists.

Work through the cause chain:

```
증상: [what the error says]
  ↓
직접 원인: [what code condition produced the error]
  ↓
근본 원인: [why that condition exists — the actual problem to fix]
```

**Common root cause patterns to check**:
- Wrong assumption about data shape or nullability
- Misunderstood API contract (returns X, code expected Y)
- State mutation in an unexpected place
- Timing issue (async/await missing, race condition)
- Environment difference (works locally, fails in CI)
- Off-by-one or boundary condition

Present the root cause analysis clearly before moving to a fix.

---

## Step 4: Fix Direction Agreement

Before applying a fix, present the proposed approach and confirm.

```
**근본 원인**: [one-line summary]

**수정 방향**:
[proposed fix — what will change and why]

**대안** (있는 경우):
[alternative approach, if any, with trade-offs]

이 방향으로 수정할까요?
```

Use `AskUserQuestion` when multiple valid approaches exist:

```
AskUserQuestion(
  questions: [
    {
      question: "이 에러를 어떤 방향으로 수정할까요?",
      header: "수정 방향",
      multiSelect: false,
      options: [
        { label: "[Option A]", description: "[trade-off]" },
        { label: "[Option B]", description: "[trade-off]" }
      ]
    }
  ]
)
```

---

## Step 5: Fix and Verify

Apply the fix, then verify:

1. The original error no longer occurs
2. Related behavior hasn't regressed
3. If tests exist, run them

If verification is not possible in the current environment, state what the user should check.

---

## Step 6: Recurrence Prevention Summary and Log

After fixing, briefly explain **why this error happened and how to avoid it**.

This is the most important step for beginners. Don't skip it.

### 6-1. Output the summary to the conversation

```
## 에러 요약

**원인**: [one or two sentences on why this happened]
**수정 내용**: [what was changed]
**앞으로 주의할 점**: [the habit or check that prevents this from recurring]
```

Keep this summary short. The goal is a takeaway the user will actually remember — not a lecture.

### 6-2. Determine scope: team or personal

Before saving, assess whether this error is a **team** pattern or a **personal** pattern.

#### Auto-judgment criteria

| Signal | Scope |
|--------|-------|
| Error is tied to project-specific module, API contract, or business logic | **team** |
| Error reveals a misunderstanding about how the codebase is structured | **team** |
| Error is a language/syntax basics mistake (missing await, wrong operator, etc.) | **personal** |
| Error is a tooling or environment issue (wrong config, missing dependency) | **personal** |
| Error is a general coding habit gap (null check, boundary condition, etc.) | **personal** |

#### Confirm with the user

After assessing, present the recommendation and ask for confirmation:

```
AskUserQuestion(
  questions: [
    {
      question: "이 에러 케이스를 어디에 저장할까요?\n\n추천: [team / personal]\n이유: [one-line reason for the recommendation]",
      header: "저장 범위 선택",
      multiSelect: false,
      options: [
        {
          label: "team — 프로젝트 공유",
          description: "이 프로젝트 코드와 직결된 패턴입니다. 팀원도 같은 함정에 빠질 수 있습니다."
        },
        {
          label: "personal — 개인 보관",
          description: "개인 학습 패턴입니다. git에 포함되지 않습니다."
        }
      ]
    }
  ]
)
```

### 6-3. Save to debug log

Save the case to `.claude/debug-log/` under the confirmed scope.

#### Directory structure

```
.claude/debug-log/
  team/                     ← committed to git (project-wide patterns)
    INDEX.md
    compile/
    runtime/
    test-failure/
    logic/
  personal/                 ← excluded from git (individual learning patterns)
    INDEX.md
    compile/
    runtime/
    test-failure/
    logic/
```

Map the error type from Step 1 to the corresponding subfolder:

| Error Type | Subfolder |
|-----------|--------|
| Compile / Type | `compile/` |
| Runtime | `runtime/` |
| Test Failure | `test-failure/` |
| Logic | `logic/` |

Full path example: `.claude/debug-log/personal/runtime/2026-04-07_14-32-05_async-await-forgotten.md`

#### gitignore handling

When saving a `personal` case for the first time, check if `.claude/debug-log/personal/` is already ignored.

If not, append to the project's ignore file (`.gitignore` if it exists):

```
.claude/debug-log/personal/
```

#### Case file

Filename: `YYYY-MM-DD_HH-MM-SS_[error-keyword].md`
- Use the current date and time for `YYYY-MM-DD_HH-MM-SS`
- For `error-keyword`, use 2–4 words that identify the specific error (e.g., `null-check-missing`, `async-await-forgotten`, `type-mismatch-api`)

File content:

```markdown
# [error-keyword]

**날짜**: YYYY-MM-DD HH:MM:SS
**유형**: [Compile / Runtime / Test Failure / Logic]
**범위**: [team / personal]
**증상**: [what the error message said]

## 근본 원인

[root cause — the actual problem, not just the symptom]

## 수정 내용

[what was changed and why]

## 앞으로 주의할 점

[the habit or pattern check that prevents recurrence]
```

#### INDEX.md

Each scope (`team/` and `personal/`) has its own `INDEX.md`. After saving a case file, append a one-line entry to the relevant category section.

If `INDEX.md` does not exist in the target scope folder, create it with this structure:

```markdown
# Debug Log Index ([team / personal])

에러 케이스를 카테고리별로 정리합니다.
각 케이스 파일에는 근본 원인, 수정 내용, 재발 방지 포인트가 담겨 있습니다.

---

## Compile / Type

<!-- compile error cases -->

## Runtime

<!-- runtime error cases -->

## Test Failure

<!-- test failure cases -->

## Logic

<!-- logic error cases -->
```

Append to the appropriate section in this format:

```
- [YYYY-MM-DD HH:MM:SS] [error-keyword] — [one-line summary of root cause] → [relative path to case file]
```

Example:

```
- [2026-04-07 14:32:05] async-await-forgotten — fetchUser 호출 시 await 누락으로 Promise 객체 반환 → runtime/2026-04-07_14-32-05_async-await-forgotten.md
```

---

## Key Principles

- **Root cause over symptom**: Fixing the symptom without understanding the cause means the same bug returns in a different form.
- **No silent fixes**: Always explain what changed and why. A fix the user doesn't understand doesn't build understanding.
- **Classify before diagnosing**: Different error types require different diagnostic approaches. Misclassifying wastes time.
- **Agree before applying**: Especially for non-trivial fixes, confirm the direction. Surprises in error recovery are worse than surprises in new features.
- **Leave with a takeaway**: Every debugging session should end with something the user now knows that they didn't before.

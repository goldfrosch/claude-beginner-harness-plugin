---
name: compile-setup
description: Detects the project's build command and registers it as a PostToolUse hook in .claude/settings.json, so that compilation is automatically verified after every file edit. Runs during project setup to establish the compile check habit from the start.
version: 1.0.0
---

# Compile Setup — Automatic Compile Verification Skill

A skill that **catches compile errors at the moment they're introduced**, not after the fact.

Beginners often write several files of code before discovering that something doesn't compile. By the time the error surfaces, the context of what changed is already lost. This skill wires up a compile check that runs automatically after every file edit — making broken builds visible immediately, while the change is still fresh.

---

## Activation Conditions

**Automatic**: Runs as part of `setup-agent` during project initialization, after `init` and `worktree`.

**Manual**: `/compile-setup`

**Skip if**:
- A compile hook is already configured in `.claude/settings.json`
- The project has no build step (plain HTML/CSS, shell scripts, interpreted languages where "compile" has no meaning)

---

## Step 1: Detect Project Type

Scan the project root for build configuration files:

| File Found | Project Type | Default Check Command |
|-----------|-------------|----------------------|
| `package.json` | Node.js / JS / TS | `npm run build` or `npx tsc --noEmit` |
| `go.mod` | Go | `go build ./...` |
| `Cargo.toml` | Rust | `cargo check` |
| `pom.xml` | Java (Maven) | `mvn compile -q` |
| `build.gradle` / `build.gradle.kts` | Java/Kotlin (Gradle) | `./gradlew compileJava -q` |
| `*.csproj` / `*.sln` | C# / .NET | `dotnet build -q` |
| `pyproject.toml` / `setup.py` | Python | `python -m py_compile` (limited) |
| `CMakeLists.txt` | C/C++ | `cmake --build build` |

If multiple files are found, ask the user which applies. If none are found, ask directly.

---

## Step 2: Confirm or Customize the Build Command

Present the detected command for confirmation before registering:

```
프로젝트 타입을 감지했습니다: [project type]

파일 수정 후 자동으로 실행할 컴파일 명령어:
  [detected command]

이 명령어로 컴파일 검증을 설정할까요?
다른 명령어가 있으면 알려주세요 (예: npm run type-check, make build).
```

Use `AskUserQuestion` if the command is ambiguous or multiple options exist:

```
AskUserQuestion(
  questions: [
    {
      question: "어떤 명령어로 컴파일 검증을 할까요?",
      header: "빌드 명령어 선택",
      multiSelect: false,
      options: [
        { label: "npm run build", description: "전체 빌드 실행" },
        { label: "npx tsc --noEmit", description: "타입 체크만 (빠름)" },
        { label: "직접 입력", description: "다른 명령어를 사용합니다" }
      ]
    }
  ]
)
```

---

## Step 3: Register the Hook

Add a `PostToolUse` hook to `.claude/settings.json`.

Read the existing file first if it exists, then merge the hook — do not overwrite other settings.

Target structure:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "<confirmed build command>"
          }
        ]
      }
    ]
  }
}
```

If `.claude/settings.json` does not exist, create it with only this content.

If `PostToolUse` already exists, append to the array rather than replacing it.

---

## Step 4: Confirm Setup

After writing the hook, confirm to the user:

```
컴파일 검증 hook이 설정되었습니다.

**명령어**: [command]
**실행 시점**: 파일 수정(Edit / Write / MultiEdit) 후 자동 실행
**설정 파일**: .claude/settings.json

이제부터 파일을 수정할 때마다 자동으로 컴파일을 확인합니다.
에러가 발생하면 즉시 알려드립니다.
```

---

## Step 5: Run Once to Verify

Run the build command once immediately to confirm it works in this environment:

```bash
[confirmed build command]
```

If it fails:
- If the failure is due to pre-existing errors in the project, note that and confirm the hook is still registered correctly
- If the command itself fails (not found, wrong path), help the user correct it before finalizing

---

## Key Principles

- **Confirm before registering**: Never write to settings.json without showing the user what command will be used.
- **Merge, don't overwrite**: Existing hook configuration must be preserved. Only add to it.
- **Run-once verification**: A hook that doesn't work gives false confidence. Always verify the command runs.
- **Fast feedback over thorough build**: If the full build is slow (>30s), recommend a faster type-check command instead — the goal is immediate feedback, not a complete production build.
- **Optional, not forced**: If the user declines or the project has no build step, skip gracefully without affecting other setup steps.

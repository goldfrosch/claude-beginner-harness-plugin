---
name: worktree
description: 기능이 크거나 이전 작업과 완전히 다른 작업을 시작할 때 git worktree 생성을 권고한다. 관련 MCP 도구가 있으면 즉시 worktree를 만들어 격리된 환경에서 안전하게 개발할 수 있도록 안내한다.
version: 1.0.0
---

# Worktree — Git Worktree 격리 개발 스킬

대규모 변경이나 이전 작업과 무관한 새 기능을 개발할 때, **git worktree를 사용해 격리된 작업 환경을 만드는 습관**을 형성하기 위한 스킬입니다.

main 브랜치를 직접 건드리지 않고 별도 디렉토리에서 안전하게 개발할 수 있습니다.

---

## 이 스킬이 활성화되는 조건

### 자동 감지: Claude가 스스로 판단

아래 신호 중 하나라도 감지되면 worktree 생성을 권고합니다:

- **기능 규모가 큰 경우**: 여러 파일/모듈/레이어에 걸쳐 변경이 필요한 기능
- **이전 작업과 완전히 다른 작업**: 현재 브랜치의 작업 맥락과 무관한 새로운 기능 개발 시작
- **수정 사항이 많은 경우**: 10개 이상의 파일 수정이 예상되거나 핵심 파일에 큰 변경이 예상되는 경우
- **다른 코드에 영향이 큰 경우**: 공통 모듈, 인터페이스, 베이스 클래스, DB 스키마 등 파급 효과가 큰 코드 변경

### 수동 호출: `/worktree`

사용자가 명시적으로 worktree 생성을 요청할 때 호출합니다.

---

## MCP 도구 확인 및 처리

### MCP 도구가 있는 경우 (자동 생성)

Claude Code 환경에서 `EnterWorktree` 도구가 사용 가능한 경우, 사용자의 동의를 받은 뒤 **즉시 worktree를 생성**합니다.

**처리 순서**:
1. worktree 생성 권고 이유를 사용자에게 설명합니다.
2. 브랜치명을 제안합니다 (아래 브랜치명 규칙 참고).
3. 사용자의 승인을 받습니다.
4. `EnterWorktree` 도구로 worktree를 생성하고 해당 환경으로 진입합니다.
5. worktree 경로와 브랜치명을 사용자에게 알립니다.

### MCP 도구가 없는 경우 (수동 안내)

`EnterWorktree` 도구를 사용할 수 없는 경우, 아래 명령어를 안내합니다:

```bash
# worktree 생성
git worktree add -b <브랜치명> ../<작업폴더명>

# 생성된 worktree 목록 확인
git worktree list

# 작업 완료 후 worktree 제거
git worktree remove ../<작업폴더명>
```

---

## 브랜치명 규칙

작업 내용을 한눈에 알 수 있도록 다음 형식을 따릅니다:

```
feat/<기능명>
fix/<버그명>
refactor/<대상명>
chore/<작업명>
```

**예시**:
- `feat/user-auth`
- `feat/payment-integration`
- `refactor/api-layer`
- `fix/session-expiry`

브랜치명은 소문자와 하이픈만 사용합니다.

---

## 권고 메시지 형식

### 자동 감지 시 — 권고 메시지

```
이 작업은 [이유: 변경 파일이 많음 / 핵심 모듈 수정 / 이전 작업과 무관한 새 기능]에 해당하여
git worktree를 사용한 격리 개발을 권고합니다.

worktree를 사용하면:
- 현재 브랜치(main 등)를 건드리지 않고 작업할 수 있습니다.
- 중간에 작업을 중단하거나 되돌리기 쉽습니다.
- 여러 작업을 병렬로 진행할 수 있습니다.

브랜치명: feat/<기능명>
worktree 경로: ../<프로젝트명>-<기능명>

worktree를 지금 생성할까요?
```

### MCP로 생성 완료 시 — 안내 메시지

```
worktree를 생성했습니다.

- 브랜치: feat/<기능명>
- 경로: ../<작업폴더>

해당 디렉토리에서 작업을 진행합니다.
작업 완료 후 PR을 올리고, worktree는 `git worktree remove`로 정리합니다.
```

---

## worktree 작업 완료 후 처리

작업이 끝나면 아래 순서를 안내합니다:

1. **변경 사항 커밋**: worktree 내에서 변경 사항을 커밋합니다.
2. **PR 생성**: `gh pr create` 또는 GitHub UI에서 PR을 생성합니다.
3. **worktree 제거**: PR 생성 후 로컬 worktree를 제거합니다.

```bash
# PR 생성 후 worktree 정리
git worktree remove ../<작업폴더>
git branch -d <브랜치명>  # 원하는 경우
```

`ExitWorktree` MCP 도구가 있는 경우 이를 사용해 worktree를 종료합니다.

---

## 중요 원칙

- **먼저 권고, 강제하지 않는다**: worktree 사용은 권고이며, 사용자가 원하지 않으면 건너뛴다.
- **브랜치명은 명확하게**: 무엇을 작업하는 브랜치인지 이름만 봐도 알 수 있어야 한다.
- **MCP 우선**: `EnterWorktree` 도구가 있으면 수동 명령어 안내 대신 직접 생성한다.
- **완료 후 정리**: 작업 완료 후 worktree 제거와 PR 생성을 항상 함께 안내한다.
- **scope/preplan과 연계**: 협업 설계 영역으로 분류되고 규모가 클 경우 worktree 생성을 함께 권고한다.

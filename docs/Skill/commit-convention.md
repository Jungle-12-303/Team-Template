---
name: commit-convention
description: Git 커밋 메시지 작성, 브랜치 전략, PR 워크플로우, merge vs rebase 등 Git 협업 전반을 다루는 스킬. 커밋 메시지 제안이 필요하거나, 브랜치 전략을 결정하거나, PR 설명을 작성하거나, 충돌을 해결하거나, "커밋 메시지", "커밋 컨벤션", "브랜치 전략", "PR 작성"이 언급될 때 활성화.
---

# Git 워크플로우 & 커밋 컨벤션

## 커밋 메시지

### 형식

```text
<type>: <한국어 제목>
```

본문이 필요한 경우:

```text
<type>: <한국어 제목>

<한국어 본문>
```

Breaking Change가 있는 경우:

```text
<type>: <한국어 제목>

<한국어 본문>

BREAKING CHANGE: explain the incompatible change in English
```

### 허용 타입

- `feat`: 사용자 관점의 기능 추가
- `fix`: 버그 수정
- `refactor`: 동작 변화 없이 구조 개선
- `docs`: 문서 변경
- `test`: 테스트 추가 또는 수정
- `chore`: 잡무성 변경, 유지보수
- `style`: 동작에 영향 없는 포맷팅
- `perf`: 성능 개선
- `build`: 빌드 설정, 패키지, 컴파일 구성 변경
- `ci`: CI 설정 변경
- `revert`: 이전 커밋 되돌리기

### 스코프 정책

커밋 제목에 스코프를 사용하지 않는다.
변경된 파일과 한국어 제목으로 영향 범위를 전달한다.

### 제목 규칙

- 한국어로 작성
- 한 줄로 유지
- 마침표로 끝내지 않기
- 행위가 아닌 변경의 결과를 서술
- `수정`, `작업`, `변경`, `업데이트` 같은 모호한 단어 지양
- `자식 노드 비교 순서를 바로잡아`, `타입스크립트 빌드 설정을 추가` 같은 구체적 표현 선호

### 본문 규칙

아래 중 하나라도 해당하면 본문 추가:

- 변경 이유가 명확하지 않을 때
- 영향 범위가 넓을 때
- 마이그레이션 또는 사용 주의사항이 있을 때
- 리뷰어에게 컨텍스트가 필요할 때

한국어로 작성, 간결하고 사실에 기반해 서술.

### 예시

```text
feat: DOM 렌더러 초기 구조를 추가

fix: 자식 노드 재정렬 시 인덱스 계산 오류를 고쳐

docs: README에 타입스크립트 시작 방법을 정리

build: TypeScript 출력 경로와 타입 선언 생성을 설정

refactor: 가상 노드 생성 흐름을 단순화
```

본문 포함 예시:

```text
fix: 자식 노드 재정렬 시 인덱스 계산 오류를 고쳐

키 비교 후 재배치 순서를 다시 계산하도록 바꿔
중첩 목록 갱신에서 잘못된 DOM 이동이 발생하지 않게 한다
```

Breaking Change 예시:

```text
feat: 렌더러 초기화 API를 단순화

기본 사용 흐름을 하나로 맞추기 위해 진입 함수를 통합한다

BREAKING CHANGE: replace createRenderer() with createRoot()
```

### 응답 스타일

커밋 후보를 바로 복사할 수 있는 텍스트 블록으로 반환한다.
필요 시 `이유:` 한 줄을 추가한다.
요약이 모호한 경우 최선의 가정을 하고 간략히 명시한다.

---

## 브랜치 전략

### 우리 팀의 브랜치 구조

```
main (릴리즈 전용 — 항상 동작하는 코드만)
 │
 │  ← PR 머지 (dev → main, 릴리즈 태그 부여)
 │
dev (통합 브랜치 — 팀의 중간 저장소)
 │
 ├── dev/woonyong     ← 개인 작업 브랜치 (PR → dev)
 ├── dev/jihye        ← 개인 작업 브랜치 (PR → dev)
 ├── dev/minsoo       ← 개인 작업 브랜치 (PR → dev)
 │
 └── hotfix/설명      ← 긴급 수정 (main에서 분기 → main+dev 머지)
```

### 브랜치 규칙

- `main`은 직접 커밋/push 금지. PR을 통해서만 머지.
- `dev`도 직접 push 금지. 개인 브랜치에서 PR을 통해서만 머지.
- 개인 브랜치(`dev/<이름>`)는 반드시 dev에서 분기.
- hotfix 브랜치는 반드시 main에서 분기, 수정 후 main과 dev 양쪽에 PR.

### 흐름 요약

1. `dev`에서 `dev/<이름>` 브랜치 생성
2. 작업 후 `dev/<이름>` → `dev`로 **PR** 생성
3. 리뷰 후 머지
4. dev에서 기능이 완성되고 동작 확인 → `dev` → `main`으로 **PR** 생성 + 릴리즈 태그
5. main에서 긴급 버그 발생 → `hotfix/<설명>` 분기 → main PR + dev PR

---

## Merge 정책

### 핵심 원칙: 모든 머지는 PR을 통해서만

```
[X] git merge → git push (직접 머지 금지)
[O] GitHub에서 PR 생성 → 리뷰 → Merge 버튼 클릭
```

이 규칙은 main, dev 모두에 적용된다.
직접 merge 명령어로 dev나 main에 push하지 않는다.

### 머지 방식: Merge Commit 사용 (팀 기본)

```bash
# 로컬에서 직접 머지하지 않음.
# GitHub PR의 "Merge pull request" 버튼을 사용.
# Squash merge나 Rebase merge는 사용하지 않음.
```

Merge Commit을 쓰면 "누가 언제 어떤 브랜치를 머지했는지" 히스토리가 남아
초보자도 흐름을 추적하기 쉽다.

### Rebase — 초보자는 사용하지 않기

Rebase는 히스토리를 깔끔하게 만들지만, 실수하면 복구가 어렵다.
팀이 Git에 익숙해질 때까지 rebase는 사용하지 않는다.

```
[X] git rebase (당분간 사용 금지)
[O] git merge (안전한 방식)
[O] GitHub PR (가장 안전한 방식)
```

---

## Pull Request

### 핵심 규칙

```
[O] 모든 머지는 반드시 PR을 통해 진행
[O] PR 없이는 dev에도, main에도 머지할 수 없음
[O] 최소 1명 이상의 리뷰 후 머지 (셀프 머지 금지)
```

### PR 종류와 흐름

| PR 종류 | 방향 | 리뷰어 | 용도 |
|---------|------|--------|------|
| 일반 PR | `dev/<이름>` → `dev` | 팀원 1명 이상 | 일상적인 기능/수정 머지 |
| 릴리즈 PR | `dev` → `main` | 팀 전체 합의 | 안정 버전 릴리즈 |
| 핫픽스 PR | `hotfix/<설명>` → `main` | 가능한 빨리 1명 | 긴급 수정 |
| 핫픽스 동기화 | `hotfix/<설명>` → `dev` | 자동 또는 1명 | 핫픽스를 dev에 반영 |

### PR 제목 형식

커밋 컨벤션과 동일한 형식:

```
<type>: <한국어 설명>

예시:
feat: 알람 클록 sleep/wakeup 메커니즘 구현
fix: 타이머 인터럽트에서 tick 비교 오류를 수정
docs: README에 빌드 방법을 추가
```

### PR 설명 템플릿 — 일반 PR (dev/<이름> → dev)

```markdown
## 무엇을 변경했나요?

변경 내용을 간단히 설명.

## 왜 변경했나요?

변경 동기와 맥락 설명.

## 주요 변경 파일

- `파일1.c` — 변경 요약
- `파일2.h` — 변경 요약

## 테스트

- [ ] 빌드 성공 (`make` 통과)
- [ ] 관련 테스트 통과
- [ ] 수동 동작 확인

## 셀프 체크리스트

- [ ] 커밋 메시지가 컨벤션을 따르는가
- [ ] 코딩 스타일 가이드를 준수했는가
- [ ] 불필요한 디버그 출력(printf 등)을 제거했는가
- [ ] 충돌 없이 dev에 머지 가능한가
```

### PR 설명 템플릿 — 릴리즈 PR (dev → main)

```markdown
## 릴리즈 버전

v0.1.0

## 포함된 변경사항

- feat: 알람 클록 구현 (#3)
- fix: 타이머 오버플로우 수정 (#5)
- docs: README 업데이트 (#7)

## 테스트 결과

- [ ] 전체 빌드 성공
- [ ] 전체 테스트 통과
- [ ] 팀 전체 동의
```

### PR 올리는 절차 (처음부터 끝까지)

```bash
# 1. dev 최신화를 내 브랜치에 반영
git checkout dev
git pull origin dev
git checkout dev/woonyong
git merge dev
# 충돌 있으면 해결 → git add → git commit

# 2. 내 브랜치 push
git push origin dev/woonyong

# 3. GitHub 웹에서 PR 생성
#    Base: dev ← Compare: dev/woonyong
#    제목과 본문을 템플릿에 맞게 작성
#    리뷰어 지정

# 4. 리뷰 피드백 반영 (필요시)
#    코드 수정 → commit → push (PR에 자동 반영됨)

# 5. 승인 후 GitHub에서 "Merge pull request" 클릭
#    [X] 로컬에서 git merge 하지 않음
```

### 리뷰어 가이드 (리뷰하는 법)

- 코드가 빌드되는지 확인
- 커밋 메시지가 컨벤션을 따르는지 확인
- 이해가 안 되는 부분은 질문 (비판이 아닌 질문으로)
- 사소한 것(오타, 포맷)은 "nit:" 접두사로 표시
- 좋은 코드에는 칭찬도 남기기

---

## 브랜치 명명 규칙

```
# 개인 작업 브랜치 — dev/<이름>
dev/woonyong
dev/jihye
dev/minsoo

# 핫픽스 브랜치 — hotfix/<간단한-설명>
hotfix/timer-overflow
hotfix/login-crash
hotfix/null-pointer-in-scheduler
```

사용하지 않는 브랜치 형식:
```
[X] feature/...      (개인 브랜치로 통합)
[X] fix/...          (개인 브랜치에서 작업)
[X] release/...      (dev → main PR로 대체)
```

---

## 자주 쓰는 Git 명령어

```bash
# 개인 브랜치 생성 (dev에서 분기)
git checkout dev
git pull origin dev
git checkout -b dev/woonyong

# dev 최신화 반영 (매일 아침)
git checkout dev && git pull origin dev
git checkout dev/woonyong && git merge dev

# 작업 임시 저장
git stash push -m "WIP: 타이머 작업 중"
git stash pop

# 실수 되돌리기
git reset --soft HEAD~1   # 마지막 커밋 취소 (변경사항 유지)
git revert HEAD           # 공개 브랜치에서 커밋 되돌리기

# 머지 취소 (충돌 중)
git merge --abort
```

---

## 안티패턴

```
[X] main이나 dev에 직접 push (반드시 PR을 통해)
[X] PR 없이 로컬에서 git merge로 dev/main에 머지
[X] .env, API 키 커밋
[X] 1000줄 이상의 거대한 PR
[X] "update", "fix", "수정" 같은 모호한 커밋 메시지
[X] 공개 브랜치(dev, main)에 force push
[X] 다른 사람의 dev/<이름> 브랜치에 push
[X] rebase 사용 (팀이 익숙해질 때까지)
[X] dist/, node_modules/, build/ 커밋
```

---

## 빠른 참조

| 작업 | 명령어 |
|------|--------|
| 개인 브랜치 생성 | `git checkout -b dev/<이름>` |
| dev 최신화 반영 | `git checkout dev && git pull && git checkout dev/<이름> && git merge dev` |
| 히스토리 보기 | `git log --oneline --graph` |
| 변경사항 확인 | `git diff` |
| 스테이징 | `git add -p` |
| 커밋 | `git commit -m "type: 한국어 제목"` |
| 푸시 | `git push origin dev/<이름>` |
| 머지 | GitHub PR → Merge 버튼 (로컬 머지 금지) |
| 스태시 | `git stash push -m "설명"` |
| 커밋 취소 | `git reset --soft HEAD~1` |
| 머지 취소 | `git merge --abort` |

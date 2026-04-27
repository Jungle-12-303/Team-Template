---
name: git-workflow
description: 초보 개발자 팀을 위한 Git 워크플로우 종합 스킬. 브랜치 전략, 일상 작업 흐름, 머지, 충돌 해결, 릴리즈 절차, 커밋 컨벤션, 실수 복구를 다룬다. "git", "브랜치", "머지", "충돌", "conflict", "push", "pull", "rebase", "PR", "릴리즈", "hotfix", "커밋 메시지" 등이 언급되면 활성화.
---

# Git 워크플로우 — 초보 개발자 팀 가이드

> 이 문서는 Git에 아직 익숙하지 않은 팀원들이 **안전하게 협업**할 수 있도록 만든 실전 가이드입니다.
> 모든 명령어는 "왜 이걸 치는지"를 함께 설명합니다. 외우지 말고, 필요할 때 찾아보세요.

---

## 1. 브랜치 구조

### 한눈에 보는 전체 그림

```
main (릴리즈 전용 — 항상 동작하는 코드만)
 │
 │  ← 릴리즈 머지 (dev → main, 태그 부여)
 │
dev (통합 브랜치 — 팀의 중간 저장소)
 │
 ├── dev/woonyong    ← 우녕의 작업 브랜치
 ├── dev/jihye       ← 지혜의 작업 브랜치
 ├── dev/minsoo      ← 민수의 작업 브랜치
 │
 └── hotfix/login-crash  ← 긴급 수정 (main에서 분기)
```

### 각 브랜치의 역할

| 브랜치 | 목적 | 누가 머지하나 | 규칙 |
|--------|------|---------------|------|
| `main` | 릴리즈된 안정 버전 | 팀장 또는 합의 후 | 직접 커밋 금지, PR로만 머지 |
| `dev` | 팀 통합 브랜치 | 각자 PR 후 머지 | 항상 빌드 가능한 상태 유지 |
| `dev/<이름>` | 개인 작업 공간 | 본인만 | 자유롭게 커밋, dev에 PR |
| `hotfix/<설명>` | 긴급 버그 수정 | 담당자 | main에서 분기 → main 머지 → dev 머지 |

### 브랜치 생성 규칙

```bash
# 개인 브랜치 — 반드시 dev에서 분기
git checkout dev
git pull origin dev
git checkout -b dev/woonyong

# 핫픽스 브랜치 — 반드시 main에서 분기
git checkout main
git pull origin main
git checkout -b hotfix/login-crash
```

### 절대 하지 말 것

```
[X] main에 직접 push
[X] 다른 사람의 dev/<이름> 브랜치에 push
[X] dev를 거치지 않고 main에 머지
[X] hotfix를 dev에만 머지하고 main에 안 머지하기
```

---

## 2. 일상 워크플로우 — 매일 반복하는 루틴

### 아침: 작업 시작

```bash
# 1. dev 최신화 (다른 팀원이 밤새 머지했을 수 있음)
git checkout dev
git pull origin dev

# 2. 내 브랜치로 이동
git checkout dev/woonyong

# 3. dev의 최신 변경사항을 내 브랜치에 반영
git merge dev
# 충돌이 나면 → 섹션 3 "머지 & 충돌 해결" 참고
```

**왜 매일 해야 하나요?**
안 하면 내 브랜치가 dev와 점점 멀어져서 나중에 머지할 때 충돌이 폭발합니다.
매일 조금씩 합치면 충돌이 작고, 해결도 쉽습니다.

### 작업 중: 커밋하기

```bash
# 변경 확인
git status
git diff

# 파일 단위로 스테이징 (무엇을 커밋할지 고르기)
git add src/thread.c
git add include/thread.h

# 또는 변경된 줄 단위로 고르기 (고급, 추천)
git add -p

# 커밋 (컨벤션에 맞게)
git commit -m "feat: 스레드 우선순위 비교 함수를 추가"
```

**커밋 타이밍:**
- 하나의 논리적 변경이 완성될 때마다
- "컴파일이 되는 상태"에서 커밋
- 절대 하루치를 한 번에 커밋하지 않기

### 퇴근 전: 푸시하기

```bash
# 내 브랜치를 원격에 백업
git push origin dev/woonyong
```

**왜 매일 푸시?** 로컬에만 있으면 컴퓨터가 고장나면 작업이 사라집니다.

### 기능 완성: dev에 머지 요청

```bash
# 1. 머지 전 dev 최신화를 내 브랜치에 반영
git checkout dev
git pull origin dev
git checkout dev/woonyong
git merge dev
# 충돌 해결 후...

# 2. 푸시
git push origin dev/woonyong

# 3. GitHub에서 Pull Request 생성
#    dev/woonyong → dev
#    PR 제목: "feat: 스레드 우선순위 스케줄링 구현"
```

---

## 3. 머지 & 충돌 해결 — 완전 가이드

> **충돌은 실패가 아닙니다.** 두 사람이 같은 부분을 수정했다는 신호일 뿐입니다.
> 침착하게 하나씩 해결하면 됩니다.

### 3.1 충돌은 왜 발생하나?

```
나(dev/woonyong):  line 42를 "A"로 수정
팀원(dev/jihye):   line 42를 "B"로 수정 → dev에 먼저 머지

→ 내가 dev를 머지할 때: Git이 "둘 다 42번 줄을 바꿨는데, 뭘 써야 해?" → 충돌!
```

충돌이 나는 상황과 안 나는 상황:

| 상황 | 충돌? |
|------|-------|
| 서로 다른 파일을 수정 | [O] 자동 머지됨 |
| 같은 파일, 다른 줄을 수정 | [O] 자동 머지됨 |
| 같은 파일, 같은 줄을 수정 | [X] **충돌 발생** |
| 한 명이 파일 삭제, 한 명이 수정 | [X] **충돌 발생** |
| 같은 파일 이름으로 동시 생성 | [X] **충돌 발생** |

### 3.2 충돌 해결 — 단계별 완전 가이드

#### Step 0: 머지 시도

```bash
git checkout dev/woonyong
git merge dev
```

충돌이 없으면 자동으로 머지됩니다. 충돌이 있으면 아래 메시지가 뜹니다:

```
Auto-merging src/thread.c
CONFLICT (content): Merge conflict in src/thread.c
Auto-merging src/timer.c
CONFLICT (content): Merge conflict in src/timer.c
Automatic merge failed; fix conflicts and then commit the result.
```

**이 시점에서 당황하지 마세요.** 아직 아무것도 망가지지 않았습니다.

#### Step 1: 충돌 파일 확인

```bash
# 충돌 파일 목록 보기
git status
```

출력:
```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   src/thread.c
        both modified:   src/timer.c
```

`both modified`라고 표시된 파일이 충돌 파일입니다.

#### Step 2: 충돌 파일 열어서 확인

충돌 파일을 에디터로 열면 이런 마커가 보입니다:

```c
void
thread_sleep (int64_t wakeup_tick) {
    struct thread *curr = thread_current ();
<<<<<<< HEAD
    /* 내가 작성한 코드 (현재 내 브랜치) */
    curr->wakeup_tick = wakeup_tick;
    list_insert_ordered (&sleep_list, &curr->elem,
                         wakeup_tick_less, NULL);
=======
    /* 상대방이 작성한 코드 (dev에서 가져온 것) */
    curr->sleep_ticks = wakeup_tick;
    list_push_back (&sleep_list, &curr->elem);
>>>>>>> dev
    thread_block ();
}
```

**마커 읽는 법:**

```
<<<<<<< HEAD
    (내 코드 — 현재 브랜치의 내용)
=======
    (상대방 코드 — 머지 대상 브랜치의 내용)
>>>>>>> dev
```

#### Step 3: 어떤 코드를 살릴지 결정

**세 가지 선택지:**

**선택 A — 내 코드만 살리기:**
```c
    curr->wakeup_tick = wakeup_tick;
    list_insert_ordered (&sleep_list, &curr->elem,
                         wakeup_tick_less, NULL);
```

**선택 B — 상대방 코드만 살리기:**
```c
    curr->sleep_ticks = wakeup_tick;
    list_push_back (&sleep_list, &curr->elem);
```

**선택 C — 둘 다 합치기 (가장 흔함):**
```c
    curr->wakeup_tick = wakeup_tick;           /* 내 변수명 사용 */
    list_insert_ordered (&sleep_list, &curr->elem,  /* 내 정렬 삽입 사용 */
                         wakeup_tick_less, NULL);
```

**핵심: 마커(`<<<<<<<`, `=======`, `>>>>>>>`)를 반드시 전부 삭제해야 합니다.**

잘못된 해결 (마커가 남아있음):
```c
<<<<<<< HEAD
    curr->wakeup_tick = wakeup_tick;
=======
>>>>>>> dev
```

올바른 해결 (마커 완전 제거, 원하는 코드만 남김):
```c
    curr->wakeup_tick = wakeup_tick;
```

#### Step 4: 해결한 파일을 스테이징

```bash
# 충돌을 해결한 파일을 Git에 "해결했어"라고 알려주기
git add src/thread.c
git add src/timer.c
```

#### Step 5: 모든 충돌이 해결됐는지 확인

```bash
git status
```

`Unmerged paths`가 사라지고 `Changes to be committed`만 보이면 성공:
```
Changes to be committed:
        modified:   src/thread.c
        modified:   src/timer.c
```

아직 `Unmerged paths`가 보이면 해당 파일의 충돌을 더 해결해야 합니다.

#### Step 6: 머지 커밋

```bash
git commit
# 자동으로 "Merge branch 'dev' into dev/woonyong" 메시지가 생성됨
# 그대로 저장하고 닫기
```

#### Step 7: 확인

```bash
# 머지가 잘 됐는지 로그 확인
git log --oneline --graph -10

# 빌드/테스트
make        # 또는 프로젝트의 빌드 명령어
```

### 3.3 충돌 해결 중 "모르겠다, 원래대로 돌리고 싶다"

**머지를 취소하고 충돌 전 상태로 되돌리기:**

```bash
# 머지 중이라면 (아직 commit 전)
git merge --abort
```

이 명령어를 치면 머지 시도 자체가 취소되고, 충돌 전 깨끗한 상태로 돌아갑니다.
그 후 팀원과 상의해서 다시 시도하면 됩니다.

### 3.4 충돌이 너무 많을 때 — 파일 단위 전략

충돌 파일이 10개 이상이면 한 번에 다 하려고 하지 말고:

```bash
# 1. 충돌 파일 목록 확인
git diff --name-only --diff-filter=U

# 2. 한 파일씩 해결
#    쉬운 파일(내가 안 건드린 파일)부터 시작

# 상대방 버전을 그대로 쓸 파일 (내가 안 고친 파일)
git checkout --theirs src/timer.c
git add src/timer.c

# 내 버전을 그대로 쓸 파일 (상대방이 안 고친 파일)
git checkout --ours src/thread.h
git add src/thread.h

# 둘 다 고친 파일 — 직접 에디터로 열어서 수동 해결
code src/thread.c    # 또는 vim src/thread.c
# 해결 후
git add src/thread.c
```

**`--ours` vs `--theirs` 기억법:**
- `--ours` = "내 거" (현재 체크아웃된 브랜치)
- `--theirs` = "상대 거" (머지 대상 브랜치)

### 3.5 같은 충돌이 반복될 때

같은 파일에서 계속 충돌이 나면:

```bash
# rerere 활성화 — Git이 충돌 해결 패턴을 기억함
git config --global rerere.enabled true
```

한 번 해결한 충돌 패턴을 Git이 기억해서 다음에 같은 충돌이 나면 자동으로 해결합니다.

### 3.6 IDE에서 충돌 해결하기

터미널이 어려우면 IDE의 머지 도구를 사용하세요:

**VS Code:**
1. 충돌 파일을 열면 마커 위에 버튼이 나타남
2. `Accept Current Change` (내 것), `Accept Incoming Change` (상대 것), `Accept Both Changes` (둘 다) 클릭
3. 저장 → `git add` → `git commit`

**IntelliJ / CLion:**
1. `Git → Resolve Conflicts` 메뉴
2. 3-way 머지 에디터에서 좌(내 것) / 우(상대 것) / 중앙(결과) 비교
3. 화살표 버튼으로 원하는 쪽 선택
4. Apply → Commit

### 3.7 충돌 예방 — 가장 중요한 습관

```
[O] 매일 아침 dev를 pull하고 내 브랜치에 merge
[O] 작은 단위로 자주 커밋하고 PR 올리기
[O] 같은 파일을 두 명이 동시에 수정하지 않도록 사전 합의
[O] 큰 리팩터링은 팀에 미리 공유
[O] PR이 오래 열려있지 않게 빠르게 리뷰/머지

[X] 일주일치 작업을 한 번에 머지 → 충돌 지옥
[X] dev를 안 당기고 혼자 작업 → 점점 멀어짐
[X] 충돌이 무서워서 머지를 미룸 → 더 심해짐
```

### 3.8 충돌 해결 치트시트

```
┌─────────────────────────────────────────────────────┐
│              충돌이 발생했다!                          │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. 당황하지 말 것. 아무것도 망가지지 않았다.            │
│                                                     │
│  2. git status → 충돌 파일 확인                       │
│                                                     │
│  3. 파일 열기 → <<<<, ====, >>>> 마커 찾기             │
│                                                     │
│  4. 어떤 코드를 남길지 결정                             │
│     - 내 것만? 상대 것만? 둘 다 합치기?                 │
│     - 모르겠으면 → 상대방에게 물어보기                   │
│                                                     │
│  5. 마커 전부 삭제 + 원하는 코드만 남기기                │
│                                                     │
│  6. git add <파일>                                   │
│                                                     │
│  7. 모든 충돌 해결? → git commit                      │
│                                                     │
│  8. 포기하고 싶다? → git merge --abort                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 4. 릴리즈 절차

### 4.1 dev → main 릴리즈

dev에서 기능이 완성되고 테스트를 통과하면 릴리즈합니다.

```bash
# 1. dev가 최신인지 확인
git checkout dev
git pull origin dev

# 2. 빌드 & 테스트 통과 확인
make clean && make
# 테스트 실행...

# 3. main으로 이동
git checkout main
git pull origin main

# 4. dev를 main에 머지
git merge dev

# 5. 버전 태그 부여
git tag -a v1.0.0 -m "첫 번째 릴리즈: 스레드 스케줄링 구현"

# 6. 푸시 (코드 + 태그)
git push origin main
git push origin v1.0.0
```

### 4.2 버전 태그 규칙

```
v<주 버전>.<부 버전>.<패치>

v1.0.0  — 첫 릴리즈
v1.1.0  — 기능 추가 (하위 호환)
v1.1.1  — 버그 수정
v2.0.0  — 큰 변경 (하위 호환 깨짐)
```

프로젝트 초기에는 단순하게:
```
v0.1.0  — 알람 클록 구현
v0.2.0  — 우선순위 스케줄링 추가
v0.3.0  — 우선순위 기부 추가
v1.0.0  — Project 1 최종 제출
```

### 4.3 핫픽스 절차

main에 올라간 버전에서 긴급 버그 발견 시:

```bash
# 1. main에서 핫픽스 브랜치 생성
git checkout main
git pull origin main
git checkout -b hotfix/timer-overflow

# 2. 버그 수정 & 커밋
git add src/timer.c
git commit -m "fix: 타이머 오버플로우로 인한 무한 루프를 수정"

# 3. 테스트 통과 확인
make clean && make
# 테스트 실행...

# 4. main에 머지
git checkout main
git merge hotfix/timer-overflow
git tag -a v1.0.1 -m "hotfix: 타이머 오버플로우 수정"
git push origin main
git push origin v1.0.1

# 5. dev에도 머지 (핫픽스를 dev에 반영)
git checkout dev
git pull origin dev
git merge hotfix/timer-overflow
git push origin dev

# 6. 핫픽스 브랜치 삭제
git branch -d hotfix/timer-overflow
git push origin --delete hotfix/timer-overflow
```

**핵심: 핫픽스는 반드시 main과 dev 양쪽에 머지합니다.**
dev에 안 머지하면 다음 릴리즈 때 같은 버그가 다시 나타납니다.

---

## 5. 커밋 컨벤션

> 상세한 내용은 `commit-convention.md`를 참고하세요. 여기서는 핵심만 정리합니다.

### 형식

```
<type>: <한국어 제목>
```

### 타입

| 타입 | 의미 | 예시 |
|------|------|------|
| `feat` | 기능 추가 | `feat: 스레드 우선순위 비교 함수를 추가` |
| `fix` | 버그 수정 | `fix: 타이머 인터럽트에서 tick 비교 오류를 수정` |
| `refactor` | 구조 개선 (동작 변화 없음) | `refactor: sleep_list 순회 로직을 단순화` |
| `docs` | 문서 변경 | `docs: README에 빌드 방법을 추가` |
| `test` | 테스트 | `test: 우선순위 역전 시나리오 테스트를 추가` |
| `chore` | 잡무 | `chore: .gitignore에 빌드 산출물을 추가` |
| `style` | 포맷팅 (동작 무관) | `style: thread.c 인덴트를 탭으로 통일` |

### 좋은 커밋 vs 나쁜 커밋

```
[O] feat: alarm clock의 sleep/wakeup 메커니즘을 구현
[O] fix: 자식 노드 재정렬 시 인덱스 계산 오류를 수정
[O] refactor: 불필요한 전역 변수를 제거하고 지역 변수로 교체

[X] 수정
[X] 작업 중
[X] update
[X] asdf
[X] 여러 가지 수정함
```

### 커밋 단위 가이드

```
[O] 한 커밋 = 하나의 논리적 변경
    "timer_sleep 함수 구현" → 1커밋
    "thread_sleep 함수 구현" → 1커밋
    "두 함수 연결 및 테스트" → 1커밋

[X] 한 커밋에 여러 변경
    "timer_sleep, thread_sleep, 테스트 전부" → 이건 3커밋으로 쪼개야 함
```

---

## 6. 자주 하는 실수 & 복구법

### 실수 1: 커밋 메시지를 잘못 썼다

```bash
# 마지막 커밋 메시지 수정 (아직 push 안 했을 때만!)
git commit --amend -m "fix: 올바른 메시지로 수정"
```

[!] 이미 push한 커밋은 `--amend` 하지 마세요. `revert`를 사용하세요.

### 실수 2: 잘못된 파일을 커밋했다

```bash
# 마지막 커밋을 취소하고 변경사항은 유지
git reset --soft HEAD~1

# 잘못된 파일을 빼고 다시 커밋
git reset HEAD 잘못된파일.txt
git commit -m "feat: 올바른 파일만 커밋"
```

### 실수 3: 커밋을 안 하고 브랜치를 바꿔야 할 때

```bash
# 작업 중인 변경사항을 임시 저장
git stash push -m "WIP: 타이머 작업 중"

# 다른 브랜치로 이동
git checkout dev/jihye

# 볼 일 다 보고 돌아와서 복원
git checkout dev/woonyong
git stash pop
```

### 실수 4: 잘못된 브랜치에서 작업했다

```bash
# 커밋 전이라면: 변경사항을 stash → 올바른 브랜치에서 pop
git stash
git checkout dev/woonyong
git stash pop

# 이미 커밋했다면 (1개 커밋):
git log --oneline -3     # 커밋 해시 확인 (예: abc1234)
git checkout dev/woonyong
git cherry-pick abc1234  # 해당 커밋을 내 브랜치로 복사
git checkout 잘못된브랜치
git reset --hard HEAD~1  # 잘못된 브랜치에서 커밋 제거
```

### 실수 5: push한 걸 되돌리고 싶다

```bash
# revert — 되돌리는 "새 커밋"을 만듦 (안전한 방법)
git revert HEAD
git push origin dev/woonyong

# [!] force push는 절대 공유 브랜치(dev, main)에 하지 마세요
# 내 개인 브랜치에서만 허용:
git push --force-with-lease origin dev/woonyong
```

### 실수 6: git pull 했더니 내 작업이 사라진 것 같다

사라진 게 아니라 머지가 꼬인 것입니다:

```bash
# 최근 변경 이력 확인
git log --oneline --graph -20

# reflog에서 이전 상태 찾기 (Git의 타임머신)
git reflog

# 원하는 시점으로 되돌리기
git reset --hard HEAD@{3}    # reflog에서 찾은 번호
```

**`git reflog`는 최후의 보루입니다.** Git에서 한 모든 작업의 기록을 보여줍니다.
커밋한 적이 있다면 거의 100% 복구 가능합니다.

### 실수 7: .env 파일이나 비밀번호를 커밋했다

```bash
# 1. 해당 파일을 .gitignore에 추가
echo ".env" >> .gitignore

# 2. Git 추적에서 제거 (파일은 유지)
git rm --cached .env

# 3. 커밋
git commit -m "chore: .env를 Git 추적에서 제거"

# 4. 이미 push했다면 → 비밀번호/키 즉시 변경
#    Git 히스토리에 남아있으므로 키를 바꾸는 것이 가장 안전
```

### 실수 8: 전부 망한 것 같다 — 최후의 수단

```bash
# 현재 상태를 백업
cp -r . ../프로젝트_백업_$(date +%Y%m%d)

# 원격 저장소의 깨끗한 상태로 복원
git fetch origin
git reset --hard origin/dev/woonyong
```

이래도 안 되면 팀원에게 도움을 요청하세요. 혼자 해결하려다 더 꼬이는 게 가장 위험합니다.

---

## 부록: 유용한 Git 명령어 모음

### 상태 확인

```bash
git status                          # 현재 상태
git log --oneline --graph -10       # 최근 10개 커밋을 그래프로
git log --oneline --all --graph     # 모든 브랜치의 전체 히스토리
git diff                            # 스테이징 전 변경사항
git diff --staged                   # 스테이징된 변경사항
git branch -a                       # 모든 브랜치 목록
```

### 브랜치 관리

```bash
git checkout -b dev/woonyong        # 브랜치 생성 + 이동
git checkout dev                    # 브랜치 이동
git branch -d dev/woonyong          # 브랜치 삭제 (머지된 것만)
git branch -D dev/woonyong          # 브랜치 강제 삭제
```

### 되돌리기

```bash
git reset --soft HEAD~1             # 커밋 취소 (변경사항 유지)
git reset --hard HEAD~1             # 커밋 + 변경사항 전부 취소
git revert HEAD                     # 되돌리는 새 커밋 생성
git checkout -- <파일>              # 특정 파일을 마지막 커밋 상태로
git merge --abort                   # 머지 취소
```

### 임시 저장

```bash
git stash push -m "설명"            # 저장
git stash list                      # 목록
git stash pop                       # 복원 (목록에서 제거)
git stash apply                     # 복원 (목록에 유지)
git stash drop                      # 삭제
```

---

## 부록: Pull Request 작성법

### PR 제목

커밋 컨벤션과 동일한 형식:
```
feat: 알람 클록 sleep/wakeup 메커니즘 구현
```

### PR 본문 템플릿

```markdown
## 무엇을 변경했나요?

timer_sleep()에서 busy waiting 대신 thread_sleep()을 호출하도록 변경했습니다.

## 왜 변경했나요?

기존 busy waiting 방식은 CPU를 낭비합니다.
sleep_list를 사용해 효율적으로 스레드를 재우고 깨웁니다.

## 주요 변경 파일

- `devices/timer.c` — timer_sleep 수정
- `threads/thread.c` — thread_sleep, thread_awake 추가
- `threads/thread.h` — wakeup_tick 필드, 함수 선언 추가

## 테스트

- [x] alarm-single 통과
- [x] alarm-multiple 통과
- [x] alarm-zero 통과
- [x] alarm-negative 통과
```

### 리뷰어를 위한 팁

- PR은 **300줄 이내**가 이상적 (많으면 쪼개기)
- "이 파일부터 읽으세요"라고 순서를 안내하면 리뷰어가 편합니다
- 스크린샷이나 테스트 결과를 첨부하면 좋습니다

---

## 변경 이력

| 날짜 | 변경 |
|------|------|
| 2026-04-25 | 초판. 초보 개발자 팀을 위한 Git 워크플로우 가이드. |

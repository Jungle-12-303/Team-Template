---
name: wiki-archiving
description: WIKI 저장소의 학습 기록을 키워드 기반으로 아카이빙하는 스킬. 키워드 레벨 10 체계로 폴더/문서를 구조화하고, 중복 제거 및 병합을 수행한다. "아카이빙", "정리", "키워드", "학습 기록", "중복 제거", "병합", "지식 정리", "위키 정리"가 언급되면 활성화.
---

# WIKI 학습 기록 아카이빙 스킬

> 이 스킬은 팀의 학습 기록을 **키워드 기반 계층 구조**로 정리하고,
> 중복 콘텐츠를 제거·병합하여 지식을 축적하는 절차를 안내합니다.

---

## 1. 키워드 레벨 체계

학습 기록은 **레벨 1(최상위) ~ 레벨 10(최하위)** 까지의 키워드 계층으로 분류합니다.

### 레벨 정의

| 레벨 | 역할 | 예시 |
|------|------|------|
| L1 | 대분류 — 기술 도메인 | `OS`, `Network`, `Algorithm`, `AI`, `Web` |
| L2 | 중분류 — 주요 주제 | `OS/Process`, `OS/Memory`, `OS/Scheduling` |
| L3 | 소분류 — 세부 주제 | `OS/Scheduling/Priority` |
| L4 | 구체 개념 | `OS/Scheduling/Priority/Priority-Donation` |
| L5 | 구현/이론 구분 | `OS/Scheduling/Priority/Priority-Donation/Implementation` |
| L6 | 세부 구현 | `.../Implementation/Nested-Donation` |
| L7 | 특정 파일/모듈 | `.../Nested-Donation/thread.c` |
| L8 | 함수/구조체 단위 | `.../thread.c/thread_donate_priority` |
| L9 | 이슈/버그/TIL | `.../thread_donate_priority/deadlock-case` |
| L10 | 최말단 메모 | `.../deadlock-case/2026-04-25-해결방법` |

### 실제 폴더 구조 예시

```
docs/knowledge/
├── OS/
│   ├── README.md                          # L1 — OS 키워드 인덱스
│   ├── Process/
│   │   ├── README.md                      # L2
│   │   ├── Context-Switch/
│   │   │   └── README.md                  # L3
│   │   └── System-Call/
│   │       ├── README.md                  # L3
│   │       └── exec-vs-fork.md            # L4
│   ├── Memory/
│   │   ├── README.md
│   │   ├── Virtual-Memory/
│   │   │   ├── README.md
│   │   │   ├── Page-Table/
│   │   │   │   └── README.md
│   │   │   └── Page-Fault/
│   │   │       └── README.md
│   │   └── Stack-Growth/
│   │       └── README.md
│   └── Scheduling/
│       ├── README.md
│       ├── Priority/
│       │   ├── README.md
│       │   └── Priority-Donation/
│       │       ├── README.md
│       │       ├── Implementation/
│       │       │   └── nested-donation.md
│       │       └── TIL/
│       │           └── 2026-04-25.md
│       └── MLFQS/
│           └── README.md
├── Network/
│   └── ...
├── Algorithm/
│   └── ...
└── AI/
    ├── LLM/
    │   ├── Transformer/
    │   │   └── ...
    │   └── Tokenizer/
    │       └── ...
    └── ...
```

### 규칙

- **모든 폴더에 README.md** — 해당 키워드의 개요, 하위 키워드 목록, 관련 링크를 담는다.
- **폴더명은 영문 PascalCase 또는 kebab-case** — 한글은 문서 내용에만 사용.
- **레벨 4 이하는 필요할 때만 생성** — 처음부터 10단계를 다 만들지 않는다.
- **TIL은 날짜 기반 파일명** — `YYYY-MM-DD.md` 또는 `YYYY-MM-DD-제목.md`

---

## 2. 키워드 분류 기준

### 새 학습 기록이 들어왔을 때

```
1. 제목과 내용에서 핵심 키워드를 추출한다
2. 기존 키워드 트리에서 가장 가까운 위치를 찾는다
3. 정확히 맞는 위치가 있으면 → 해당 폴더에 배치
4. 없으면 → 적절한 레벨에 새 폴더/키워드 생성
5. 여러 키워드에 걸치면 → 주 키워드 위치에 배치 + 다른 키워드에서 링크
```

### 키워드 추출 방법

학습 기록에서 다음을 추출:

- **주 키워드** (1~2개) — 이 문서의 핵심 주제. 폴더 위치 결정에 사용.
- **부 키워드** (0~3개) — 관련 주제. README의 "관련 문서" 링크에 사용.
- **태그** (자유) — 문서 상단 frontmatter에 기록.

```markdown
---
keywords: [Priority-Donation, Nested-Donation]
related: [Scheduling, Lock, Semaphore]
tags: [PintOS, Project1, Implementation]
date: 2026-04-25
author: woonyong
---
```

---

## 3. 중복 제거 절차

### Step 1: 중복 탐지

같은 주제를 다루는 문서가 여러 개 있는지 확인:

```
검사 기준:
- 같은 키워드 폴더에 비슷한 제목의 문서가 2개 이상
- 다른 폴더에 있지만 내용의 70% 이상이 동일
- 같은 코드 스니펫이 3곳 이상에서 반복
```

### Step 2: 중복 유형 판별

| 유형 | 설명 | 조치 |
|------|------|------|
| 완전 중복 | 내용이 거의 동일 | 하나만 남기고 나머지 삭제 |
| 부분 중복 | 공통 부분 + 각자 고유 부분 | 병합 → 하나의 문서로 통합 |
| 관점 중복 | 같은 주제를 다른 관점에서 | 유지하되 서로 링크 연결 |
| 시간 중복 | 같은 주제의 과거/현재 버전 | 최신만 유지, 과거는 변경 이력에 기록 |

### Step 3: 병합 절차

```markdown
## 병합 전

문서 A: Priority Donation 개념 설명 (woonyong, 04/20)
문서 B: Priority Donation 구현 방법 (jihye, 04/22)
문서 C: Priority Donation 디버깅 (woonyong, 04/24)

## 병합 후

Priority-Donation/README.md:
  - 개념 설명 (문서 A에서)
  - 구현 방법 (문서 B에서)
  - 디버깅 팁 (문서 C에서)
  - 원본 기여자: woonyong, jihye
```

병합 시 지켜야 할 것:
- **원 작성자를 반드시 기록** — frontmatter의 `contributors` 필드
- **병합 커밋 메시지** — `docs: Priority-Donation 관련 문서 3개를 병합`
- **원본은 병합 커밋에서 삭제** — Git 히스토리로 복구 가능

### Step 4: 링크 업데이트

병합 후 다른 문서에서 삭제된 문서를 참조하고 있다면 링크를 업데이트:

```bash
grep -r "priority-donation-concept.md" docs/
```

---

## 4. README.md 템플릿 (각 키워드 폴더)

```markdown
# [키워드 이름]

> 한 줄 설명

## 개요

이 키워드에 대한 2~3문장 요약.

## 하위 키워드

- [하위 키워드 1](./Sub-Keyword-1/README.md) — 한 줄 설명
- [하위 키워드 2](./Sub-Keyword-2/README.md) — 한 줄 설명

## 문서 목록

- [문서 제목](./document.md) — 작성자, 날짜, 한 줄 요약

## 관련 키워드

- [관련 키워드 1](../Related/README.md)
- [관련 키워드 2](../../Other/Topic/README.md)
```

---

## 5. 학습 기록 문서 템플릿

```markdown
---
keywords: [주 키워드]
related: [부 키워드1, 부 키워드2]
tags: [PintOS, Project1]
date: YYYY-MM-DD
author: 이름
contributors: [이름1, 이름2]
---

# 제목

## 배경

왜 이것을 학습했는지, 어떤 문제를 풀다가 만났는지.

## 핵심 내용

학습한 핵심 개념, 원리, 구현 방법.

## 코드 / 예시

관련 코드가 있다면.

## 알게 된 것 (TIL)

- 이전에 몰랐던 것
- 실수했던 것과 해결 방법

## 참고 자료

- [링크 제목](URL)
```

---

## 6. 아카이빙 워크플로우

### 일상: 학습 기록 추가

```
1. 오늘 배운 것을 학습 기록 템플릿에 작성
2. 키워드 분류 기준에 따라 적절한 폴더에 배치
3. 해당 폴더의 README.md에 문서 링크 추가
4. 커밋: docs: [키워드] 학습 기록 추가
```

### 주간: 정리 & 중복 제거

```
1. 이번 주에 추가된 문서 목록 확인
   git log --since="1 week ago" --name-only -- docs/knowledge/

2. 중복 탐지
   - 같은 폴더에 비슷한 문서가 있는지
   - 다른 폴더에 같은 내용이 있는지

3. 중복 발견 시 → 병합 절차 수행

4. 키워드 트리 검토
   - 하위 문서가 5개 이상이면 → 하위 폴더(키워드) 분리 검토
   - 문서가 1개뿐인 하위 폴더 → 상위 폴더로 흡수 검토

5. 커밋: docs: 주간 아카이빙 정리
```

### 프로젝트 종료 시: 대규모 정리

```
1. 전체 키워드 트리를 검토
2. 팀원별 TIL을 키워드별로 재분류
3. 중복 콘텐츠 일괄 병합
4. README 인덱스 전체 업데이트
5. 다음 팀을 위한 "이 프로젝트에서 배운 것" 요약 문서 작성
```

---

## 7. AI 활용 가이드

이 스킬을 Claude와 함께 사용할 때:

### 학습 기록 분류 요청

```
이 학습 기록을 키워드 체계에 맞게 분류해줘:
[학습 내용 붙여넣기]
```

### 중복 탐지 요청

```
docs/knowledge/ 아래에서 중복된 내용을 찾아줘
```

### 주간 정리 요청

```
이번 주 학습 기록을 정리해줘
```

---

## 8. 주의사항

- **삭제보다 병합** — 내용을 버리지 말고 합치기. 누군가의 노력이 담긴 기록입니다.
- **작성자 존중** — 병합 시 원 작성자를 반드시 `contributors`에 기록.
- **완벽하지 않아도 됨** — L1~L3만으로 시작해도 충분.
- **키워드는 진화한다** — 프로젝트가 진행되면서 체계가 바뀔 수 있음. 자연스러운 과정.
- **Git이 백업** — 병합/삭제해도 Git 히스토리에 원본이 남아있으므로 걱정 없이 정리.

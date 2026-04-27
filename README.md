# Team-Template

크래프톤 정글 12기 303호 반 전체가 공유하는 개발 환경, 유틸리티, 컨벤션, 스킬 저장소입니다.

## 이 저장소의 역할

이 저장소는 개인 프로젝트가 아닙니다. 반에 속한 모든 팀이 공통으로 사용하는 기반 저장소입니다.

- 동일한 개발 환경에서 출발할 수 있도록 설정과 도구를 정리합니다.
- 팀 간 코드 스타일, 커밋 메시지, Git 워크플로우 등 컨벤션을 통일합니다.
- 디버깅, 테스트, 코드 리뷰 등 실무에 필요한 스킬 가이드를 제공합니다.

같은 환경에서 같은 규칙으로 시작해야, 팀 간 협업과 코드 공유가 가능합니다.

## 빠른 시작 (온보딩)

### 1단계: 레포 클론

```bash
git clone https://github.com/Jungle-12-303/Team-Template.git
cd Team-Template
```

### 2단계: 구조 파악

이 저장소의 모든 문서는 `skills/` 폴더에 있습니다. 각 문서는 독립적으로 읽을 수 있으며, 필요한 것만 골라서 보면 됩니다.

```
skills/
  pintos-test-explorer.md   Pintos Test Explorer VS Code 확장 설치 및 사용법
  debug-pintos-vscode.md    VS Code에서 PintOS GDB 디버깅하는 방법
  debug-pintos-clion.md     CLion에서 PintOS 디버깅하는 방법
  c-style-pintos.md         PintOS C 코딩 스타일 규칙
  commit-convention.md      커밋 메시지 작성 규칙
  git-workflow.md           Git 브랜치 전략과 워크플로우
  code-review.md            코드 리뷰 가이드
  meeting-minutes.md        회의록 작성 가이드
  claude-skill-guide.md     Claude AI 스킬 활용 가이드
  wiki-archiving.md         WIKI 레포에 학습 내용 정리하는 방법
  wiki-writing.md           WIKI 문서 작성 문법, 중복 방지, 병합 규칙
```

### 3단계: 개발 환경 설정

아래 문서를 순서대로 확인하세요. 자신이 사용하는 IDE에 맞는 문서를 선택합니다.

| 순서 | 문서 | 설명 |
|------|------|------|
| 1 | [pintos-test-explorer.md](skills/pintos-test-explorer.md) | Pintos 테스트를 VS Code에서 실행하고 디버깅하는 확장 설치 |
| 2-A | [debug-pintos-vscode.md](skills/debug-pintos-vscode.md) | VS Code 사용자: GDB 연동 디버깅 설정 |
| 2-B | [debug-pintos-clion.md](skills/debug-pintos-clion.md) | CLion 사용자: PintOS 디버깅 설정 |
| 3 | [c-style-pintos.md](skills/c-style-pintos.md) | PintOS 코드 작성 시 지켜야 할 C 스타일 |

### 4단계: 컨벤션 확인

코드를 작성하기 전에 팀 전체가 따르는 규칙을 확인하세요.

| 문서 | 핵심 내용 | 언제 읽나 |
|------|-----------|-----------|
| [commit-convention.md](skills/commit-convention.md) | `<type>: <한국어 제목>` 형식, scope 없음 | 첫 커밋 전에 반드시 |
| [git-workflow.md](skills/git-workflow.md) | 브랜치 전략, PR 규칙, merge 방식 | 팀 작업 시작 전에 |
| [code-review.md](skills/code-review.md) | 리뷰 요청/응답 방법, 체크리스트 | PR을 올리거나 리뷰할 때 |
| [wiki-writing.md](skills/wiki-writing.md) | WIKI 문서 문법, 중복 방지, 병합 규칙 | WIKI에 학습 내용을 쓸 때 |

### 5단계: 기여하기

이 저장소에 새로운 스킬 문서를 추가하고 싶다면:

1. `skills/` 폴더에 새 md 파일을 생성합니다.
2. 파일명은 영문 소문자 + 하이픈으로 작성합니다 (예: `gdb-cheatsheet.md`).
3. 이 README의 구조 섹션과 스킬 문서 목록 표에 새 항목을 추가합니다.
4. 커밋 컨벤션을 따라 커밋합니다.

```bash
git add skills/gdb-cheatsheet.md README.md
git commit -m "docs: GDB 자주 쓰는 명령어 치트시트를 추가"
git push origin main
```

## 스킬 문서 목록

| 문서 | 설명 | 대상 |
|------|------|------|
| [pintos-test-explorer.md](skills/pintos-test-explorer.md) | Pintos Test Explorer VS Code 확장 설치, 테스트 실행, 디버깅 연동 | VS Code 사용자 |
| [debug-pintos-vscode.md](skills/debug-pintos-vscode.md) | VS Code에서 GDB를 연동하여 PintOS 커널을 디버깅하는 방법 | VS Code 사용자 |
| [debug-pintos-clion.md](skills/debug-pintos-clion.md) | CLion에서 PintOS 프로젝트를 설정하고 디버깅하는 방법 | CLion 사용자 |
| [c-style-pintos.md](skills/c-style-pintos.md) | PintOS 코드베이스의 C 코딩 스타일 규칙 | 전원 |
| [commit-convention.md](skills/commit-convention.md) | Conventional Commits 기반 커밋 메시지 규칙 (한국어 제목) | 전원 |
| [git-workflow.md](skills/git-workflow.md) | Git 브랜치 전략, PR 생성, merge 규칙 | 전원 |
| [code-review.md](skills/code-review.md) | 코드 리뷰 요청 방법, 리뷰어 체크리스트 | 전원 |
| [meeting-minutes.md](skills/meeting-minutes.md) | 회의록 작성 형식과 보관 규칙 | 팀장 |
| [claude-skill-guide.md](skills/claude-skill-guide.md) | Claude AI를 학습과 개발에 활용하는 방법 | 선택 |
| [wiki-archiving.md](skills/wiki-archiving.md) | WIKI 레포에 학습 내용을 정리하는 절차 | 전원 |
| [wiki-writing.md](skills/wiki-writing.md) | WIKI 문서 작성 문법, 중복 방지, 외부 소스 병합 규칙 | 전원 |

## 커밋 컨벤션 요약

이 레포에서도 Conventional Commits 형식을 사용합니다.

```
<type>: <한국어 제목>
```

주로 사용하는 타입:

| 타입 | 용도 | 예시 |
|------|------|------|
| `docs` | 스킬 문서 추가/수정 | `docs: GDB 치트시트를 추가` |
| `feat` | 새 스크립트, 도구 추가 | `feat: Makefile 린트 스크립트를 추가` |
| `fix` | 오류, 오타 수정 | `fix: debug-pintos-vscode 경로 오류를 수정` |
| `chore` | 설정, 구조 변경 | `chore: .gitignore에 빌드 산출물을 추가` |

제목은 결과를 구체적으로 설명합니다. `수정`, `업데이트` 같은 막연한 단어는 피합니다.

## 관련 저장소

| 저장소 | 역할 | 언제 사용하나 |
|--------|------|---------------|
| [WIKI](https://github.com/Jungle-12-303/WIKI) | 공동 학습 기록, 키워드별 지식 정리 | 학습한 내용을 문서로 남길 때 |
| 이 저장소 (Team-Template) | 개발 환경, 도구, 컨벤션 | 환경 설정, 규칙 확인, 스킬 참고 시 |

두 저장소는 역할이 다릅니다. Team-Template은 "어떻게 작업하는가"를, WIKI는 "무엇을 배웠는가"를 다룹니다.

# Claude Skill 등록 가이드

`docs/Skill` 아래 문서들을 Claude(Cowork / Claude Code)가 직접 사용할 수 있는 스킬로 등록하고, 프롬프트에서 호출하는 방법을 정리한 가이드입니다.

## 대상 파일

이 저장소에서는 아래 파일들을 스킬 원본으로 사용합니다.

- `docs/Skill/c-style-pintos.md`
- `docs/Skill/commit-convention.md`
- `docs/Skill/SKILL.md` (회의록 작성 스킬)

이 파일들은 이미 YAML frontmatter의 `name`, `description`을 포함하고 있어 스킬 본문으로 재사용하기 좋습니다.

## Claude 스킬 기본 구조

Claude 스킬은 프로젝트 내 `docs/Skill/` 또는 사용자 글로벌 경로에 배치할 수 있습니다.

### 프로젝트 내 스킬 (권장)

```text
<project-root>/docs/Skill/
├── SKILL.md                  # 회의록 작성 스킬
├── c-style-pintos.md         # PintOS C 스타일
├── commit-convention.md      # Git 커밋 컨벤션
└── claude-skill-guide.md     # 이 가이드
```

프로젝트 안에 스킬을 두면 팀원 모두가 동일한 스킬을 공유하고, Git으로 버전 관리할 수 있습니다.

### 사용자 글로벌 스킬 (개인 환경)

개인 환경에서 모든 프로젝트에 적용할 스킬은 아래 경로에 배치합니다.

```text
~/workspace/skills/
├── commit-convention/
│   └── SKILL.md
├── c-style-pintos/
│   └── SKILL.md
└── ...
```

## 등록 방법

### 1. 프로젝트 내 스킬 (팀 공유)

`docs/Skill/` 폴더에 스킬 파일을 넣으면 됩니다. 별도 설치 과정이 없습니다.

```bash
# 이미 docs/Skill/ 안에 파일이 있으면 바로 사용 가능
ls docs/Skill/
```

### 2. 글로벌 스킬 (개인 환경)

```bash
mkdir -p ~/workspace/skills/c-style-pintos
mkdir -p ~/workspace/skills/commit-convention

cp docs/Skill/c-style-pintos.md ~/workspace/skills/c-style-pintos/SKILL.md
cp docs/Skill/commit-convention.md ~/workspace/skills/commit-convention/SKILL.md
```

### 3. 프로젝트 심링크 설정 (자동화)

프로젝트에 `.claude/skills/` 심링크를 만들면 Claude가 자동으로 인식합니다.

```bash
# setup-skills.sh 스크립트가 있다면:
bash ~/workspace/skills/setup-skills.sh <프로젝트경로>
```

## 업데이트 방법

프로젝트 내 `docs/Skill/*.md`를 원본으로 유지합니다.
문서를 수정한 뒤 글로벌 스킬로도 사용 중이라면 복사본을 갱신합니다.

```bash
cp docs/Skill/commit-convention.md ~/workspace/skills/commit-convention/SKILL.md
```

새 세션에서 반영하는 것이 가장 안전합니다.

## 사용하는 방법

스킬 이름을 프롬프트에 직접 언급하면 가장 확실합니다.

### 예시 프롬프트

#### commit-convention

```text
commit-convention 스킬로 이번 변경에 맞는 커밋 메시지를 작성해줘
```

```text
브랜치 전략은 commit-convention 기준으로 따라줘
```

#### c-style-pintos

```text
c-style-pintos 스킬 기준으로 thread.c를 리뷰해줘
```

```text
PintOS 코딩 스타일에 맞게 이 함수를 수정해줘
```

#### 회의록 (SKILL.md)

```text
오늘 회의록을 작성해줘
```

```text
회의록 스킬로 이번 주간 회의를 정리해줘
```

## 여러 스킬을 함께 쓰는 방법

필요하면 한 요청에서 여러 개를 같이 부를 수 있습니다.

```text
c-style-pintos 스킬 기준으로 코드를 수정하고,
마지막 커밋 메시지는 commit-convention 기준으로 작성해줘
```

## 권장 운영 방식

- 프로젝트 안의 `docs/Skill/*.md`를 팀의 원본 문서로 유지합니다.
- 개인 환경의 `~/workspace/skills/*/SKILL.md`는 실행용 복사본으로 봅니다.
- 컨벤션 문서를 바꿨으면 글로벌 스킬 복사본도 함께 갱신합니다.
- 스킬 호출이 중요할 때는 프롬프트에 이름을 직접 적습니다.

## Codex에서 마이그레이션

기존에 Codex 스킬(`~/.codex/skills/`)을 사용하고 있었다면:

```bash
# Codex 스킬을 Claude 스킬 경로로 복사
cp -r ~/.codex/skills/* ~/workspace/skills/
```

파일 구조(`<skill-name>/SKILL.md`)는 동일하므로 그대로 사용 가능합니다.

## 빠른 점검 체크리스트

- `docs/Skill/` 안에 스킬 파일이 존재하는가
- 파일 상단 frontmatter에 `name`, `description`이 있는가
- 스킬 이름을 프롬프트에 직접 언급했는가
- 글로벌 스킬 사용 시 `~/workspace/skills/<skill-name>/SKILL.md`가 존재하는가
- 문서 수정 후 새 세션에서 다시 확인했는가

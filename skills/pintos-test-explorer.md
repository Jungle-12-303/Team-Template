---
name: pintos-test-explorer
description: VS Code 확장 Pintos Test Explorer의 설치 및 사용법. PintOS 테스트를 사이드바에서 실행하고 GDB 디버깅을 연동하는 방법을 다룬다. "테스트", "Test Explorer", "테스트 실행", "Pintos 테스트"가 언급되면 활성화.
---

# Pintos Test Explorer — VS Code 확장

> PintOS 테스트를 VS Code 사이드바에서 실행하고 디버깅할 수 있는 확장입니다.
> 테스트 이름을 외울 필요 없이 트리 구조에서 클릭만으로 실행/디버깅이 가능합니다.

- 제작: Yoojin Koh
- 마켓플레이스: https://marketplace.visualstudio.com/items?itemName=YoojinKoh.pintos-test-explorer
- 식별자: `yoojinkoh.pintos-test-explorer`
- 버전: 0.1.6
- 범주: Debuggers, Testing

---

## 1. 설치

### 요구 사항

- VS Code `1.85.0` 이상
- Linux 또는 Dev Container 환경
- `make`가 설치되어 있어야 함
- `gdb`가 PATH에 있어야 디버그 세션 사용 가능
- `ms-vscode.cpptools` 확장 필요 (C/C++ 확장)

### 설치 방법

1. VS Code에서 Extensions 패널 열기 (`Ctrl+Shift+X`)
2. `Pintos Test Explorer` 검색
3. `Install` 클릭
4. Dev Container를 사용하는 경우, 컨테이너 내부에도 설치
5. 설치 후 `Developer: Reload Window` 실행

설치가 완료되면 Activity Bar에 `P os` 아이콘이 나타납니다.

### 워크스페이스 구조

아래 두 가지 레이아웃 중 하나를 지원합니다:

```
방법 1: 프로젝트 루트에 직접 배치
<workspace>/threads/
<workspace>/userprog/
<workspace>/vm/
<workspace>/tests/

방법 2: pintos 하위 폴더
<workspace>/pintos/threads/
<workspace>/pintos/userprog/
<workspace>/pintos/vm/
<workspace>/pintos/tests/
```

---

## 2. 기본 사용법

### 테스트 탐색

1. Activity Bar에서 `P os` 아이콘 클릭
2. `Threads`, `User Programs`, `Virtual Memory`, `File System` 중 원하는 프로젝트 펼치기
3. 각 프로젝트 아래에 내장 테스트 목록이 트리 형태로 표시됨

### 테스트 실행

- 개별 실행: 테스트 옆의 녹색 `Run` 버튼 클릭
- 일괄 실행: 여러 테스트를 체크한 후 상단 툴바의 `Run Checked Tests` 클릭

### 테스트 디버깅

- 테스트 옆의 주황색 `Debug` 버튼 클릭
- GDB 디버그 세션이 자동으로 시작됨
- 브레이크포인트를 미리 설정해두면 해당 위치에서 멈춤

### 테스트 결과 확인

테스트 실행 후 트리에 결과 아티팩트 링크가 표시됩니다:

- `output` — 테스트 실행 출력
- `result` — 테스트 통과/실패 결과
- `errors` — 에러 로그

### 기타 기능

- 정렬 전환: 툴바의 정렬 버튼으로 `Number order`와 `Latest first` 전환
- 초기화: 휴지통 버튼으로 체크된 테스트와 기존 `output`, `result`, `errors` 삭제

---

## 3. 추천 워크플로우

```
1. 코드 수정
2. Pintos Test Explorer에서 관련 테스트 선택
3. Run으로 통과 여부 확인
4. 실패 시 Debug로 GDB 디버깅 진입
5. 브레이크포인트에서 변수 확인, 스텝 실행
6. 수정 후 다시 Run
```

---

## 4. 트러블슈팅

| 문제 | 해결 방법 |
|------|----------|
| `P os` 아이콘이 안 보임 | `Developer: Reload Window` 실행 |
| 테스트 목록이 비어 있음 | 워크스페이스 구조가 위의 두 레이아웃 중 하나인지 확인 |
| Debug 버튼이 작동 안 함 | `gdb`가 PATH에 있는지, `ms-vscode.cpptools` 설치 여부 확인 |
| Dev Container에서 안 됨 | 컨테이너 내부에 확장을 별도 설치했는지 확인 |

---

## 5. 관련 문서

- [PintOS 디버깅 가이드 — VS Code 편](debug-pintos-vscode.md)
- [PintOS 디버깅 가이드 — CLion 편](debug-pintos-clion.md)
- [pintos_22.04_lab_docker](https://github.com/krafton-jungle/pintos_22.04_lab_docker) — 공식 Docker 이미지

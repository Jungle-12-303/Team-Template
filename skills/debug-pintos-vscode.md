---
name: debug-pintos-vscode
description: VS Code 환경에서 PintOS를 디버깅하는 가이드 스킬. GDB 연동, launch.json 설정, 브레이크포인트, QEMU/Bochs 연결 방법을 다룬다. "VSCode", "VS Code", "디버깅", "GDB", "PintOS 디버깅", "launch.json", "backtrace"가 언급되면 활성화.
---

# PintOS 디버깅 가이드 — VS Code 편

> VS Code에서 PintOS 커널을 GDB로 디버깅하는 방법을 단계별로 설명합니다.
> C/C++ 확장과 launch.json 설정만으로 GUI 디버깅이 가능합니다.

---

## 1. 디버깅 방법 선택 가이드

| 상황 | 추천 방법 | 난이도 |
|------|-----------|--------|
| "이 변수 값이 뭐지?" | printf 디버깅 | 쉬움 |
| "여기서 왜 멈추지?" | ASSERT + backtrace | 쉬움 |
| "실행 흐름을 추적하고 싶다" | GDB + VS Code 원격 디버깅 | 보통 |
| "커널 패닉이 났다" | backtrace 분석 | 보통 |
| "메모리가 이상하다" | GDB 메모리 검사 | 어려움 |

---

## 2. printf 디버깅

CLion 편과 동일한 방법을 사용합니다. 자세한 내용은 `debug-pintos-clion.md`의 섹션 2를 참고하세요.

핵심만 요약:

```c
/* threads/debug.h */
#define DEBUG_LEVEL 1

#if DEBUG_LEVEL >= 1
#define DBG(fmt, ...) \
    printf ("[DBG:%s:%d] " fmt "\n", __func__, __LINE__, ##__VA_ARGS__)
#else
#define DBG(fmt, ...) ((void)0)
#endif
```

제출 전 `DEBUG_LEVEL`을 0으로 변경.

---

## 3. VS Code 사전 준비

### 필수 확장 설치

1. **C/C++** (Microsoft) — `ms-vscode.cpptools`
2. **C/C++ Extension Pack** (선택) — IntelliSense 향상

VS Code 터미널에서:

```bash
code --install-extension ms-vscode.cpptools
```

### GDB 확인

```bash
# 시스템 GDB 확인
gdb --version

# 또는 크로스 컴파일러용 GDB
i386-elf-gdb --version    # Mac (Homebrew)
```

없으면 설치:

```bash
# Ubuntu/WSL
sudo apt install gdb

# Mac (Homebrew)
brew install i386-elf-gdb
```

---

## 4. VS Code + GDB 원격 디버깅 설정

### 4.1 launch.json 생성

프로젝트 루트에 `.vscode/launch.json` 파일 생성:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "PintOS Debug (QEMU)",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/kernel.o",
            "miDebuggerServerAddress": "localhost:1234",
            "miDebuggerPath": "/usr/bin/gdb",
            "cwd": "${workspaceFolder}/build",
            "stopAtEntry": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "pretty printing 활성화",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "PintOS 심볼 로드",
                    "text": "symbol-file ${workspaceFolder}/build/kernel.o",
                    "ignoreFailures": false
                }
            ]
        }
    ]
}
```

**Mac에서 i386-elf-gdb를 사용하는 경우:**

```json
"miDebuggerPath": "/usr/local/bin/i386-elf-gdb"
```

### 4.2 tasks.json으로 빌드+실행 자동화 (선택)

`.vscode/tasks.json`:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "PintOS Build",
            "type": "shell",
            "command": "make",
            "args": ["clean", "&&", "make"],
            "options": { "cwd": "${workspaceFolder}/threads/build" },
            "group": "build"
        },
        {
            "label": "PintOS QEMU GDB",
            "type": "shell",
            "command": "pintos",
            "args": ["--qemu", "--gdb", "--", "run", "alarm-multiple"],
            "options": { "cwd": "${workspaceFolder}/threads/build" },
            "isBackground": true,
            "problemMatcher": []
        }
    ]
}
```

### 4.3 c_cpp_properties.json (IntelliSense)

`.vscode/c_cpp_properties.json`:

```json
{
    "configurations": [
        {
            "name": "PintOS",
            "includePath": [
                "${workspaceFolder}/**",
                "${workspaceFolder}/include/**",
                "${workspaceFolder}/include/lib/**",
                "${workspaceFolder}/include/lib/kernel/**",
                "${workspaceFolder}/include/threads/**",
                "${workspaceFolder}/include/devices/**"
            ],
            "defines": ["USERPROG"],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c99",
            "intelliSenseMode": "linux-gcc-x86"
        }
    ],
    "version": 4
}
```

---

## 5. 디버깅 시작 — 단계별

### Step 1: PintOS 빌드

```bash
cd threads/build    # 또는 해당 프로젝트 폴더
make clean && make
```

### Step 2: QEMU를 GDB 모드로 실행

터미널 1:

```bash
pintos --qemu --gdb -- run alarm-multiple
```

QEMU가 시작되고 GDB 연결을 기다립니다 (CPU 정지 상태).

### Step 3: VS Code에서 디버깅 시작

1. 좌측 사이드바 → 벌레+재생 아이콘 (Run and Debug)
2. 상단 드롭다운에서 `PintOS Debug (QEMU)` 선택
3. 초록색 재생 버튼 클릭 (또는 F5)

연결되면 VS Code 상단에 디버그 컨트롤 바가 나타납니다.

### Step 4: 브레이크포인트 설정

- **줄 번호 왼쪽 클릭** → 빨간 점 = 브레이크포인트
- **조건부 브레이크포인트** → 빨간 점 우클릭 → Expression 입력
  - 예: `curr->tid == 3`
- **함수 브레이크포인트** → Debug Console에서 `break thread_sleep` 입력

---

## 6. VS Code 디버깅 조작

### 단축키

| 동작 | 단축키 (Mac) | 단축키 (Windows/Linux) |
|------|-------------|----------------------|
| 계속 실행 | F5 | F5 |
| Step Over | F10 | F10 |
| Step Into | F11 | F11 |
| Step Out | Shift+F11 | Shift+F11 |
| 재시작 | Cmd+Shift+F5 | Ctrl+Shift+F5 |
| 중지 | Shift+F5 | Shift+F5 |

### 변수 확인

**VARIABLES 패널** (좌측 디버그 사이드바):
- Locals — 현재 함수의 지역 변수
- Globals — 전역 변수 (Register 포함)

**WATCH 패널** — `+` 버튼으로 감시 표현식 추가:
- `thread_current()->tid`
- `thread_current()->priority`
- `thread_current()->name`
- `list_size(&ready_list)`

### Debug Console (GDB 명령어 직접 입력)

VS Code 하단의 Debug Console에서 `-exec` 접두사로 GDB 명령어 사용:

```
-exec p *thread_current()
-exec p sleep_list
-exec x/16xw 0xc0000000
-exec info threads
-exec bt                      # backtrace
```

---

## 7. 커널 패닉 분석

CLion 편과 동일합니다. 핵심만 요약:

```bash
# 패닉 메시지의 Call stack 주소를 변환
backtrace build/kernel.o 0xc0106b38 0xc01022a1 0xc0102547

# 아래에서 위로 읽으면 호출 경로
```

### 흔한 패닉 원인

| 패닉 메시지 | 원인 | 해결 |
|-------------|------|------|
| `intr_get_level() == INTR_OFF` | 인터럽트 안 끄고 block 호출 | `intr_disable()` 추가 |
| `!intr_context()` | 인터럽트 핸들러에서 block 호출 | 핸들러 코드 수정 |
| `page fault at 0x0` | NULL 포인터 | 포인터 초기화 확인 |
| `page fault at 0xcccccccc` | 해제된 메모리 | use-after-free 확인 |

---

## 8. QEMU 모니터 활용

QEMU 창에서 `Ctrl+A` → `C`로 모니터 모드:

```
info registers       # 레지스터
info mem             # 메모리 맵
info tlb             # TLB
savevm snap1         # 스냅샷 저장
loadvm snap1         # 스냅샷 복원
```

---

## 9. VS Code 특화 팁

### 멀티 터미널 활용

VS Code 내장 터미널을 분할해서:
- 터미널 1: QEMU 실행 (GDB 모드)
- 터미널 2: make 빌드
- 터미널 3: Git 작업

`Cmd+\` (Mac) 또는 `Ctrl+\`로 터미널 분할.

### 코드 내비게이션

디버깅 중에도 사용 가능:
- `Cmd+Click` — 함수 정의로 이동
- `Cmd+Shift+O` — 현재 파일의 심볼 목록
- `Cmd+T` — 전체 프로젝트 심볼 검색
- `F12` — Go to Definition
- `Shift+F12` — 참조 찾기

### settings.json 추천 설정

```json
{
    "files.associations": {
        "*.h": "c"
    },
    "editor.tabSize": 8,
    "editor.insertSpaces": false,
    "editor.rulers": [80, 100],
    "C_Cpp.default.cStandard": "c99"
}
```

PintOS의 TAB 인덴트와 80컬럼 규칙에 맞춘 설정입니다.

---

## 10. 디버깅 체크리스트

```
□ 1. 에러 메시지를 정확히 읽었는가
□ 2. backtrace를 변환해서 호출 경로를 확인했는가
□ 3. 해당 함수의 ASSERT 전제조건을 확인했는가
□ 4. printf로 주요 변수 값을 출력해봤는가
□ 5. 인터럽트 상태가 올바른가
□ 6. NULL 포인터 접근이 없는가
□ 7. 리스트가 초기화되었는가
□ 8. 동기화 사용이 올바른가
□ 9. make clean && make 했는가
□ 10. 팀원에게 설명해봤는가 (러버덕 디버깅)
```

---

## 변경 이력

| 날짜 | 변경 |
|------|------|
| 2026-04-25 | 초판. VS Code + GDB 원격 디버깅, launch.json, 디버그 콘솔 포함. |

---
name: debug-pintos-clion
description: CLion 환경에서 PintOS를 디버깅하는 가이드 스킬. GDB 연동, 브레이크포인트, 변수 감시, QEMU/Bochs 연결, 커널 패닉 분석 방법을 다룬다. "CLion", "디버깅", "GDB", "PintOS 디버깅", "브레이크포인트", "backtrace", "커널 패닉"이 언급되면 활성화.
---

# PintOS 디버깅 가이드 — CLion 편

> CLion에서 PintOS 커널을 GDB로 디버깅하는 방법을 단계별로 설명합니다.
> printf 디버깅부터 GDB 원격 디버깅까지, 초보자도 따라할 수 있도록 구성했습니다.

---

## 1. 디버깅 방법 선택 가이드

| 상황 | 추천 방법 | 난이도 |
|------|-----------|--------|
| "이 변수 값이 뭐지?" | printf 디버깅 | 쉬움 |
| "여기서 왜 멈추지?" | ASSERT + backtrace | 쉬움 |
| "실행 흐름을 추적하고 싶다" | GDB + CLion 원격 디버깅 | 보통 |
| "커널 패닉이 났다" | backtrace 분석 | 보통 |
| "메모리가 이상하다" | GDB 메모리 검사 | 어려움 |
| "타이밍/동기화 문제" | printf 타임스탬프 + 로그 | 어려움 |

---

## 2. printf 디버깅 — 가장 빠른 방법

### 기본 사용

```c
#include <stdio.h>

void
thread_sleep (int64_t wakeup_tick) {
    struct thread *curr = thread_current ();

    printf ("[DEBUG] thread_sleep: tid=%d, name=%s, wakeup_tick=%lld\n",
            curr->tid, curr->name, wakeup_tick);

    /* ... 코드 ... */

    printf ("[DEBUG] thread_sleep: about to block, status=%d\n", curr->status);
    thread_block ();
    printf ("[DEBUG] thread_sleep: woke up! tid=%d\n", curr->tid);
}
```

### 조건부 디버그 매크로

매번 printf를 지우고 다시 쓰는 것은 비효율적. 매크로를 사용:

```c
/* threads/debug.h (새로 만들기) */
#ifndef THREADS_DEBUG_H
#define THREADS_DEBUG_H

#include <stdio.h>

/* 1로 설정하면 디버그 출력 활성화, 0이면 비활성화 */
#define DEBUG_LEVEL 1

#if DEBUG_LEVEL >= 1
#define DBG(fmt, ...) \
    printf ("[DBG:%s:%d] " fmt "\n", __func__, __LINE__, ##__VA_ARGS__)
#else
#define DBG(fmt, ...) ((void)0)
#endif

#if DEBUG_LEVEL >= 2
#define DBG2(fmt, ...) \
    printf ("[DBG2:%s:%d] " fmt "\n", __func__, __LINE__, ##__VA_ARGS__)
#else
#define DBG2(fmt, ...) ((void)0)
#endif

#endif /* threads/debug.h */
```

사용법:

```c
#include "threads/debug.h"

void
thread_sleep (int64_t wakeup_tick) {
    DBG ("tid=%d, wakeup_tick=%lld", thread_current ()->tid, wakeup_tick);
    /* ... */
}
```

제출 전에 `DEBUG_LEVEL`을 0으로 바꾸면 모든 디버그 출력이 사라집니다.

### 주의: printf가 출력 안 될 때

- PintOS에서는 인터럽트가 꺼진 상태에서 printf가 동작하지 않을 수 있다.
- 인터럽트 핸들러 안에서는 printf 사용을 피하고, 전역 변수에 값을 저장 후 나중에 출력.
- **채점 스크립트의 출력을 오염시키지 않도록** 디버그 출력은 제출 전 반드시 제거.

---

## 3. ASSERT와 backtrace

### ASSERT 활용

```c
void
thread_block (void) {
    ASSERT (!intr_context ());                    /* 인터럽트 핸들러 아님 */
    ASSERT (intr_get_level () == INTR_OFF);       /* 인터럽트 비활성화 상태 */
    ASSERT (thread_current ()->status != THREAD_DYING);  /* 죽는 중 아님 */

    thread_current ()->status = THREAD_BLOCKED;
    schedule ();
}
```

ASSERT 실패 시 자동으로 backtrace가 출력됩니다.

### backtrace 읽는 법

커널 패닉이나 ASSERT 실패 시 이런 출력이 나옵니다:

```
Kernel PANIC at ../../threads/thread.c:234 in thread_block():
assertion `intr_get_level () == INTR_OFF' failed.

Call stack: 0xc0106b38 0xc01022a1 0xc0102547 0xc01035f2
```

**backtrace 변환:**

```bash
# PintOS 루트에서
backtrace build/kernel.o 0xc0106b38 0xc01022a1 0xc0102547 0xc01035f2
```

출력:
```
0xc0106b38: debug_panic (lib/kernel/debug.c:34)
0xc01022a1: thread_block (threads/thread.c:234)
0xc0102547: sema_down (threads/synch.c:62)
0xc01035f2: timer_sleep (devices/timer.c:101)
```

아래에서 위로 읽으면 호출 순서가 됩니다: `timer_sleep → sema_down → thread_block → panic`

---

## 4. CLion + GDB 원격 디버깅 설정

### 4.1 사전 준비

PintOS가 QEMU에서 실행되어야 합니다. Bochs도 가능하지만 QEMU가 GDB 연동이 더 편합니다.

### 4.2 QEMU를 GDB 서버 모드로 실행

```bash
cd build

# GDB 대기 모드로 QEMU 실행 (포트 1234에서 GDB 연결 대기)
pintos --qemu --gdb -- run alarm-multiple
```

또는 직접:

```bash
qemu-system-i386 -hda os.dsk -m 4 -net none -serial stdio -s -S
```

`-s` = GDB 서버를 포트 1234에서 시작
`-S` = CPU를 멈춘 상태로 시작 (GDB 연결까지 대기)

### 4.3 CLion GDB Remote Debug 설정

1. **Run → Edit Configurations → + → GDB Remote Debug**

2. 설정 입력:

| 항목 | 값 |
|------|-----|
| Name | `PintOS Debug` |
| GDB | 시스템 GDB 경로 (예: `/usr/bin/gdb` 또는 `/usr/local/bin/i386-elf-gdb`) |
| Target remote args | `localhost:1234` |
| Symbol file | `build/kernel.o` (절대 경로) |
| Sysroot | (비워두기) |
| Path mappings | (비워두기 또는 프로젝트 루트) |

3. **Apply → OK**

### 4.4 디버깅 시작

1. QEMU를 GDB 모드로 먼저 실행 (터미널에서)
2. CLion에서 `PintOS Debug` 설정 선택 → 벌레 아이콘 클릭 (Debug)
3. 연결되면 QEMU가 멈춰 있는 상태 → CLion에서 Resume(F9)

### 4.5 브레이크포인트 설정

CLion에서 일반 프로젝트처럼 사용:

- **줄 번호 클릭** → 빨간 점 = 브레이크포인트
- **조건부 브레이크포인트** → 빨간 점 우클릭 → Condition에 `curr->tid == 3` 입력
- **함수 브레이크포인트** → Run → Toggle Breakpoint → Function Breakpoint → `thread_sleep`

### 4.6 디버깅 중 유용한 동작

| CLion 동작 | 단축키 (Mac) | 설명 |
|-----------|-------------|------|
| Resume | F9 | 다음 브레이크포인트까지 실행 |
| Step Over | F8 | 현재 줄 실행 (함수 안 들어가지 않음) |
| Step Into | F7 | 함수 안으로 들어감 |
| Step Out | Shift+F8 | 현재 함수에서 나감 |
| Evaluate | Alt+F8 | 표현식 실행 (변수 값 확인) |

### 4.7 변수 감시

- **Variables 패널** — 현재 스코프의 지역 변수 자동 표시
- **Watches** — `+` 버튼으로 감시할 표현식 추가
  - `thread_current()->tid`
  - `thread_current()->priority`
  - `list_size(&ready_list)`
  - `*(struct thread *)list_entry(list_front(&sleep_list), struct thread, elem)`

### 4.8 메모리 뷰

CLion 하단 GDB 콘솔에서 직접 GDB 명령어 입력 가능:

```gdb
# 메모리 내용 확인
x/16xw 0xc0000000

# struct thread 내용 출력
p *thread_current()

# sleep_list의 모든 스레드 출력
p sleep_list

# 특정 주소의 thread 구조체
p *(struct thread *)0xc010a000
```

---

## 5. 커널 패닉 분석

### 패닉 메시지 구조

```
Kernel PANIC at ../../threads/thread.c:234 in thread_block():
assertion `intr_get_level () == INTR_OFF' failed.

Interrupt state: on
Call stack: 0xc0106b38 0xc01022a1 0xc0102547 0xc01035f2
```

분석 순서:

1. **어디서?** — `thread.c:234 in thread_block()` → thread_block 함수 234번 줄
2. **왜?** — `intr_get_level() == INTR_OFF` 실패 → 인터럽트가 켜진 상태에서 thread_block 호출
3. **누가 불렀나?** — Call stack을 backtrace로 변환 → 호출 경로 확인
4. **왜 그 상태였나?** — 호출 경로를 따라가며 `intr_disable()` 누락 확인

### 흔한 패닉 원인과 해결

| 패닉 메시지 | 원인 | 해결 |
|-------------|------|------|
| `assertion 'intr_get_level() == INTR_OFF'` | 인터럽트 안 끄고 block/unblock | `intr_disable()` 추가 |
| `assertion '!intr_context()'` | 인터럽트 핸들러에서 block 호출 | 핸들러에서 block 계열 제거 |
| `page fault at 0x0` | NULL 포인터 역참조 | 포인터 초기화 확인 |
| `page fault at 0xcccccccc` | 해제된 메모리 접근 | use-after-free 확인 |
| `PANIC 'kernel bug'` | schedule() 관련 오류 | ready_list, running thread 상태 확인 |

---

## 6. QEMU 모니터 활용

QEMU 실행 중 `Ctrl+A` → `C`로 모니터 모드 진입:

```
# 레지스터 확인
info registers

# 메모리 맵 확인
info mem

# TLB 확인
info tlb

# 인터럽트 상태
info irq

# 스냅샷 저장 (특정 시점으로 되돌리기 가능)
savevm checkpoint1
loadvm checkpoint1
```

---

## 7. 디버깅 체크리스트

문제가 생겼을 때 순서대로 확인:

```
□ 1. 에러 메시지를 정확히 읽었는가
□ 2. backtrace를 변환해서 호출 경로를 확인했는가
□ 3. 해당 함수의 ASSERT 전제조건을 확인했는가
□ 4. printf로 주요 변수 값을 출력해봤는가
□ 5. 인터럽트 활성화/비활성화 상태가 올바른가
□ 6. NULL 포인터 접근이 없는가
□ 7. 리스트 조작 전에 리스트가 초기화되었는가
□ 8. 동기화(lock, semaphore) 사용이 올바른가
□ 9. 수정한 코드가 빌드되는가 (make clean && make)
□ 10. 같은 문제를 팀원에게 설명해봤는가 (러버덕 디버깅)
```

---

## 변경 이력

| 날짜 | 변경 |
|------|------|
| 2026-04-25 | 초판. CLion + GDB 원격 디버깅, printf, backtrace, QEMU 모니터 포함. |

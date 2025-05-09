---
title: Python Coroutine
date: 2025-05-09 09:35:20 +09:00
categories: ['python']
tags: ['python', 'coroutine', 'concurrency']
---

Python에서는 동시성(concurrency)과 병렬성(parallelism)이 필수입니다. 
데이터 처리량이 커지고, 네트워크나 파일 입출력(I/O)을 동반한 작업이 많아지면서 순차 실행만으로는 프로그램의 응답성과 처리 속도에 한계가 있습니다.

본 글에서는 반복 가능한 객체를 만드는 제너레이터와 코루틴, concurrent.futures (ThreadPoolExecutor / ProcessPoolExecutor)를 다룹니다.

---

## 제너레이터 & 코루틴 기초

### `yield`

| 구분                | 역할                       |
| ----------------- | ------------------------ |
| **`yield` 우측 값**  | 호출자에게 *내보내는(return)* 값   |
| **`yield` 좌측 변수** | 호출자가 `.send()`로 *보내주는* 값 |

```python
# 제너레이터
def gen():
    yield 1            # → 호출자에게 1 반환 & 일시 정지
    yield 2            # → 다시 호출되면 여기부터 실행

g = gen()
next(g)  # 1
next(g)  # 2

# 코루틴
def coro():
    x = yield 10       # ① 10 내보내고 멈춤
    print('받은 값:', x) # ② .send()로 받은 값 출력

c = coro()
next(c)      # 시작(= send(None))
c.send(99)   # x ← 99, 이후 종료
```

❗ 코루틴은 **반드시** `next(c)` *또는* `c.send(None)`으로 ‘ 호출해야 `.send()`로 값을 줄 수 있습니다.

## Python Concurrency

| 용어              | 설명                                           | 적합한 상황           |
| --------------- | -------------------------------------------- | ---------------- |
| **Concurrency** | 한 CPU에서 작업을 빠르게 번갈아 실행해 ‘동시에’ 보이게 함          | I/O 대기 많은 프로그램   |
| **Parallelism** | 여러 CPU/코어가 **진짜로 동시에** 실행                    | CPU 계산이 무거운 프로그램 |
| **GIL**         | CPython 인터프리터의 전역 락. 스레드가 동시에 바이트코드를 실행하지 못함 | 스레드 성능 제한 요인     |

## `concurrent.futures` API

### 공통 패턴

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed

with <ExecutorClass>(max_workers=...) as executor:
    futures = [executor.submit(fn, arg) for arg in work_list]
    for f in as_completed(futures):
        print(f.result())
```

### `ThreadPoolExecutor vs ProcessPoolExecutor`

| 항목         | ThreadPoolExecutor | ProcessPoolExecutor |
| ---------- | ------------------ | ------------------- |
| **GIL 영향** | 받음 → **가짜 병렬**     | 안 받음 → **진짜 병렬**    |
| 적합 작업      | 파일, 네트워크 등 I/O     | 수치 계산, 이미지 처리 등 CPU |
| 오버헤드       | 낮음 (메모리 공유)        | 높음 (프로세스 간 직렬화)     |

### 결과 수집 메서드

| 메서드                      | 특징                  | 사용 예            |
| ------------------------ | ------------------- | --------------- |
| `map(fn, iterable)`      | 입력 순서 == 출력 순서      | 순서 중요, 간단 반복    |
| `wait(futures, timeout)` | 전체 완료 or 타임아웃까지 블록  | 종료 시점 제어        |
| `as_completed(futures)`  | **끝나는 순서**대로 반복자 제공 | 실시간 출력, UI 업데이트 |

## 코드

### ThreadPool + `as_completed()`

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

WORK = [10_000, 1_000_000, 10_000_000]

def calc(n):
    return sum(range(1, n + 1))

start = time.time()
with ThreadPoolExecutor() as ex:
    futures = [ex.submit(calc, n) for n in WORK]
    for f in as_completed(futures):
        print('결과:', f.result())
print(f'Time: {time.time() - start:.2f}s')
```

### 4‑2 ProcessPool 실험 (CPU 바운드)

```python
from concurrent.futures import ProcessPoolExecutor
import time

N = 100_000_000
WORK = [N, N, N, N]  # 4개 작업

def calc(n):
    return sum(range(1, n + 1))

start = time.time()
with ProcessPoolExecutor() as ex:
    results = list(ex.map(calc, WORK))
print('결과[0]:', results[0])
print(f'소요 시간: {time.time() - start:.2f}s')
```

실제 테스트 시 ProcessPool이 ThreadPool보다 수배 빠르게 끝납니다 (코어 수에 비례).

---

## When, What?

| 작업            | What                                             |
| ----------------- | -------------------------------------------------- |
| **대용량 파일 읽기/쓰기**  | `ThreadPoolExecutor`                               |
| **웹 크롤링/HTTP 호출** | `ThreadPoolExecutor` + `asyncio` 가능                |
| **행렬 곱, 이미지 필터**  | `ProcessPoolExecutor` 또는 `NumPy`/`multiprocessing` |
| **DB 쿼리 수백 개**    | `ThreadPoolExecutor` (I/O)                         |

---

## 맺음말

> 제너레이터와 코루틴은 ‘fine-grained concurrency’을, `concurrent.futures`는 'coarse-grained concurrency’을 다룹니다. 
> 두 레이어를 적재적소에 조합하면 읽기 쉽고, 성능도 좋은 Python 코드를 작성할 수 있습니다. 

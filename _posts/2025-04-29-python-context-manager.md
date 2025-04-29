---
title: Python Context Manager
date: 2025-04-29 08:39:20 +09:00
categories: ['python']
tags: ['python', 'context-manager', 'decorator', 'generator']
---

Python 문법인 데코레이터(Decorator), 제너레이터(Generator), 그리고 with 문에서 사용하는 컨텍스트 매니저(Context Manager) 개념을 정리합니다.

Python에서는 리소스를 열고 사용하는 과정을 안전하게 처리하기 위해 with 문을 사용합니다.
이 흐름을 효과적으로 구성하기 위해 yield, __enter__, __exit__ 같은 문법이 활용됩니다.

본 문서에서는 데코레이터를 통한 함수 감싸기, 제너레이터를 통한 실행 흐름 제어,
그리고 with 문을 구성하는 다양한 방법을 정리합니다.

## 1. 데코레이터 (Decorator)
> 함수를 *포장*해 실행 전후(또는 실행 방식)를 바꿉니다.

```python
def simple_decorator(func):
    def wrapper(*args, **kwargs):
        print("전처리")
        result = func(*args, **kwargs)
        print("후처리")
        return result
    return wrapper

@simple_decorator
def say_hello():
    print("Hello!")

say_hello()
# 전처리 → Hello! → 후처리
```

| 특징 | 설명 |
|------|------|
|목적|함수/클래스를 간편히 래핑해서 부가 기능 삽입|
|사용하는 곳|로깅, 권한 체크, 캐싱, `@contextmanager` 등|

---

## 2. 제너레이터 (Generator) & `yield`
> “값을 하나씩 *스트리밍*하고, 함수 실행을 잠시 중단·재개할 수 있는 객체입니다.”

```python
def numbers():
    yield 1        # ① 값 전송 후 멈춤
    yield 2        # ② 이어서 실행
    yield 3

g = numbers()
next(g)  # 1
next(g)  # 2
next(g)  # 3
```

### `yield` vs `return`

|구분|`return`|`yield`|
|---|---|---|
|역할|값 반환 후 **종료**|값 반환 후 **중단**|
|상태 유지|X|O (로컬 변수, 실행 위치 보존)|
|재실행|불가|`next()` 호출 때 이어서 실행|

---

## 3. `@contextmanager` (데코레이터 + 제너레이터)
> “`yield` 앞을 **`__enter__`**, `yield` 뒤를 **`__exit__`** 처럼 만들어 with 문을 손쉽게 구현합니다.”

```python
from contextlib import contextmanager
import time

@contextmanager
def execute_timer(msg):
    start = time.monotonic()          # __enter__
    try:
        yield start                   # with 블록에 값 전달
    except Exception as e:
        print("Logging exception:", e)
        raise
    else:
        print(f"{msg}: {time.monotonic() - start:.3f}s")  # __exit__
```

```python
with execute_timer("작업 시간"):
    time.sleep(1)
# → "작업 시간: 1.001s"
```

### Why yield?
`with` 흐름은 “**enter → (블록) → exit**” 3단계를 필요로 합니다.  
`yield`는 *중단점*을 제공해 이 3단계를 하나의 함수 안에서 나눌 수 있게 해줍니다.

---

## 4. 클래스 기반 Context Manager

```python
class ExecuteTimer:
    def __init__(self, msg):
        self.msg = msg

    def __enter__(self):
        import time
        self.start = time.monotonic()
        return self.start

    def __exit__(self, exc_type, exc_value, tb):
        import time, traceback
        if exc_type:
            traceback.print_exception(exc_type, exc_value, tb)
        else:
            print(f"{self.msg}: {time.monotonic() - self.start:.3f}s")
        return True  # 예외 무시
```

동일한 **with** 사용:

```python
with ExecuteTimer("작업 시간"):
    time.sleep(1)
```

---

## 5. 세 가지 구현 비교

|구현|with 지원|코드 길이|복잡한 상태 관리|
|----|---------|---------|----------------|
|① 데코레이터만 (`yield` 없음)|✖|가장 짧음|전처리/후처리 정도|
|② `@contextmanager` + `yield`|✔|짧고 직관적|단순 자원 관리에 이상적|
|③ 클래스 `__enter__/__exit__`|✔|다소 길다|다양한 메소드·상태 필요할 때|

---

## 6. 실전 사용 가이드

|필요 조건|추천 구현|
|---------|---------|
|단순히 파일·타이머 등 간단 리소스|`@contextmanager`|
|다수 메서드와 복합 상태, 상속이 필요한 경우|클래스 `__enter__/__exit__`|
|with 문 필요 없고 호출 전후 훅만|일반 데코레이터|

---

## 맺음말
`yield`는 **“값을 내보내고 함수 실행을 잠시 중단”** 하는 Python의 제어 흐름 도구입니다.  
`@contextmanager`로 **with 문 리소스 관리**를 간단하게 해결할 수 있고, 데이터 스트리밍·비동기 처리까지 확장됩니다.

> **return은 종료, yield는 일시정지.**  
> 필요에 따라 데코레이터·제너레이터·컨텍스트 매니저를 적재적소에 배치하면 가독성과 안정성을 모두 챙길 수 있습니다.

> 본 글이 도움이 되었기를 바랍니다.

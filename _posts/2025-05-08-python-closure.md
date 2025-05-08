---
title: Python Closure
date: 2025-05-08 09:35:20 +09:00
categories: ['python']
tags: ['python', 'closure', 'higher-order']
---

파이썬에서는 함수가 일급 객체입니다. 함수를 변수에 담고, 다른 함수에 전달하며, 실행 결과로 반환할 수 있습니다. 이러한 특징은  higher‑order function, closure 개념으로 이어집니다.

---

## 함수는 일급 객체다

| 특징           | 설명                         | 예제 코드 스니펫               |
| ------------ | -------------------------- | ----------------------- |
| **런타임 초기화**  | 실행 시점에 정의·바인딩              | `def factorial(n): ...` |
| **변수 할당 가능** | `var_func = factorial`     |                         |
| **함수 인수 전달** | `map(factorial, range(5))` |                         |
| **함수 반환**    | 클로저 함수가 내부 함수를 반환     |                         |

```python
def factorial(n):
    """재귀 팩토리얼 함수"""
    if n == 1:
        return 1
    return n * factorial(n - 1)

var_func = factorial
print(var_func(5))            # 120
```

> `dir(factorial)` 을 살펴보면 함수 객체만의 메타데이터(`__code__`, `__name__`…)를 확인할 수 있습니다.

## Closure

> 정의 : 내부 함수가 외부 함수의 지역 변수를 캡처(capture)하고, 외부 함수가 종료된 뒤에도 그 값을 보존하여 사용하는 함수 객체.

### Why?

| 시나리오           | 클로저가 제공하는 가치                                          |
| -------------- | ----------------------------------------------------- |
| **상태 누적**      | 함수 자체가 은닉된 상태를 품고 호출될 때마다 누적 가능                    |
| **팩토리 함수**     | 파라미터를 고정한 새 함수를 생성(partial)                        |
| **콜백·이벤트 핸들러** | GUI·async 환경에서 컨텍스트 정보를 함수를 통해 전달                     |
| **데이터 은닉**     | 외부에서 속성을 직접 건드릴 수 없도록 캡슐화 (읽기는 가능, 쓰기는 `nonlocal` 필요) |

### 2‑2. 내부 메커니즘

* 함수가 정의될 때, **자유 변수(free variable)** 목록을 분석하여 `cell` 객체로 래핑합니다.
* 내부 함수 객체의 `__closure__` 속성은 이 `cell`들의 튜플을 보관합니다.
* 각 `cell`은 `cell_contents` 속성으로 실제 값을 가리킵니다.
* 공유 변수를 수정하려면 `nonlocal` 키워드로 해당 `cell`에 바인딩해야 합니다.

```python
def outer():
    x = 42           # 자유 변수
    def inner():
        return x     # 읽기 전용 액세스
    return inner

f = outer()
print(f())                       # 42
print(f.__closure__[0].cell_contents)  # 42
```

### 읽기 예제

```python
def make_adder(n):
    def adder(x):
        return x + n  # n을 읽기만 함
    return adder

add5 = make_adder(5)
print(add5(10))  # 15
```

* 내부에서 `n`을 **읽기만** 하므로 `nonlocal` 없이도 정상 동작.

### 쓰기(재바인딩)가 필요한 경우 → `nonlocal`

```python
def running_avg():
    count = 0
    total = 0
    def averager(v):
        nonlocal count, total  # 외부 지역 변수 재바인딩
        count += 1
        total += v
        return total / count
    return averager

avg = running_avg()
print(avg(10))  # 10.0
print(avg(30))  # 20.0
```

> `nonlocal`을 생략하면 `UnboundLocalError`: 파이썬은 `count`를 **inner의 새로운 지역 변수**로 생각해 에러를 냅니다.

#### `nonlocal` vs `global`

| 키워드        | 바인딩 대상                     | 사용 권장도                     |
| ---------- | -------------------------- | -------------------------- |
| `nonlocal` | **바로 위** 스코프(중첩 함수)의 지역 변수 | 👍 상태가 함수 내부에만 국한될 때 안전    |
| `global`   | 모듈 전역 변수                   | ⚠️ 전역 오염 가능, 멀티스레드 환경에서 위험 |

### 클로저와 메모리

* **가비지 컬렉션**: 내부 함수가 살아 있는 한 캡처된 변수(cell)는 GC 대상이 아님.
* **참조 카운트**: 함수 객체가 해제되면 closure cell도 함께 해제.
* **성능**: 셀 접근은 로컬 변수보다 살짝 느리지만 일반 연산 대비 무시할 수 있는 수준.

### 클로저 vs 클래스

| 비교 항목  | 클로저                | 클래스(`__call__`)   |
| ------ | ------------------ | ----------------- |
| 선언 길이  | 짧다                 | 상대적으로 길다          |
| 상태 노출  | 캡슐화(은닉)            | 속성으로 명시적 노출       |
| 디버깅    | 셀을 들춰봐야 함          | `self.attr` 로 직관적 |
| 상속·다형성 | 불가                 | 가능                |
| 주 용도   | 간단한 상태 보존, 팩토리, 콜백 | 복잡 상태·확장성·OOP     |

### 사용 예시

```python
# 1) 커스텀 정렬 키 함수 생성
def sort_by(field):
    def key_func(obj):
        return getattr(obj, field)
    return key_func

people.sort(key=sort_by('age'))

# 2) 파라미터 고정 (부분 적용)
from operator import mul

def make_multiplier(n):
    def multiplier(x):
        return mul(n, x)
    return multiplier

times3 = make_multiplier(3)
print(times3(7))  # 21
```

### 클로저의 한계와 주의점

1. **디버깅 난이도** : 상태가 숨어 있어 추적 힘듦 → 로그·테스트 강화 필요.
2. **남용 위험** : 지나치게 중첩되면 가독성 하락 → 복잡해지면 클래스로 리팩터링.
3. **순환 참조** : 내부 함수가 자신을 캡처하는 구조는 피해야 GC 지연을 예방.

| 필요/상황               | 추천 방안                 | 비고        |
| ------------------- | --------------------- | --------- |
| 간단한 상태 누적, 빠른 프로토타입 | **클로저**               | 코드 짧고 직관적 |
| 복잡한 상태, 멀티메서드 필요    | **클래스**               | 명시적·확장성   |
| 글로벌 공유 상태           | 되도록 피함 (`global` 최소화) | 동시성 문제    |

---

## 맺음말

* ✨ **간단**하면 *클로저*,
* 🏗️ **복잡**하면 *클래스*

> 클로저와 클래스를 사용하면 코드를 더 간결하게 작성할 수 있습니다. 도움이 되었기를 바랍니다.

---
title: Python Shadowing
date: 2025-05-06 11:16:20 +09:00
categories: ['docker']
tags: ['python', 'class', 'method', 'shadowing']
---

> 객체지향이 필요한 이유는 **데이터(상태)**와 **행동(기능)**을 하나로 묶어, 코드의 재사용성과 유지보수성을 높이기 위해서입니다. 
> 이 과정에서 자연스럽게 다음 개념들이 등장합니다:
 
> 클래스 변수 / 인스턴스 변수 — 모든 객체가 공유하는 값과, 각 객체가 개별로 가지는 값을 구분합니다. 
> 인스턴스 메서드 / 클래스 메서드 / 정적 메서드 — 객체의 상태를 다루는 메서드, 클래스 자체의 속성을 다루는 메서드, 그리고 클래스와 무관하게 동작하는 함수를 나눕니다.

어떤 이름이 어디에 저장되는가(네임스페이스), 그리고 어떤 순서로 탐색되는지가 중요합니다.

---

## 1. 네임스페이스와 속성 탐색 순서

| 탐색 단계 | 설명 |
|-----------|----------------------------------------------------|
| ① `obj.__dict__` | 객체(인스턴스) 자체에 저장된 속성 |
| ② `obj.__class__.__dict__` | 클래스에 정의된 속성 |
| ③ 부모 클래스들 | 상속 계층을 MRO(Method Resolution Order)대로 검색 |

> **Shadowing** = 하위 네임스페이스(①)가 같은 이름을 가지면 상위 속성을 “가림”.

---

## 2. 클래스 변수 vs 인스턴스 변수

```python
class MyClass:
    class_var = 10          # 클래스 변수

    def __init__(self):
        self.inst_var = 20  # 인스턴스 변수
````

| 코드                   | 저장 위치                             | 결과           |
| -------------------- | --------------------------------- | ------------ |
| `MyClass.class_var`  | `MyClass.__dict__['class_var']`   | 10           |
| `obj = MyClass()`    |                                   |              |
| `obj.inst_var`       | `obj.__dict__['inst_var']`        | 20           |
| `obj.class_var`      | **① 없음→② 클래스에서 탐색**               | 10           |
| `obj.class_var = 99` | `obj.__dict__['class_var']=99` 생성 | Shadowing 발생 |

```python
print(obj.__dict__)      # {'inst_var': 20, 'class_var': 99}
print(MyClass.__dict__['class_var'])  # 10 (여전히 존재)
del obj.class_var
print(obj.class_var)     # 10 (Shadowing 해제 → 클래스 값 다시 보임)
```

---

## 3. 메서드 바인딩

| 종류           | 선언 형태                            | 첫 인자   | 바인딩 대상 | 대표 쓰임       |
| ------------ | -------------------------------- | ------ | ------ | ----------- |
| **인스턴스 메서드** | `def foo(self)`                  | `self` | 인스턴스   | 객체 상태 이용·변경 |
| **클래스 메서드**  | `@classmethod`<br>`def foo(cls)` | `cls`  | 클래스    | 공용 설정, 팩토리  |
| **정적 메서드**   | `@staticmethod`<br>`def foo()`   | (없음)   | 없음     | 유틸리티 함수     |

### 메서드도 Shadowing

```python
class Greeting:
    @staticmethod
    def hi():
        print("Hi (static)")

g = Greeting()
g.hi()          # Hi (static)

g.hi = lambda: print("Hi (instance)")
g.hi()          # Hi (instance)  ← 인스턴스 속성이 클래스 속성을 가림
del g.hi
g.hi()          # Hi (static)    ← 복원
```

---

## 4. `dir()` vs `__dict__` 빠른 점검

```python
class Dog:
    species = "개"
    def __init__(self, name): self.name = name

d = Dog("초코")
print(dir(d)[:5], "...")  # 수십 개 이름(내장·상속 포함) 리스트
print(d.__dict__)         # {'name': '초코'}
print(Dog.__dict__['species'])  # '개'
```

* `dir(obj)` ↔ **접근 가능한 모든 이름**(문자열 리스트)
* `obj.__dict__` ↔ **객체가 직접 소유한 속성**(딕셔너리)

---

## 맺음말

정리하면

1. **저장 위치**(인스턴스 vs 클래스)와 **탐색 순서**에 따라 shadowing이 일어납니다.
2. 메서드 세 종류는 \*“어떤 대상(self/cls/없음)에 자동으로 연결되어 동작하는지”\* <br> 그리고 어떤 데이터 값을 다루는지에 따라 구분됩니다.
3. `dir()`·`__dict__`로 네임스페이스를 바로 확인할 수 있습니다.

> 필요할 때마다 객체의 `__dict__`를 들여다보며 \*“내가 지금 어디에 무엇을 만들고 있나?”\*를 점검해 보세요. Shadowing을 활용하는데 도움이 되었기를 바랍니다.

---
title: Python MetaClass
date: 2025-05-02 08:37:20 +09:00
categories: ['python']
tags: ['python', 'meta-class', 'type']
---

메타클래스(Metaclass) 개념을 정리했습니다. 
`type`, `__new__`, `__init__`, `__call__`의 실행 순서와 역할을 정리했습니다.

## 1. 메타클래스란?

| 구분 | 정의 |
|------|------|
| **클래스(Class)** | 객체(인스턴스)를 찍어내는 설계도 |
| **메타클래스(Metaclass)** | "클래스를 찍어내는 설계도" (클래스 생성 제어) |

* **기본 메타클래스**: `type` `→` 모든 클래스는 `type`의 인스턴스.
* 커스텀 메타클래스: `class MyMeta(type): ...` 형태로 `type`을 상속해 정의.

```mermaid
graph LR
  type("type (메타클래스)") -->|클래스 생성| MyClass("MyClass (클래스)")
  MyClass -->|인스턴스 생성| obj("obj (인스턴스)")
```

---

## 2. `type`의 자기참조 구조

```python
>>> type(int)        # int 는 type 의 인스턴스
<class 'type'>
>>> type(type)       # type 도 자기 자신을 메타클래스로 가짐
<class 'type'>
```

* `type(type) == type` → 파이썬 객체 시스템을 하나의 축으로 통일하는 **self‑bootstrap** 구조.

---

## 3. 클래스 생성 단계 (정의 시점)

| 순서 | 호출 주체 | 메서드 | 설명 |
|------|-----------|--------|------|
| ① | 메타클래스 | `__new__(metacls, name, bases, attrs)` | 클래스 객체 *생성* (네임스페이스 가공) |
| ② | 메타클래스 | `__init__(cls, name, bases, attrs)` | 클래스 객체 *초기화* |

> `class MyClass(metaclass=MyMeta): ...` 를 만나면 위 두 훅이 즉시 실행.

---

## 4. 인스턴스 생성 단계 (호출 시점)

| 순서 | 호출 주체 | 메서드 | 설명 |
|------|-----------|--------|------|
| ③ | 메타클래스 | `__call__(cls, *args, **kw)` | 인스턴스 생성 총괄 |
| ④ | 클래스 | `__new__(cls, *args, **kw)` | 메모리 할당 |
| ⑤ | 클래스 | `__init__(self, *args, **kw)` | 인스턴스 초기화 |

---

## 5. 예제: `CustomList` 두 가지 방식

### 5‑1. `type()` 직접 호출

```python
CustomList1 = type('CustomList1', (list,), {
    'desc': '커스텀 리스트1',
    'cus_mul': lambda self,d: [self.__setitem__(i, self[i]*d) for i in range(len(self))]
})
```

### 5‑2. 커스텀 메타클래스 사용

```python
class CustomListMeta(type):
    def __new__(mcls, name, bases, ns):
        ns['desc'] = '커스텀 리스트2'
        ns['cus_mul'] = lambda self,d: ...
        return super().__new__(mcls, name, bases, ns)

    def __call__(cls, *a, **kw):
        print('__call__ intercept')
        return super().__call__(*a, **kw)

CustomList2 = CustomListMeta('CustomList2', (list,), {})
```

* `CustomList1` : **기본 메타클래스(`type`)**만 사용.
* `CustomList2` : **커스텀 메타클래스**로 클래스 정의 전/후, 인스턴스 생성 시점 모두 개입.

---

## 6. 상속 vs 메타클래스 한눈에 보기

| 관점 | 상속 | 메타클래스 |
|------|------|------------|
| 관심사 | 기능 재사용 | 클래스 정의·생성 제어 |
| 작동 시점 | 인스턴스가 메서드 탐색할 때 | `class` 정의·호출 시 |
| 키워드 | `class Child(Parent):` | `class X(metaclass=Meta):` |

---

## 맺음말

> 메타클래스는 "필요할 때만 쓰는 메타 프로그래밍 도구"입니다.  
> 대부분의 코드는 상속과 데코레이터만으로 충분하지만, **클래스 정의를 자동화하거나 검증이 필요한 프레임워크**를 만들 때 사용됩니다.

> 도움이 되었기를 바랍니다. ✨

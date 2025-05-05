---
title: Python Descriptor
date: 2025-05-03 08:37:20 +09:00
categories: ['python']
tags: ['python', 'data-descriptor', 'non-data-descriptor']
---


Python에서 Descriptor는 ‘속성(attribute) 접근을 가로채서 원하는 로직을 실행’하도록 해 주는 저수준(low-level) 메커니즘입니다.  
다음과 같은 상황이라면 `property` 만으로는 반복‧재사용이 불편하거나 기능이 부족합니다.

1. **여러 클래스에서 동일한 속성 로직을 재사용**하고 싶을 때
2. **ORM, 캐싱, 로깅 **(DB 조회, 타입 검증, 접근 추적 등)을 삽입할 때
3. **읽기 전용, 유효성 검사, 권한 제어** 등 세밀한 제어가 필요할 때

`property`도 내부적으로는 Descriptor이지만, **한 클래스 안의 ‘그 속성 하나’**를 다루는 데 최적화되어 있습니다.  
반면 **Descriptor 클래스는 하나만 정의해 두면 여러 클래스/인스턴스에서 재사용**할 수 있어 중복을 없애고 유지보수를 쉽게 합니다.

---

## 1. Desciptor 프로토콜

| 메서드 | 호출 시점 | 주 용도 |
|--------|-----------|---------|
| `__get__(self, instance, owner)`   | 속성 읽기 | 값 반환, 읽기 로깅·캐싱 |
| `__set__(self, instance, value)`   | 속성 쓰기 | 타입 검증, 변환, DB 쓰기 |
| `__delete__(self, instance)`       | 속성 삭제 | 정리, 캐시 무효화 |

- **data descriptor** : `__set__` 또는 `__delete__`를 구현 → 읽기·쓰기 모두 제어
- **non-data descriptor** : `__get__`만 구현 → 읽기 전용

---

## 2. `property()`와의 차이

| 비교 항목 | `property()` (편의 함수) | 디스크립터 클래스 (저수준) |
|-----------|-------------------------|---------------------------|
| 정의 위치 | *같은* 클래스 내부 | 별도 클래스로 분리 |
| 재사용성  | 속성마다 반복 정의 | 하나로 여러 클래스에 재사용 |
| 활용 범위 | 단순 Getter/Setter | ORM, 로깅, 캐싱, 권한 제어 등 복합 로직 |
| 구현 난이도 | 간단 | 상대적으로 복잡 (직접 프로토콜 구현) |

---

## 3. 예시

### 3-1. ORM 필드 매핑
```python
class IntegerField:
    def __set_name__(self, owner, name):  # 3.6+
        self.name = name

    def __get__(self, obj, owner):
        return obj.__dict__[self.name]

    def __set__(self, obj, value):
        if not isinstance(value, int):
            raise ValueError("정수만 허용")
        obj.__dict__[self.name] = value

class User:
    age = IntegerField()

u = User()
u.age = 30     # OK
u.age = "bad"  # ValueError
```

### 3-2. 로깅용 Descriptor
```python
import logging
logging.basicConfig(level=logging.INFO)

class Logged:
    def __init__(self, default=0):
        self.value = default
    def __get__(self, obj, owner):
        logging.info("read -> %r", self.value)
        return self.value
    def __set__(self, obj, value):
        logging.info("write -> %r", value)
        self.value = value

class Student:
    score = Logged(50)
s = Student()
s.score            # read 로그
s.score += 20      # read + write 로그
```

### 3-3. 캐싱되는 계산 속성
```python
class cached_property:
    def __init__(self, func):
        self.func = func
        self.cache_name = "_cache_" + func.__name__
    def __get__(self, obj, owner):
        if not hasattr(obj, self.cache_name):
            setattr(obj, self.cache_name, self.func(obj))
        return getattr(obj, self.cache_name)

class Heavy:
    @cached_property
    def answer(self):
        print("🛠 계산 중…")
        return sum(range(1_000_000))
```

---

## 4. Descriptor 활용

| 패턴 | 목적 | 비고 |
|------|------|------|
| **유효성 검사/타입 강제** | `age = IntegerField()` | ORM, 설정 객체 |
| **접근 로깅**           | `score = Logged()` | 보안·디버깅 |
| **읽기 전용 속성**       | `version = ReadOnly("1.0")` | 설정, 상수 노출 |
| **지연 계산 + 캐싱**     | `@cached_property` | 무거운 연산 결과 저장 |
| **동적 값 매핑**         | `files = DirectoryFileCount()` | 파일 시스템, 외부 API |

---

## 맺음말

> 디스크립터는 **“속성 = 값”이라는 단순 문법 뒤에 복잡한 로직을 숨길 수 있는** 파이썬의 강력한 도구입니다. 
> `property()`는 작은 스크립트 단계에서 간편하게 사용할 수 있는 도구이며, `Descriptor`는 프레임워크나 라이브러리 수준에서 반복적으로 재사용할 수 있는 기능입니다.
> 실제 프로젝트(ORM, 설정 관리, 캐싱, 로깅 등)에서 같은 패턴이 반복된다면 `Descriptor`를 사용해보는 것을 추천드립니다. 코드 중복이 사라지고, 속성 접근 하나만으로도 원하는 부가 기능을 확장할 수 있습니다.

> 도움이 되었기를 바랍니다.

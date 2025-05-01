---
title: Python OOP
date: 2025-05-01 08:37:20 +09:00
categories: ['python']
tags: ['python', '@property', 'overriding', 'overloading']
---


이 문서는 최근 학습한 내용을 바탕으로 Python 객체지향(OOP) 개념을 정리한 요약본입니다.
`@property`, `*args`, `**kwargs`, 메서드 오버로딩, 다중 디스패치, 상속 구조 등 자주 쓰이는 핵심 내용을 간단한 설명과 예제 중심으로 담았습니다.

## 1. `@property` vs Getter/Setter
- **캡슐화 목적**은 동일하지만, Python에서는 메서드를 속성처럼 노출해 사용성이 높아짐.
- 구현 패턴
  ```python
  class Person:
      def __init__(self, name):
          self._name = name  # 내부 저장소

      @property
      def name(self):
          return self._name

      @name.setter
      def name(self, value):
          if not value:
              raise ValueError("이름은 비어 있을 수 없습니다!")
          self._name = value
  ```
- 관례상 **`_변수`(single underscore)** 를 써서 보호하고, `__변수`(double underscore)는 특수한 경우에만 사용.

### 언더스코어 네이밍과 캡슐화

| 표기 | 의도 | 접근 가능 여부 |
|------|------|---------------|
| `name` | public (외부 공개) | ✅ 완전 가능 |
| `_name` | protected 관례 ("내부용, 건들지 마세요") | ⚠️ 가능하지만 자제 |
| `__name` | private (name‑mangling) | 🔒 우회 접근 가능 (`obj._Class__name`) |

```python
class Sample:
    def __init__(self):
        self.public = 1        # 공개 속성
        self._protected = 2    # 관례상 보호
        self.__private = 3     # name‑mangling

    @property
    def private(self):
        return self.__private  # 안전히 노출
```

> ⓘ 언더스코어 표기는 **강제(access control)가 아닌 개발자 간 약속**입니다. `@property`와 함께 쓰면 내부 데이터를 보호하면서도 외부 인터페이스를 단순하게 유지할 수 있습니다.

---

## 2. `dir()` vs `__dict__`
| 함수/속성 | 보여주는 범위 | 자료형 |
|-----------|--------------|--------|
| `dir(obj)` | 인스턴스가 **사용할 수 있는 모든** 속성·메서드 (상속 포함) | `list` |
| `obj.__dict__` | **직접 보유한** 속성만 (상속 제외) | `dict` |

> 모든 클래스는 암묵적으로 `object`를 상속하며, 속성 탐색은 **MRO**(Method Resolution Order)에 따라 `Child → Parent → object` 순으로 진행됩니다.

---

## 3. `super().__init__()` 호출 규칙
| 자식 클래스 상황 | `super().__init__()` 필요 여부 |
|------------------|-------------------------------|
| 별도 `__init__` 정의 없음 | ❌ (부모 생성자 자동 호출) |
| `__init__` 재정의함 | ✅ (직접 호출해야 부모 초기화 수행) |

다중 상속 환경에서도 `super()`를 호출해야 MRO에 맞게 각 부모 생성자가 한 번씩 실행됩니다.

---

## 4. 메서드 오버로딩 & 다중 디스패치
### 4‑1. Python 기본 동작
- **동일 이름 메서드를 재정의하면, 마지막 정의가 덮어쓴다.**
- 전통적 오버로딩(시그니처별 컴파일 타임 분기)은 지원하지 않음.

### 4‑2. 우회적 방법
1. **가변 인자** `*args`, `**kwargs` 에 대한 수동 분기
   ```python
   def add(self, *args):
       if len(args) == 2:
           ...
       elif len(args) == 3:
           ...
   ```
2. **디스패치 데코레이터 사용**
   ```python
   from multipledispatch import dispatch

   class Calc:
       @dispatch(int, int)
       def add(self, a, b):
           return a + b

       @dispatch(float, float)
       def add(self, a, b):
           return a + b + 0.5
   ```

---

## 5. `*args` vs `**kwargs`
| 구분 | 인자 형태 | 내부 전달 형태 | 호출 예시 |
|------|-----------|---------------|-----------|
| `*args` | 위치 기반 (값만) | `tuple` | `func(1, 2, 3)` |
| `**kwargs` | 키워드 기반 (`key=value`) | `dict` | `func(a=1, b=2)` |

> **주의:** `**kwargs`에 `1`처럼 값만 넘기면 `TypeError` 발생. 반드시 `key=value` 형태여야 함.

---

## 맺음말
> Python OOP에서 자주 헷갈릴 수 있는 개념들을 정리하였습니다.
> 이해에 도움이 되었기를 바랍니다.

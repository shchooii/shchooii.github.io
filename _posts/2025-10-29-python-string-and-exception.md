---
title: Python String & Exception
date: 2025-10-29 13:59:58 +09:00
categories: ['python']
tags: ['python', 'string', 'exception']
---


Python Essentials 2 과정의 두 번째 모듈인 문자열, 리스트 메서드 및 예외(Strings, string and list methods, and exceptions)의 내용을 체계적으로 정리한 학습 자료입니다.
프로그래밍에서 숫자가 아닌 데이터를 처리하는 능력은 필수적입니다. 이 모듈에서는 컴퓨터가 텍스트를 저장하고 처리하는 원리부터 시작하여, Python의 강력한 문자열 조작 메서드들을 학습합니다. 또한, 프로그램 실행 중 발생할 수 있는 오류를 처리하는 예외 처리(Exception Handling) 기법에 대해서도 깊이 있게 다룹니다.

## 문자와 컴퓨터 (Characters and Computers)

컴퓨터는 기본적으로 숫자만을 처리합니다. 따라서 문자나 기호를 저장하기 위해서는 각 문자에 고유한 숫자를 대응시키는 약속이 필요합니다.

### ASCII와 유니코드 (ASCII & Unicode)

* ASCII (American Standard Code for Information Interchange): 가장 널리 사용되는 표준으로, 라틴 알파벳과 일부 특수 문자를 128개의 숫자(0\~127)로 매핑합니다. (예: 'A'는 65, 'a'는 97, 공백은 32)
* I18N (Internationalization): 전 세계의 다양한 언어를 지원하기 위한 국제화 표준입니다.
* Code Point (코드 포인트): 문자에 부여된 고유한 숫자 값입니다.
* Unicode (유니코드): 전 세계의 거의 모든 문자를 포함하는 표준입니다. ASCII의 상위 호환이며 백만 개 이상의 코드 포인트를 가집니다.

### 인코딩 방식 (Encoding)

유니코드를 컴퓨터 메모리에 저장하는 방식에는 여러 가지가 있습니다.

* UCS-4: 모든 문자에 32비트(4바이트)를 사용하여 저장합니다. 간단하지만 메모리 낭비가 심할 수 있습니다.
* UTF-8: 가장 많이 사용되는 방식입니다. 가변 길이 인코딩을 사용하여 ASCII 문자는 8비트, 그 외 문자는 필요에 따라 16비트, 24비트 등을 사용하므로 메모리를 효율적으로 절약합니다.
* Python 3: 완벽하게 I18N을 지원하며, 내부적으로 유니코드를 사용합니다.

-----

## Python 문자열의 본질 (The Nature of Strings)

Python의 문자열은 불변(Immutable)의 시퀀스(Sequence)입니다. 이는 튜플과 유사하며, 리스트와는 다릅니다.

### 문자열의 기본 특징

* 길이 확인 (`len()`): 문자열의 길이를 반환합니다. 이스케이프 문자(`\n` 등)는 1글자로 취급됩니다.
* 다중 라인 문자열: 따옴표 세 개(`'''` 또는 `"""`)를 사용하여 여러 줄의 문자열을 생성할 수 있습니다.
* 불변성 (Immutability): `del` 명령어로 특정 문자를 지우거나, `append()`, `insert()` 등의 메서드로 내용을 수정할 수 없습니다.

### 문자열 연산

* 연산: `+` (연결), `*` (복제)
* ord() 함수: 문자를 해당 코드 포인트(정수)로 변환합니다.
* chr() 함수: 코드 포인트(정수)를 문자로 변환합니다.

```python
# ord()와 chr() 예제
char_1 = 'a'
print(ord(char_1))  # 97 출력

code_point = 945
print(chr(code_point)) # α (그리스 문자 알파) 출력
```

### 시퀀스(Sequence)로서의 문자열

문자열은 시퀀스이므로 인덱싱, 슬라이싱, 반복(iteration)이 가능합니다.

* 인덱싱 (Indexing): `string[0]`
* 슬라이싱 (Slicing): `string[1:3]`
* 반복 (Iterating): `for char in string:`
* 포함 여부 확인: `in`, `not in` 연산자 사용

```python
# 인덱싱과 슬라이싱
s = "Python"
print(s[0])    # 'P'
print(s[1:4])  # 'yth'

# min(), max() 함수 사용 (ASCII 코드 값 기준)
print(min("aAbByYzZ")) # 'A' (대문자가 소문자보다 코드값이 작음)
print(max("aAbByYzZ")) # 'z'
```

### 주요 메서드 및 함수

* index() 메서드: 특정 문자의 첫 번째 인덱스를 반환합니다. (없으면 ValueError 발생)
* list() 함수: 문자열을 문자 단위로 쪼개어 리스트로 변환합니다.
* count() 메서드: 특정 문자의 등장 횟수를 반환합니다.

-----

## 문자열 메서드 (String Methods)

Python은 문자열 처리를 위한 강력한 내장 메서드들을 제공합니다. 이 메서드들은 원본을 변경하지 않고 새로운 문자열을 반환합니다.

### 대소문자 변환 및 조작

* `capitalize()`: 첫 글자만 대문자로 변환.
* `lower()`: 모든 문자를 소문자로 변환.
* `upper()`: 모든 문자를 대문자로 변환.
* `swapcase()`: 대소문자를 서로 바꿈.
* `title()`: 각 단어의 첫 글자를 대문자로 변환.
* `center(width, fillchar)`: 지정된 너비 가운데 정렬.

### 검색 및 수정

* `find(substring)`: 부분 문자열의 인덱스 반환 (없으면 -1).
* `rfind(substring)`: 오른쪽 끝에서부터 검색.
* `replace(old, new)`: 문자열 치환.
* `strip()`: 양쪽 공백 제거 (`lstrip`: 왼쪽, `rstrip`: 오른쪽).

### 분리 및 결합

* `split(delimiter)`: 구분자를 기준으로 문자열을 리스트로 분리.
* `join(sequence)`: 리스트 등의 요소를 특정 구분자로 연결하여 문자열 생성.

```python
# split과 join 예제
sentence = "This is Python"
words = sentence.split() # ['This', 'is', 'Python']
print(words)

new_sentence = "-".join(words) # "This-is-Python"
print(new_sentence)
```

### 내용 검사 (Boolean 반환)

* `isalnum()`: 알파벳 또는 숫자로만 구성되었는지 확인.
* `isalpha()`: 알파벳으로만 구성되었는지 확인.
* `isdigit()`: 숫자로만 구성되었는지 확인.
* `islower()`, `isupper()`: 소문자/대문자 여부 확인.
* `startswith()`, `endswith()`: 특정 문자열로 시작/끝나는지 확인.

### 문자열 비교 및 변환

* 문자열끼리비교 연산자(`==`, `!=`, `>`, `<`) 사용 가능 (사전 순서 비교).
* 문자열과 숫자는 절대 같을 수 없음 (`"10" == 10`은 `False`).
* 형 변환: `str(number)`, `int(string)`, `float(string)`.

-----

## 예외 처리 (Exceptions)

프로그램 실행 중 발생하는 오류를 '예외(Exception)'라고 합니다. 예외를 처리하지 않으면 프로그램은 강제 종료됩니다.

### try-except 구문

예외가 발생할 가능성이 있는 코드를 `try` 블록에 넣고, 예외 발생 시 처리할 코드를 `except` 블록에 작성합니다.

```python
try:
    value = int(input("나눌 숫자를 입력하세요: "))
    print("결과:", 1 / value)
except ZeroDivisionError:
    print("0으로 나눌 수 없습니다.")
except ValueError:
    print("유효한 정수가 아닙니다.")
except:
    print("알 수 없는 에러가 발생했습니다.")
```

### 예외 계층 구조 (Exception Hierarchy)

Python의 예외는 클래스 형태로 정의되어 있으며 계층 구조(트리 구조)를 가집니다.

* BaseException: 모든 예외의 최상위 부모 클래스입니다.
* Exception: 일반적인 예외들의 부모 클래스입니다.
* 구체적인 예외(`ZeroDivisionError` 등)는 추상적인 예외(`ArithmeticError`)의 하위 클래스입니다.
* 중요: `except` 블록을 작성할 때, 구체적인 예외를 먼저 작성하고 일반적인 예외를 나중에 작성해야 합니다.

### 유용한 내장 예외 (Built-in Exceptions)

* ArithmeticError: 산술 연산 오류의 상위 클래스 (`ZeroDivisionError`, `OverflowError` 포함).
* AssertionError: `assert` 문이 실패할 때 발생.
* IndexError: 시퀀스의 인덱스 범위를 벗어났을 때 발생.
* KeyError: 딕셔너리에 없는 키를 요청했을 때 발생.
* KeyboardInterrupt: 사용자가 프로그램 실행을 중단(Ctrl+C)했을 때 발생 (BaseException의 직계 자손).
* ImportError: 모듈 임포트 실패 시 발생.
* MemoryError: 메모리가 부족하여 작업을 수행할 수 없을 때 발생.

### 예외 발생 및 검증

* raise: 프로그래머가 의도적으로 예외를 발생시킬 때 사용합니다.
* assert: 조건이 `True`인지 검사하고, `False`이면 `AssertionError`를 발생시킵니다. 주로 디버깅이나 입력값 검증에 사용됩니다.

```python
def calculate_inverse(x):
    assert x != 0 # x가 0이면 AssertionError 발생
    return 1/x
```

## 맺음말

> Python 프로그래밍의 핵심 요소인 문자열 처리와 예외 처리에 대해 자세히 살펴보았습니다. 
> 문자열은 데이터 처리, 웹 검색, 텍스트 분석 등 현대 프로그래밍의 거의 모든 분야에서 사용되는 중요한 데이터 타입입니다. 문자열의 불변성과 다양한 메서드(`split`, `join`, `find` 등)를 이해함으로써 데이터를 효율적으로 조작할 수 있습니다. 
> 예외 처리는 견고한 소프트웨어를 만들기 위한 필수 기술입니다. `try-except` 블록과 예외 계층 구조를 이해하고 적절히 활용하면, 예상치 못한 오류 상황에서도 프로그램이 중단되지 않고 사용자에게 적절한 피드백을 제공하며 안전하게 동작하도록 만들 수 있습니다.

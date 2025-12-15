---
title: Python Generators & Files
date: 2025-11-04 15:13:48 +09:00
categories: ['python']
tags: ['python', 'oop', 'properties', 'methods', 'exception']
---

Python Essentials 2 (PE2) 과정의 핵심 모듈인 제너레이터와 이터레이터, 클로저, 그리고 파일 입출력(I/O) 시스템에 대해 다룹니다. Python의 강력한 기능인 `yield` 문법과 함수형 프로그래밍 요소인 람다와 클로저를 이해하고, 텍스트 및 바이너리 파일을 효율적으로 처리하는 방법을 코드를 통해 상세히 설명하겠습니다.

## 제너레이터와 이터레이터 (Generators and Iterators)

### 이터레이터 프로토콜 (Iterator Protocol)

파이썬에서 `for` 루프가 작동하는 방식은 이터레이터 프로토콜에 기반합니다. 객체가 이터레이터가 되기 위해서는 두 가지 매직 메서드를 구현해야 합니다.

* `__iter__()`: 이터레이터 객체 자체를 반환합니다.
* `__next__()`: 다음 값을 반환하며, 더 이상 반환할 값이 없으면 `StopIteration` 예외를 발생시킵니다.

예시: 피보나치 수열 이터레이터

```python
class Fib:
    def __init__(self, nn):
        self.__n = nn
        self.__i = 0
        self.__p1 = self.__p2 = 1

    def __iter__(self):
        return self

    def __next__(self):
        self.__i += 1
        if self.__i > self.__n:
            raise StopIteration
        if self.__i in [1, 2]:
            return 1
        ret = self.__p1 + self.__p2
        self.__p1, self.__p2 = self.__p2, ret
        return ret

for i in Fib(10):
    print(i)
```

### 제너레이터와 `yield` 문

이터레이터 클래스를 직접 만드는 것은 상태를 저장해야 하므로 코드가 길어질 수 있습니다. `yield` 키워드를 사용하면 훨씬 간결하게 제너레이터를 만들 수 있습니다.

* `return` vs `yield`: `return`은 값을 반환하고 함수를 종료하지만, `yield`는 값을 반환하고 함수의 상태를 동결(freeze)하여 다음 호출 시 그 지점부터 다시 실행합니다.

예시: 피보나치 제너레이터

```python
def fibonacci(n):
    p = pp = 1
    for i in range(n):
        if i in [0, 1]:
            yield 1
        else:
            n = p + pp
            pp, p = p, n
            yield n

fibs = list(fibonacci(10))
print(fibs)
```

### 리스트 컴프리헨션 vs 제너레이터 표현식

리스트 컴프리헨션은 대괄호 `[]`를 사용하며 즉시 모든 데이터를 생성하여 메모리에 올립니다. 반면, 소괄호 `()`를 사용하면 제너레이터 표현식이 되어, 필요할 때마다 값을 하나씩 생성합니다.

```python
# 리스트 컴프리헨션 (즉시 생성)
the_list = [1 if x % 2 == 0 else 0 for x in range(10)]

# 제너레이터 표현식 (지연 생성)
the_generator = (1 if x % 2 == 0 else 0 for x in range(10))

for v in the_generator:
    print(v, end=" ")
```

## 람다 함수와 함수형 도구 (Lambda & Functional Tools)

### 람다(Lambda) 함수

람다 함수는 이름이 없는 익명 함수입니다. 코드를 간결하게 만들 때 유용합니다.

* 문법: `lambda 매개변수: 표현식`

```python
sqr = lambda x: x * x
print(sqr(5))  # 출력: 25
```

### `map()` 함수

`map(function, iterable)`은 리스트의 모든 요소에 지정된 함수를 적용하여 새로운 이터레이터를 반환합니다.

```python
list_1 = [x for x in range(5)]
# 각 요소를 2의 거듭제곱으로 변환
list_2 = list(map(lambda x: 2 ** x, list_1))
print(list_2)
```

### `filter()` 함수

`filter(function, iterable)`은 함수의 결과가 `True`인 요소만 걸러냅니다.

```python
from random import seed, randint

seed()
data = [randint(-10,10) for x in range(5)]
# 0보다 크고 짝수인 값만 필터링
filtered = list(filter(lambda x: x > 0 and x % 2 == 0, data))
print(filtered)
```

-----

## 클로저 (Closures)

클로저는 자신이 생성된 환경(컨텍스트)이 사라진 후에도, 그 환경의 변수 값을 기억하고 있는 함수를 말합니다.

예시: 거듭제곱기 생성 함수

```python
def make_closure(par):
    loc = par
    def power(p):
        return p ** loc  # 외부 함수 make_closure의 loc 변수를 기억함
    return power

fsqr = make_closure(2) # 제곱 함수 생성
fcub = make_closure(3) # 세제곱 함수 생성

print(fsqr(5)) # 25
print(fcub(5)) # 125
```

## 파일 처리 기초 (File Processing Basics)

### 파일 시스템과 경로

* Windows: 경로 구분자로 백슬래시(`\`)를 사용합니다. (예: `C:\dir\file`) 파이썬 문자열에서는 `\\`로 이스케이프해야 합니다.
* Unix/Linux: 경로 구분자로 슬래시(`/`)를 사용합니다. (예: `/dir/file`)
* Python의 특징: 파이썬은 Windows에서도 슬래시(`/`)를 경로 구분자로 사용하는 것을 허용하므로 이식성을 위해 `/` 사용을 권장합니다.

### 스트림(Stream)과 파일 핸들

파일을 다룰 때는 `open()` 함수를 통해 스트림 객체(파일 핸들)를 생성합니다. 작업이 끝나면 반드시 `close()` 해야 합니다.

### 파일 열기 모드 (Open Modes)

`open(file_name, mode, encoding)` 함수를 사용합니다.

* `'r'`: 읽기 모드 (파일이 없으면 에러)
* `'w'`: 쓰기 모드 (파일이 없으면 생성, 있으면 내용 삭제 후 새로 작성)
* `'a'`: 추가 모드 (파일 끝에 내용 추가)
* `'r+'`: 읽기/쓰기 모드 (파일이 있어야 함)
* `'w+'`: 읽기/쓰기 모드 (파일 내용 삭제 후 새로 작성)
* **접미사**: `'t'` (텍스트, 기본값), `'b'` (바이너리)

### 미리 정의된 스트림

별도의 `open` 없이 사용할 수 있는 스트림입니다 (`import sys` 필요).

* `sys.stdin`: 표준 입력 (키보드)
* `sys.stdout`: 표준 출력 (화면)
* `sys.stderr`: 표준 에러 출력 (화면)

### 에러 진단

`IOError` 객체의 `errno` 속성을 통해 구체적인 에러 원인을 알 수 있습니다. `os.strerror()`를 사용하면 에러 번호를 문자로 변환해줍니다.

```python
from os import strerror
import errno

try:
    s = open("non_existent_file.txt", "rt")
    s.close()
except IOError as exc:
    if exc.errno == errno.ENOENT:
        print("파일이 존재하지 않습니다.")
    else:
        print("에러 발생:", strerror(exc.errno))
```

## 텍스트 파일 처리 (Working with Text Files)

텍스트 파일은 라인 단위로 구성된 파일입니다.

### 읽기 메서드

1.  `read(n)`: n개의 문자를 읽습니다. n을 생략하면 전체를 읽습니다.
2.  `readline()`: 한 줄을 읽습니다.
3.  `readlines()`: 모든 줄을 읽어 리스트로 반환합니다.
4.  `for line in file`: 파일 객체를 직접 이터레이션하여 한 줄씩 처리합니다 (가장 권장되는 방식).

예시: 파일 내용 읽기 (이터레이터 사용)

```python
from os import strerror

try:
    # 텍스트 파일 열기
    for line in open('text.txt', 'rt'):
        print(line, end='') # line 자체에 개행이 포함되어 있으므로 end='' 사용
except IOError as e:
    print("I/O 에러:", strerror(e.errno))
```

### 쓰기 메서드

* `write(string)`: 문자열을 파일에 씁니다. 줄 바꿈 문자(`\n`)는 자동으로 추가되지 않으므로 명시적으로 넣어줘야 합니다.

```python
try:
    fo = open('newtext.txt', 'wt')
    for i in range(10):
        fo.write("line #" + str(i+1) + "\n")
    fo.close()
except IOError as e:
    print("I/O 에러:", strerror(e.errno))
```

## 바이너리 파일 처리 (Working with Binary Files)

이미지나 실행 파일 같은 바이너리 데이터는 `bytes`나 `bytearray`를 사용해 처리합니다.

### bytearray

수정 가능한 바이트의 시퀀스입니다. 0\~255 사이의 정수만 저장 가능합니다.

```python
data = bytearray(10)
for i in range(len(data)):
    data[i] = 10 + i
```

### 바이너리 쓰기

모드에 `'wb'`를 사용합니다. `write()` 메서드는 바이트 배열을 인자로 받습니다.

```python
try:
    bf = open('file.bin', 'wb')
    bf.write(data)
    bf.close()
except IOError as e:
    print("에러:", strerror(e.errno))
```

### 바이너리 읽기

* **`readinto(bytearray)`**: 파일 내용을 읽어 기존 `bytearray` 버퍼를 채웁니다. 새로 객체를 만들지 않아 메모리 효율적입니다.
* **`read(n)`**: n 바이트를 읽어 불변(immutable) 객체인 `bytes` 객체를 반환합니다.

예시: 파일 복사 도구 (버퍼 사용)
이 예제는 소스 파일을 읽어서 대상 파일로 복사하는 기능을 구현합니다. 대용량 파일도 처리할 수 있도록 64KB 버퍼를 사용합니다.

```python
from os import strerror

srcname = input("소스 파일명 입력: ")
dstname = input("대상 파일명 입력: ")

try:
    src = open(srcname, 'rb')
    dst = open(dstname, 'wb')
    
    buffer = bytearray(65536) # 64KB 버퍼
    total = 0
    
    # 첫 번째 읽기 시도
    readin = src.readinto(buffer)
    
    while readin > 0:
        written = dst.write(buffer[:readin]) # 읽은 만큼만 쓰기
        total += written
        readin = src.readinto(buffer) # 다음 청크 읽기
        
    print(f"{total} 바이트가 성공적으로 복사되었습니다.")
    
    src.close()
    dst.close()
    
except IOError as e:
    print("파일 처리 중 에러 발생:", strerror(e.errno))
```

## 맺음말

> 지금까지 Python의 고급 기능인 제너레이터와 클로저, 파일 입출력 방법에 대해 알아보았습니다. 
 
> 제너레이터는 메모리를 효율적으로 사용하며 대용량 데이터를 순차적으로 처리할 때 필수적입니다.
> 클로저와 람다는 코드를 간결하게 하고 함수형 프로그래밍 패턴을 가능하게 합니다. 
> 파일 처리에서는 텍스트와 바이너리 모드의 차이를 이해하고, `open` 시 적절한 모드를 선택하는 것이 중요합니다. 또한 `try-except` 블록과 `errno`를 활용한 견고한 에러 처리는 안정적인 프로그램의 기본입니다.

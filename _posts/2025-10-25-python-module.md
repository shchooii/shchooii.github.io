---
title: Python Module
date: 2025-10-25 12:59:58 +09:00
categories: ['python']
tags: ['python', 'module', 'math', 'random', 'platform', 'sys']
---


Python Essentials 2 과정의 첫 번째 모듈에서는 파이썬 프로그래밍의 확장성과 유지보수성을 결정짓는 모듈(Module), 패키지(Package), 그리고 패키지 관리 도구인 PIP에 대해 학습하였습니다. 
코드가 길어지고 프로젝트의 규모가 커짐에 따라 모든 코드를 하나의 파일에 작성하는 것은 비효율적입니다. 
코드를 기능별로 나누어 관리하는 모듈화 기술과 이를 체계적으로 묶는 패키지 기술은 필수적입니다. 
전 세계 개발자들이 공유하는 방대한 라이브러리를 활용하기 위한 PIP 사용법은 실무에서 매우 중요합니다. 
모듈의 다양한 가져오기 방식, 주요 표준 라이브러리의 활용 예시, 패키지 구조 생성법, 그리고 PIP 명령어의 실제 사용례를 상세하게 다루겠습니다. 

### 모듈의 기초와 사용법 및 예시

모듈은 파이썬 정의와 문장을 담고 있는 파일입니다. 모듈을 사용하면 코드를 재사용하기 쉽고 관리가 용이해집니다.

모듈 가져오기 (Import) 방식과 예시
모듈을 가져오는 방법에는 여러 가지가 있으며 상황에 맞게 선택해야 합니다.

첫째, 모듈 전체를 가져오는 방식입니다. 모듈명을 계속 명시해야 하므로 이름 충돌을 방지할 수 있습니다.

```python
import math

# 모듈명.함수명 형식으로 사용
print(math.pi)
print(math.ceil(3.6))  # 올림 함수 사용
```

둘째, 특정 함수나 변수만 가져오는 방식입니다. 모듈명을 생략하고 바로 사용할 수 있어 코드가 간결해집니다.

```python
from math import sqrt, pi

# 모듈명 없이 바로 사용 가능
print(sqrt(16))
print(pi)
```

셋째, 별칭(Alias)을 사용하는 방식입니다. 모듈 이름이 길거나 이름 충돌이 예상될 때 유용합니다.

```python
import math as m

print(m.sin(m.pi / 2))
```

이름 공간 (Namespace) 확인
`dir()` 함수를 사용하면 모듈이 가지고 있는 모든 함수와 변수 이름을 확인할 수 있습니다.

```python
import math
import random

# math 모듈 내의 모든 엔티티 이름을 리스트로 반환
print(dir(math)) 
```

### 주요 표준 모듈 활용 예시

파이썬은 강력한 표준 라이브러리를 내장하고 있습니다.

math 모듈 예시
수학적 계산을 돕는 모듈입니다.

```python
import math

# 팩토리얼 계산 (5! = 5 * 4 * 3 * 2 * 1)
print(math.factorial(5))

# 최대공약수(GCD) 계산
print(math.gcd(12, 18))

# 각도 변환 (라디안 -> 도)
print(math.degrees(math.pi))
```

random 모듈 예시
무작위 데이터를 다룰 때 사용합니다.

```python
import random

# 0.0 이상 1.0 미만의 실수 생성
print(random.random())

# 리스트에서 임의의 요소 선택
fruits = ['apple', 'banana', 'cherry']
print(random.choice(fruits))

# 리스트 요소 섞기
numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)
print(numbers)
```

platform 모듈 예시
실행 환경에 대한 정보를 얻을 때 사용합니다.

```python
import platform

# 운영체제 이름 확인 (Windows, Linux, Darwin 등)
print(platform.system())

# 프로세서 정보 확인
print(platform.processor())

# 파이썬 버전 확인
print(platform.python_version())
```

### 모듈과 패키지의 구조 및 생성 예시

사용자 정의 모듈과 `__name__` 활용
직접 만든 모듈을 테스트할 때 `__name__` 변수를 활용하면, 모듈이 직접 실행되는지 아니면 다른 파일에서 import 되는지를 구분할 수 있습니다.

```python
# my_module.py 파일의 내용

def greet(name):
    return f"Hello, {name}!"

# 이 파일이 직접 실행될 때만 아래 코드가 작동함
if __name__ == "__main__":
    print("모듈 직접 실행 테스트")
    print(greet("Tester"))
else:
    print("다른 파일에서 모듈로 import 되었습니다.")
```

패키지 (Package) 구조 예시
패키지는 모듈들을 디렉터리로 묶은 것입니다. `__init__.py` 파일이 해당 폴더에 있어야 파이썬이 이를 패키지로 인식합니다.

```text
my_project/
    main.py
    my_package/
        __init__.py
        sub_module1.py
        sub_module2.py
```

위와 같은 구조에서 `main.py`에서 패키지 내 모듈을 사용할 때는 다음과 같이 작성합니다.

```python
# main.py

from my_package import sub_module1

sub_module1.some_function()
```

모듈 탐색 경로 추가 (sys.path)
파이썬이 모듈을 찾지 못할 때 경로를 직접 추가하는 방법입니다.

```python
import sys

# 사용자 정의 경로 추가
sys.path.append("C:/Users/User/MyProject/Modules")

import my_custom_module
```

### Python 패키지 설치 관리자 (PIP) 사용 예시

PIP는 터미널(명령 프롬프트)에서 사용하는 도구입니다. 외부 라이브러리를 관리할 때 사용합니다.

설치 및 버전 확인
PIP가 설치되어 있는지 확인합니다.

```bash
pip --version
```

패키지 검색 및 정보 확인
설치 전 패키지를 검색하거나 설치된 패키지의 정보를 봅니다.

```bash
# 'pygame'이라는 단어가 포함된 패키지 검색
pip search pygame  

# 설치된 패키지 목록 확인
pip list

# 특정 패키지의 상세 정보(버전, 설치 위치, 의존성 등) 확인
pip show pygame
```

패키지 설치 및 삭제
가장 자주 사용하는 명령어입니다.

```bash
# 최신 버전의 pandas 패키지 설치
pip install pandas

# 특정 버전(2.0.0)을 지정하여 설치
pip install pandas==2.0.0

# 사용자 전용으로 설치 (관리자 권한 없을 때 유용)
pip install --user pandas

# 이미 설치된 패키지를 최신 버전으로 업데이트
pip install -U pandas

# 패키지 삭제
pip uninstall pandas
```

### 맺음말

> 이번 정리를 통해 파이썬 코드를 효율적으로 관리하는 모듈과 패키지의 구조, 그리고 이를 활용하는 다양한 코드를 살펴보았습니다. 
> 단순히 코드를 작성하는 것을 넘어, `import` 구문을 자유자재로 활용하고 `if __name__ == "__main__":` 패턴을 통해 모듈의 독립성을 확보하는 방법은 중급 개발자로 성장하는 데 필수적인 기술입니다.
> `math`, `random` 같은 표준 모듈뿐만 아니라 PIP 명령어를 통해 외부 라이브러리를 설치하고 관리하는 방법까지 알아보았습니다.

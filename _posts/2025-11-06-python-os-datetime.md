---
title: Python OS & Datetime Essentials
date: 2025-11-06 14:13:48 +09:00
categories: ['python']
tags: ['python', 'os', 'datetime', 'time', 'calendar']
---

Python은 강력한 표준 라이브러리를 통해 운영체제(OS)의 기능을 제어하거나, 날짜와 시간을 정밀하게 다루는 기능을 제공합니다. 본 문서는 Python Essentials 2 코스에서 다루는 Module 4의 내용을 바탕으로, 파일 시스템을 제어하는 `os` 모듈과 시간 및 날짜 데이터를 처리하는 `datetime`, `time`, `calendar` 모듈의 핵심 기능과 사용법을 상세하게 정리하였습니다.

## os 모듈: 운영체제와 상호작용

`os` 모듈은 Python 스크립트가 실행되는 운영체제(Windows, Unix 등)와 상호작용할 수 있는 기능을 제공합니다.

### 운영체제 정보 확인

* `os.uname()`: 현재 운영체제에 대한 상세 정보를 담은 객체를 반환합니다. (주로 Unix 계열에서 작동)
  * 속성: `sysname` (OS 이름), `nodename` (네트워크 이름), `release`, `version`, `machine` (하드웨어 ID)
* `os.name`: 운영체제의 종류를 나타내는 문자열을 반환합니다.
  * `posix`: Unix/Linux 계열
  * `nt`: Windows 계열
  * `java`: Jython 등

```python
import os
print(os.name)  # 예: 'posix' 또는 'nt'
# print(os.uname()) # Unix 시스템에서만 작동
```

### 디렉토리 관리

파일 시스템의 폴더(디렉토리)를 생성, 이동, 삭제하는 함수들입니다.

* 디렉토리 생성:
  * `os.mkdir(path)`: 지정한 경로에 하나의 디렉토리를 생성합니다. 이미 존재하면 `FileExistsError`가 발생합니다.
  * `os.makedirs(path)`: 경로에 포함된 모든 상위 디렉토리까지 재귀적으로 생성합니다. (Unix의 `mkdir -p`와 유사)
* 디렉토리 탐색 및 이동:
  * `os.listdir(path)`: 해당 경로에 있는 파일 및 디렉토리 리스트를 반환합니다. (`.`과 `..`은 제외)
  * `os.chdir(path)`: 현재 작업 디렉토리(Current Working Directory)를 변경합니다.
  * `os.getcwd()`: 현재 작업 디렉토리의 절대 경로를 반환합니다.
* 디렉토리 삭제:
  * `os.rmdir(path)`: 비어있는 하나의 디렉토리를 삭제합니다.
  * `os.removedirs(path)`: 경로상의 빈 디렉토리들을 재귀적으로 삭제합니다.

```python
import os

# 재귀적으로 디렉토리 생성
os.makedirs("my_first_directory/my_second_directory")
os.chdir("my_first_directory")
print(os.getcwd())  # 현재 위치 확인

# 디렉토리 내용 확인
print(os.listdir()) 

# 디렉토리 삭제 (비어있어야 함)
os.chdir("..") # 상위 폴더로 이동
os.removedirs("my_first_directory/my_second_directory")
```

### 시스템 명령어 실행

* `os.system(command)`: 문자열로 전달된 시스템 명령어를 실행합니다.
  * Windows: 쉘이 반환한 값 반환
  * Unix: 프로세스의 종료 상태(exit status) 반환 (0은 성공을 의미)

```python
import os
returned_value = os.system("mkdir my_test_dir")
print(returned_value)
```

## datetime 모듈: 날짜와 시간 조작

`datetime` 모듈은 날짜(date)와 시간(time)을 조작하는 클래스들을 제공합니다. 데이터 검증, 로깅, 데이터베이스 기록 등에 필수적입니다.

### date 클래스 (날짜)

년, 월, 일 정보를 다룹니다.

* 생성: `date(year, month, day)`
* 오늘 날짜: `date.today()`
* 속성: `year`, `month`, `day` (읽기 전용)
* Timestamp 변환: `date.fromtimestamp(timestamp)` (Unix timestamp로부터 생성)
* ISO 포맷: `date.fromisoformat('YYYY-MM-DD')`
* 값 변경: `replace(year=..., month=..., day=...)` 메서드를 사용하여 새로운 객체 반환

```python
from datetime import date
import time

today = date.today()
print(f"Today: {today}") # 2025-12-15 (예시)

# Timestamp를 날짜로 변환
timestamp = time.time()
d = date.fromtimestamp(timestamp)
print(f"Date from timestamp: {d}")
```

### time 클래스 (시간)

시, 분, 초, 마이크로초 정보를 다룹니다.

* 생성: `time(hour, minute, second, microsecond)` (모든 인자는 선택적, 기본값 0)

```python
from datetime import time
t = time(14, 53, 20)
print(f"Time: {t}") # 14:53:20
```

### datetime 클래스 (날짜 + 시간)

날짜와 시간을 함께 다룹니다.

* 생성: `datetime(year, month, day, hour, minute, second, ...)`
* 현재 시각:
  * `datetime.today()`: 현재 로컬 날짜 및 시간
  * `datetime.now()`: `today()`와 유사하나 타임존 정보 처리 가능
  * `datetime.utcnow()`: UTC 기준 현재 날짜 및 시간
* 메서드: `date()` (날짜 객체 반환), `time()` (시간 객체 반환), `timestamp()` (timestamp 실수값 반환)

### timedelta 클래스 (시간 차이 계산)

날짜나 시간의 간격을 표현하며 연산에 사용됩니다.

* 생성: `timedelta(weeks, days, hours, minutes, seconds, ...)`
* 연산: 날짜 객체끼리 뺄셈을 하면 `timedelta` 객체가 반환되며, 날짜 객체에 `timedelta`를 더하거나 뺄 수 있습니다.

```python
from datetime import date, timedelta

d1 = date(2025, 12, 25)
d2 = date(2025, 12, 15)

delta = d1 - d2
print(delta) # 10 days, 0:00:00

# 2주 후 날짜 계산
future_date = d1 + timedelta(weeks=2)
print(future_date)
```

### 포맷팅 (strftime & strptime)

* `strftime(format)`: 날짜/시간 객체를 문자열로 변환합니다. (String Format Time)
* `strptime(string, format)`: 문자열을 날짜/시간 객체로 변환합니다. (String Parse Time)
* 주요 지시자:
  * `%Y`: 연도 (4자리), `%m`: 월, `%d`: 일
  * `%H`: 시간 (24시), `%M`: 분, `%S`: 초
  * `%B`: 월 이름 (전체)

```python
from datetime import datetime

# 객체 -> 문자열
dt = datetime(2020, 11, 4, 14, 53)
print(dt.strftime("%Y/%m/%d %H:%M:%S")) # 2020/11/04 14:53:00

# 문자열 -> 객체
dt_obj = datetime.strptime("2020-11-04 14:53:00", "%Y-%m-%d %H:%M:%S")
print(dt_obj)
```

## time 모듈: Unix Timestamp 및 실행 제어

`time` 모듈은 Unix Timestamp(1970년 1월 1일 00:00:00 UTC부터 경과한 초)를 중심으로 작동합니다.

* `time.time()`: 현재 시간을 timestamp(float)로 반환합니다.
* `time.sleep(seconds)`: 프로그램 실행을 지정된 초(seconds)만큼 일시 정지합니다.
* `time.ctime(seconds)`: timestamp를 읽기 편한 문자열로 변환합니다.
* 구조체 시간 (struct\_time):
  * `time.gmtime(seconds)`: timestamp를 UTC 기준의 `struct_time` 객체로 변환합니다.
  * `time.localtime(seconds)`: timestamp를 로컬 시간 기준의 `struct_time` 객체로 변환합니다.
  * `time.mktime(tuple)`: 튜플이나 `struct_time`을 timestamp로 변환합니다.
  * `time.asctime(tuple)`: 튜플이나 `struct_time`을 문자열로 변환합니다.

<!-- end list -->

```python
import time

timestamp = time.time()
print(f"Timestamp: {timestamp}")

# 프로그램 2초 정지
time.sleep(2) 
print("2초가 지났습니다.")

# timestamp -> struct_time -> string
st = time.localtime(timestamp)
print(time.strftime("%Y-%m-%d", st))
```

## calendar 모듈: 달력 출력 및 관리

달력을 텍스트나 HTML로 출력하거나 요일 정보를 확인하는 데 사용됩니다.

* 달력 출력:
  * `calendar.calendar(year)`: 한 해의 달력을 문자열로 반환합니다.
  * `calendar.month(year, month)`: 특정 월의 달력을 문자열로 반환합니다.
  * `prcal()`, `prmonth()`: `print` 함수 없이 바로 출력합니다.
* 요일 처리:
  * 월요일(0) \~ 일요일(6)의 정수 값을 가집니다. (`calendar.MONDAY` 등의 상수 사용 가능)
  * `calendar.setfirstweekday(weekday)`: 달력의 시작 요일을 설정합니다.
  * `calendar.weekday(year, month, day)`: 특정 날짜의 요일을 정수로 반환합니다.
  * `calendar.weekheader(n)`: 요일 약어를 반환합니다 (n은 글자 수).
* 윤년 확인:
  * `calendar.isleap(year)`: 윤년 여부 확인 (True/False).
  * `calendar.leapdays(y1, y2)`: y1과 y2 사이의 윤년 횟수를 반환합니다.
* 이터레이터:
  * `calendar.Calendar` 클래스를 사용하여 `itermonthdates` 등을 통해 날짜를 순회할 수 있습니다.

<!-- end list -->

```python
import calendar

# 2020년 12월 24일 요일 확인 (0: 월 ~ 6: 일)
print(calendar.weekday(2020, 12, 24)) 

# 윤년 확인
print(f"Is 2020 leap? {calendar.isleap(2020)}")

# Calendar 객체 사용
c = calendar.Calendar()
for date in c.itermonthdates(2020, 12):
    print(date, end=" ") # 해당 월에 포함된 주(week)를 완성하기 위해 전/후 달의 날짜도 포함됨
```

##  맺음말 (Conclusion)

> Python의 필수 모듈인 `os`, `datetime`, `time`, `calendar`에 대해 알아보았습니다.

> `os` 모듈은 파일 및 디렉토리 관리와 같은 시스템 작업을 자동화하는 데 핵심적입니다.
> `datetime`과 `time` 모듈은 데이터의 시간 기록(Logging), 시간차 계산, 포맷 변환 등 데이터 처리 파이프라인에서 필수적인 역할을 합니다.
> `calendar` 모듈은 날짜 기반의 스케줄링이나 시각화에 유용하게 활용될 수 있습니다.


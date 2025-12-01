---
title: Operating System Summary
date: 2025-07-16 12:40:20 +09:00
categories: ['os']
tags: ['os', 'engineer information processing']
---

# 운영체제 정리 (정보처리기사)

## 1. 운영체제의 개요

* 운영체제(OS, Operating System)
  하드웨어 자원을 효율적으로 관리하고 사용자가 컴퓨터를 편리하게 사용할 수 있도록 환경을 제공하는 시스템 소프트웨어. 응용 프로그램과 하드웨어 사이에서 인터페이스와 자원 관리자 역할 수행.
* 목표 (성능평가 기준)

  1. 처리능력(Throughput) – 일정 시간 내 처리 작업량
  2. 반환시간(Turnaround Time) – 작업 의뢰부터 완료까지 소요시간
  3. 사용가능도(Availability) – 필요 시 즉시 사용 가능
  4. 신뢰도(Reliability) – 정확한 결과 도출

---

## 2. 운영체제의 운영 방식

* Batch Processing (일괄 처리) – 사용자 개입 없이 작업을 모아 처리
* Time Sharing (시분할) – 여러 사용자가 동시에 사용하는 것처럼 CPU 분할 사용
* Multiprogramming (다중 프로그래밍) – CPU와 I/O 장치 동시 활용
* Real-Time System (실시간 처리) – 입력을 일정 시간 내 즉시 처리

  * Hard RTOS: 마감시간 엄격 보장
  * Soft RTOS: 상대적으로 유연

---

## 3. 프로세스 관리

* 프로세스(Process): 실행 중인 프로그램
* PCB(Process Control Block): PID, 상태, 레지스터 값 등 저장
* 스레드(Thread): 프로세스 내 실행 단위(경량 프로세스)
* 프로세스 상태 전이:

  * 준비(Ready) → 실행(Run) → 대기(Wait) → 준비/종료
  * Dispatch: 준비 → 실행
  * Wake up: 대기 → 준비

---

## 4. 교착 상태 (Deadlock)

### 발생 조건 (필요충분조건)

1. 상호 배제(Mutual Exclusion)
2. 점유와 대기(Hold and Wait)
3. 비선점(Non-preemption)
4. 환형 대기(Circular Wait)

### 해결 방법

* 예방 (Prevention), 회피 (Avoidance, 은행원 알고리즘)
* 발견 (Detection), 회복 (Recovery: 프로세스 종료/자원 선점)

---

## 5. 프로세스 스케줄링

* 목적: CPU 이용률 증가, 응답시간/대기시간 최소화, 공정성 보장
* 스케줄링 종류

  * 장기(Long-term), 중기(Medium-term), 단기(Short-term)
* 알고리즘

  * 비선점형: FCFS, SJF, HRN, 우선순위, Deadline
  * 선점형: RR, SRT, 선점 우선순위, 다단계 큐, 다단계 피드백 큐
  * 보완: 에이징(Aging, 기아 방지)

---

## 6. 기억 장치 관리

* 종류:

  * 주기억장치 (RAM, ROM)
  * 보조기억장치 (HDD, SSD 등)
  * 캐시(Cache), 레지스터(Register)
* 전략:

  * 반입(Fetch): 요구반입, 예상반입
  * 배치(Placement): 최초적합, 최적적합, 최악적합
  * 교체(Replacement): OPT, FIFO, LRU, LFU, NUR, SCR

---

## 7. 가상 기억 장치

* 정의: 보조기억장치 일부를 주기억장치처럼 활용
* 기법

  * 페이징(Paging): 고정 크기 단위, 내부 단편화 발생
  * 세그먼테이션(Segmentation): 논리 단위, 외부 단편화 발생
* 용어

  * 지역성(Locality) – 시간/공간 집중성
  * 워킹 셋(Working Set) – 일정 시간 자주 참조하는 페이지 집합
  * 스래싱(Thrashing) – 과도한 페이지 교체로 성능 저하

---

## 8. 디스크 스케줄링

* FCFS: 단순, 탐색시간 비효율
* SSTF: 가까운 요청 우선, 기아 가능성
* SCAN: 헤드가 왕복하며 처리
* C-SCAN: 한쪽 끝까지 갔다가 처음으로 돌아와 처리 → 균등 서비스

---

## 9. 정보 관리

* 파일 시스템: 디렉토리 구조, 접근 권한 관리, 백업/복구
* I/O 관리: 버퍼링, 장치 드라이버, I/O 스케줄링
* 보안/보호: ACL(Access Control List), 인증, 암호화

---

## 10. 분산 운영체제

* 정의: 네트워크로 연결된 여러 컴퓨터를 하나의 시스템처럼 관리
* 특징: 위치/이주/중복/병행/고장 투명성
* 장점: 자원 공유, 신뢰성 향상, 확장성 우수
* 단점: 복잡성, 지연, 데이터 일관성 문제

---

## 11. UNIX / Linux

### UNIX

* 1960년대 Bell Labs, MIT, GE에서 개발
* 철학: "모든 것은 파일이다", 단순하고 강력한 인터페이스
* 특징: 다중 사용자, 다중 작업 지원 / 계층적 파일 시스템 / 쉘(Shell)과 커널(Kernel) 구조
* 종류: System V, BSD, Solaris, HP-UX 등

### Linux

* 1991년 리누스 토발즈가 개발한 오픈소스 UNIX 계열 OS
* 다양한 배포판: Ubuntu, Fedora, CentOS, Debian 등
* 장점: 무료, 커뮤니티 기반 개발, 높은 보안성, 서버 시장 점유율 높음
* 스케줄러: Completely Fair Scheduler (CFS), 최신 커널은 확장된 스케줄링 알고리즘 지원
* 적용 분야: 서버, 클라우드, 모바일(Android 커널 기반), 슈퍼컴퓨터

---

## 맺음말

> 운영체제는 컴퓨터 자원의 관리자이자 사용자 환경의 제공자로, 프로세스·메모리·파일·입출력 관리 등 핵심 기능을 수행합니다. 
> 정보처리기사 시험에서는 운영체제의 목적, 운영 방식, 스케줄링 알고리즘, 메모리 관리 기법, 교착상태 해결 방법, UNIX/Linux까지 출제됩니다.

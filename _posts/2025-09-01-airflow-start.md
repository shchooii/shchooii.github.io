---
title: Airflow Start
date: 2025-09-01 13:45:58 +09:00
categories: ['airflow']
tags: ['airflow', 'dag', 'task', 'operator']
---

Apache Airflow는 **코드(Python)로 워크플로우를 정의하고 스케줄, 실행, 모니터링**하는 오케스트레이션 플랫폼입니다. 
데이터 파이프라인, 모델 학습 배치, ETL/ELT, 주기적 백오피스 작업 등에 적합합니다. Airflow는 작업 간 **의존성(Dependency)** 을 명시하고, **재시도/에러 처리/알림/로그**를 표준화하며, **UI 기반 모니터링**과 **확장 가능한 실행기(Executor)** 로 운영 편의성을 제공합니다.

---

## Key Concepts

* **DAG (Directed Acyclic Graph)**: 방향 비순환 그래프로 표현되는 워크플로우의 뼈대입니다. 노드는 태스크, 엣지는 의존성을 의미합니다.
* **Task**: DAG를 구성하는 최소 실행 단위입니다(예: 데이터 추출, 변환, 적재 등).
* **Operator**: Task의 템플릿/타입입니다. `PythonOperator`, `BashOperator`, `EmailOperator`, 각종 클라우드/DB 연동용 Provider Operator 등이 있습니다.
* **Sensor**: 특정 조건(파일 도착, 파티션 생성, 외부 DAG 완료 등)을 **폴링** 혹은 **디퍼러블(Deferrable)** 방식으로 기다리는 태스크입니다.
* **Hook**: 외부 시스템 연결을 캡슐화한 재사용 가능한 연결/클라이언트 래퍼입니다.
* **XCom**: 태스크 간 소량의 메타데이터를 교환하는 메커니즘입니다(대용량 데이터 이동은 외부 스토리지 권장).
* **TaskFlow API**: Python 함수 기반으로 태스크와 의존성을 간결히 선언하는 2.x 권장 방식입니다.
* **Connection/Variable**: 외부 자격증명·설정값을 UI/Secret 백엔드로 관리합니다.
* **Pool & Priority Weight**: 동시성 제어/자원 분배 정책을 설정합니다.
* **SLA & 알림**: 지연(Service Level Agreement) 감지 시 알림/이벤트를 발생시킵니다.

---

## Architecture

Airflow는 아래 구성요소로 동작합니다.

* **Webserver**: DAG/태스크 상태 시각화·트리 뷰·로그 열람·수동 실행 등 UI 제공.
* **Scheduler**: 스케줄에 따라 실행할 태스크를 판단하고 큐에 올립니다.
* **Executor**: 태스크를 **어디서/어떻게** 실행할지 결정합니다(예: Local/Celery/Kubernetes).
* **Worker**: Executor 지시에 따라 실제 태스크를 실행합니다.
* **Metadata DB**: DAG/TaskInstance 이력, 스케줄링 메타데이터 저장(RDB 권장).
* **Message Broker(선택)**: CeleryExecutor 사용 시 태스크 큐 브로커(Redis/RabbitMQ)가 필요합니다.
* **Triggerer**: Deferrable Operator 이벤트를 비동기적으로 기다리면서 자원 점유를 최소화합니다.

### Executor 선택 기준

* **LocalExecutor**: 단일 노드 병렬 실행(소규모/개발용).
* **CeleryExecutor**: 다수의 워커 노드를 수평 확장(중대형, 관리 편의 ↑).
* **KubernetesExecutor**: 태스크 단위로 파드를 생성/격리(탄력 확장, 인프라 유연성 ↑).

---

## 디렉터리/배포 패턴

* **표준 레이아웃**:

  ```
  dags/        # DAG 정의 (Python)
  plugins/     # 커스텀 Operator/Hook/Sensor
  include/     # 템플릿/SQL/스크립트
  logs/        # 실행 로그 (원격 로그 스토리지 권장)
  ```
* **로컬 개발**: `airflow standalone` 또는 Docker Compose로 빠른 실험.
* **프로덕션**: Helm 차트(K8s), Celery 클러스터, 원격 로그(S3/GCS), Secret 백엔드(Vault/SM) 조합이 일반적입니다.

---

## 스케줄링/실행 제어

* **Start Date / Schedule**: 크론/프리셋/데이터셋 기반 트리거로 주기 실행을 정의합니다.
* **Catchup / Backfill**: 미실행 구간을 재처리할지(Catchup)와 과거 기간을 메우는 실행(Backfill) 전략을 분리합니다.
* **병렬성(Concurrency)**: `dag_concurrency`, `max_active_runs`, 태스크 `retries`, `retry_delay`, `retry_exponential_backoff`로 실행 폭과 내결함성을 조절합니다.
* **Trigger Rules**: `all_success`, `one_failed`, `all_done` 등 후행 태스크의 실행 조건을 명시합니다.

---

## 운영·보안(Operations & Security)

* **RBAC**: 역할 기반 접근 제어로 UI/리소스 권한을 분리합니다.
* **Secret Backend**: 자격증명·커넥션을 Vault/AWS Secrets Manager/GCP Secret Manager 등으로 위임합니다.
* **Observability**: UI + 로그 + 메트릭(StatsD/Prometheus) + 알림(Slack/Email) 연계.
* **신뢰성**: 태스크 **멱등성(Idempotency)** 확보, 외부 의존 실패를 고려한 재시도/타임아웃 설정, 원자적 커밋/체크포인팅 권장.
* **비용/성능**: Deferrable Sensor로 대기 비용 절감, K8sExecutor로 탄력 확장, Pool로 스파이크 제어.

---

## 베스트 프랙티스(Practical Tips)

1. **작게 쪼개고 명확히 연결**: 태스크는 단일 책임 원칙, 의존성은 최소/명시적으로 설계합니다.
2. **데이터 이동은 외부 스토리지**: XCom은 메타데이터만, 대용량은 S3/GCS/HDFS 등 사용합니다.
3. **센서는 디퍼러블 우선**: Idle 자원 점유를 줄이고 확장성을 높입니다.
4. **구성은 코드로**: DAG/플러그인/요구사항을 Git에 버전 관리하고 CI로 정적 검증(린트/유닛 테스트)합니다.
5. **스케줄은 “데이터 가용성” 기준**: 소스 생성 주기·SLA를 기준으로 스케줄을 맞추고 `max_active_runs`로 겹침 실행을 제어합니다.
6. **관측 가능성 확보**: 표준 로깅·메트릭·알림 채널을 초기부터 통합합니다.

---

## 최소 예제(DAG · TaskFlow API)

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.decorators import task
from airflow.operators.empty import EmptyOperator
from airflow.utils.trigger_rule import TriggerRule

default_args = {
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
}

with DAG(
    dag_id="example_taskflow_v1",
    start_date=datetime(2025, 1, 1),
    schedule="0 * * * *",  # 매시 정각
    catchup=False,
    default_args=default_args,
    tags=["example"],
) as dag:

    start = EmptyOperator(task_id="start")

    @task
    def extract():
        # 외부에서 데이터를 읽어와 경로/키 등 메타데이터만 리턴
        return {"path": "s3://bucket/raw/2025-09-19/data.parquet"}

    @task
    def transform(meta: dict):
        # 멱등성 보장: 같은 입력이면 같은 출력 산출
        # (실제 변환 로직/임시 경로/체크포인트 전략 권장)
        return {"path": meta["path"].replace("raw", "curated")}

    @task(trigger_rule=TriggerRule.ALL_DONE)
    def load(meta: dict):
        # 실패/부분성공 케이스 고려한 커밋 전략 필요
        print(f"Load from {meta['path']}")

    end = EmptyOperator(task_id="end")

    start >> extract() >> transform() >> load() >> end
```

> 포인트: `catchup=False`, 멱등성 패턴, 재시도/트리거 규칙, 메타데이터만 XCom으로 전달.

---

## 일반적 장애 원인 & 해결

* **Queued에서 멈춤**: Pool/Concurrency 제한 과부하, Executor/워커 스케일 확인.
* **Timezone 착오**: Airflow는 기본 UTC입니다. 스케줄과 타임스탬프 기준을 통일합니다.
* **Broken DAG**: 패키지 미설치·모듈 경로 오류·순환 의존성 점검, 로컬 `airflow tasks test`로 단위 검증.
* **Sensor 병목**: 디퍼러블 전환·타임아웃/소프트 타임아웃 설정.

---

## 맺음말

> Airflow는 **의존성 기반의 배치 파이프라인을 코드로 선언**하고, 팀이 **일관된 운영 표준**을 갖추도록 돕습니다. 
> 단일 노드에서 시작해 Celery/Kubernetes로 손쉽게 확장할 수 있으며, RBAC·Secret 백엔드·관측 가능성 도구와 결합하여 **신뢰성 높은 데이터/ML 운영**을 구현합니다. 

> 이벤트 스트리밍·실시간 초저지연 처리에는 전용 스트리밍 프레임워크가 더 적합하므로 **업무 특성에 맞는 역할 분리**가 중요합니다. 
> 초기에는 **작게 시작하여 표준과 자동화**를 갖추고, **멱등성/관측/보안**을 기본 원칙으로 삼는 것이 좋습니다.

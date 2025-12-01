---
title: Airflow DAG
date: 2025-09-08 20:49:58 +09:00
categories: ['airflow']
tags: ['airflow', 'dag', 'smtp']
---

이 문서는 MacOS 환경에서 Apache Airflow를 설치하고, 기본 프로세스 실행 방식과 이메일 알림(SMTP) 설정 과정을 정리한 문서입니다.  
특히 Airflow 3.x 버전 기준으로 `api-server`, `scheduler`, `dag-processor` 3가지 프로세스를 실행해야 하는 이유와, DAG 실패 시 이메일 알림이 오도록 설정하는 과정을 단계별로 설명합니다.


## Airflow 주요 프로세스

### 1. API Server (`airflow api-server`)
- 역할: Airflow UI(Web UI) 및 REST API를 제공
- 브라우저에서 DAG 상태 확인, 실행 트리거 등을 할 수 있음
- 보통 포트 8080으로 실행 → `http://localhost:8080` 접속

### 2. Scheduler (`airflow scheduler`)
- 역할: DAG의 실행 스케줄을 관리
- DAG 정의를 읽고, 실행 시점이 되면 Task를 큐에 넣어 실행하도록 지시
- 실질적으로 DAG 실행 여부를 컨트롤하는 핵심 프로세스

### 3. DAG Processor (`airflow dag-processor`)
- 역할: Airflow 3.x에서 추가된 별도 프로세스
- DAG 파일을 파싱하고 DagBag에 반영하는 역할
- DAG을 UI에서 인식시키려면 반드시 실행 필요

👉 따라서 Airflow 3.x에서는 최소 3개 프로세스를 동시에 실행해야 함  
(`api-server`, `scheduler`, `dag-processor`)

---

## 기본 실행 순서

```bash
# 터미널 1 - API Server 실행
source ~/venv/bin/activate
export AIRFLOW_HOME=~/venv/airflow_home
airflow api-server

# 터미널 2 - Scheduler 실행
source ~/venv/bin/activate
export AIRFLOW_HOME=~/venv/airflow_home
airflow scheduler

# 터미널 3 - DAG Processor 실행
source ~/venv/bin/activate
export AIRFLOW_HOME=~/venv/airflow_home
airflow dag-processor
```

---

## DAG 예시 (실패 → 이메일 알림)

`$AIRFLOW_HOME/dags/hello_email_dag.py`

```python
from airflow.models import DAG
from airflow.providers.standard.operators.bash import BashOperator
from datetime import datetime, timedelta

default_args = {
    "owner": "airflow",
    "depends_on_past": False,
    "email": ["수신자메일주소@gmail.com"],  # 알림 받을 메일
    "email_on_failure": True,
    "email_on_retry": False,
    "retries": 1,
    "retry_delay": timedelta(minutes=1),
}

with DAG(
    dag_id="hello_email_dag",
    default_args=default_args,
    description="A simple DAG with email alert",
    schedule=None,
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=["example"],
) as dag:

    t1 = BashOperator(
        task_id="print_date",
        bash_command="date"
    )

    t2 = BashOperator(
        task_id="fail_task",
        bash_command="exit 1"  # 실패 → 이메일 알림 확인
    )

    t1 >> t2
```

---

## 이메일(SMTP) 설정

Airflow는 자체 메일 서버가 없으므로 외부 SMTP 서버(Gmail, Naver 등)를 설정해야 합니다.  
이번 예시는 Naver SMTP + 앱 비밀번호 방식입니다.

### airflow.cfg 설정 예시

`$AIRFLOW_HOME/airflow.cfg` → `[smtp]` 섹션 수정

```ini
[smtp]
smtp_host = smtp.naver.com
smtp_starttls = False
smtp_ssl = True
smtp_user = your_id@naver.com
smtp_password = 앱비밀번호
smtp_port = 465
smtp_mail_from = your_id@naver.com
```

- `smtp_user`, `smtp_mail_from`는 반드시 로그인 계정과 동일해야 함
- `smtp_password`는 네이버에서 발급받은 앱 비밀번호(12자리)

---

## 테스트 및 확인
1. Airflow 프로세스 3개 실행 (`api-server`, `scheduler`, `dag-processor`)
2. UI에서 `hello_email_dag` ON → Trigger DAG
3. `fail_task` 실패 시 → `default_args["email"]`로 알림 메일 수신 확인

---

## 맺음말
> Airflow 3.x에서는 DAG 실행을 위해 세 가지 프로세스(api-server, scheduler, dag-processor)가 모두 필요합니다.
> 알림 메일을 보내기 위해서는 SMTP 설정이 반드시 완료되어야 하며, 네이버/구글 계정의 경우 앱 비밀번호 발급이 필수입니다.

> 이번 과정을 통해 Airflow DAG 실행 → 실패 알림 메일 발송까지 기본적인 워크플로우를 구축할 수 있습니다.
> 실제 서비스에서는 Slack, Teams 같은 협업 툴 알림 연동도 가능합니다. 🚀

---


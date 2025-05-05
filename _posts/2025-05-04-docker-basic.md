---
title: Docker Basic
date: 2025-05-04 10:37:20 +09:00
categories: ['docker']
tags: ['docker', 'Dockerfile', 'docker-compose', 'docker-network']
---

현대 애플리케이션은 **다중 서비스**(예: Flask + Elasticsearch)로 구성되는 경우가 많습니다.  
각 서비스를 배포하고 실행하려면 다음 세 가지가 필수입니다.

| 도구 | 핵심 역할 | 얻는 이점 |
|------|-----------|-----------|
| **Dockerfile** | 애플리케이션과 모든 의존성을 _이미지_ 로 정의 | ✔️ 환경 재현성<br>✔️ CI/CD 자동화<br>✔️ 버전 관리 |
| **Docker Compose** | 여러 컨테이너를 한 파일로 선언·오케스트레이션 | ✔️ `docker compose up` 한 번으로 풀-스택 실행<br>✔️ 환경 변수·볼륨·네트워크를 코드화 |
| **Docker Network** | 컨테이너 간 통신 경로와 격리를 설정 | ✔️ 서비스 발견(컨테이너 이름 → IP 자동 매핑)<br>✔️ 보안 격리(네트워크별 방화벽)<br>✔️ 마이크로서비스 확장성 |

이 세 가지를 조합하면 **개발–테스트–운영 파이프라인**을 빠르게 구축할 수 있습니다.

---

## 1. Dockerfile 기본

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.10-slim                # 1. 베이스 이미지
WORKDIR /opt/app                      # 2. 작업 디렉터리
COPY requirements.txt .               # 3. 의존성 복사
RUN pip install --no-cache-dir -r requirements.txt
COPY . .                              # 4. 애플리케이션 코드 복사
EXPOSE 5000                           # 5. 컨테이너 외부로 공개할 포트
ENV FLASK_ENV=production              # 6. 환경 변수
CMD ["python", "app.py"]              # 7. 기본 실행 명령
````

### 자주 쓰는 지시어

* **FROM** : 기본 이미지
* **RUN** : 이미지 빌드 시 한 번 실행
* **CMD / ENTRYPOINT** : 컨테이너 시작 시 실행
* **COPY / ADD** : 파일·디렉터리 복사
* **ENV / ARG** : 설정값 주입
* **EXPOSE** : 문서용—실제 포트 매핑은 `-p` 또는 Compose에서 정의
* **HEALTHCHECK** : 컨테이너 헬스 확인

> **TIP** : `.dockerignore`에 불필요한 파일을 제외할 수 있습니다.


## 2. Docker Compose 기본

### 2-컨테이너 예시 (`docker-compose.yml`)

```yaml
version: "3.9"

services:
  es:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.2
    container_name: es
    environment:
      discovery.type: single-node
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - foodtrucks-net

  web:
    build: .                # 동일 디렉터리 Dockerfile 사용
    container_name: foodtrucks-web
    depends_on:
      - es
    ports:
      - "5000:5000"
    networks:
      - foodtrucks-net

networks:
  foodtrucks-net:
    driver: bridge
```

### 구성 요소

* **services** : 개별 컨테이너 정의
* **build / image** : 로컬 Dockerfile 빌드 or 레지스트리 이미지 사용
* **depends\_on** : 의존 순서 명시 (시작 순서를 보장하진 않음—헬스체크 권장)
* **ports / volumes / environment** : 런타임 옵션
* **networks** : 서비스 간 통신 범위 지정 (생략 시 Compose 프로젝트 전용 네트워크 자동 생성)

```bash
# 전체 스택 올리기
docker compose up -d

# 로그 보기
docker compose logs -f web

# 중단·정리
docker compose down
```

## 3. Docker Network

| 네트워크 모드         | 특징                        | 활용                    |
| --------------- | ------------------------- | --------------------- |
| **bridge (기본)** | 같은 호스트 내 사용자 정의 서브넷·IP 할당 | 단일 머신 개발·테스트          |
| **host**        | 컨테이너가 호스트 네트워크 스택 공유      | 네트워크 오버헤드 최소화(리눅스 전용) |
| **none**        | 네트워크 미연결                  | 보안 실험                 |
| **overlay**     | 다중 호스트-스웜/쿠버네티스           | 프로덕션 클러스터             |

> Compose에서 `driver: bridge`를 지정하면 위에서 만든 `foodtrucks-net`과 같은 **사용자 정의 브리지**가 생성됩니다.
> 컨테이너는 **서비스 이름**(예: `es`)으로 서로를 DNS 해결할 수 있어 IP 변경에 영향받지 않습니다.

### 3-1. 컨테이너별 포트·IP 확인

```bash
# 컨테이너 내부 IP 확인
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' es
# → 172.20.0.2 (예시)

# 호스트 ↔ 컨테이너 포트 매핑 확인
docker port es
# → 9200/tcp -> 0.0.0.0:9200
```

* **내부 통신** : `web → es:9200` (DNS 이름 사용)
* **외부 노출** : `0.0.0.0:9200 → es:9200` (호스트 ↔ 컨테이너 매핑)
* 컨테이너 재시작해도 **이름 기반** 통신은 안전함.
* IP가 바뀌어도 DNS가 매핑해줌.


## 4. 예시: DB + Health Check + 서버

```yaml
version: "3.9"

services:
  db:
    image: postgres:16
    container_name: foodtrucks-db
    environment:
      POSTGRES_USER: food
      POSTGRES_PASSWORD: trucks
      POSTGRES_DB: foodtrucks
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U food"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - foodtrucks-net

  es:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.2
    container_name: es
    environment:
      discovery.type: single-node
    ports:
      - "9200:9200"
    networks:
      - foodtrucks-net

  web:
    build: .
    container_name: foodtrucks-web
    depends_on:
      db:
        condition: service_healthy     # DB가 준비된 후 시작
      es:
        condition: service_started     # ES 컨테이너가 시작되면 즉시
    environment:
      DATABASE_URL: postgres://food:trucks@db:5432/foodtrucks
    ports:
      - "5000:5000"
    networks:
      - foodtrucks-net

networks:
  foodtrucks-net:
    driver: bridge

volumes:
  db-data:
```

### 작동 흐름

1. **db** 컨테이너가 기동되고 `pg_isready` Health Check를 통과해야 *healthy* 상태가 됩니다.
2. **web** 컨테이너는 `db`가 *healthy*가 될 때까지 대기하고, Elasticsearch 컨테이너가 *started*되면 즉시 연결을 시도합니다.
3. 세 컨테이너는 모두 `foodtrucks-net` 브리지에서 DNS 이름(`db`, `es`)으로 서로를 찾습니다.

> `service_healthy` 조건은 Compose spec(v3.4+)에서 지원됩니다.

## 5. CLI vs Compose 정리

| 방식                          | 명령 수               | 학습 난도 | 협업·재현성          |
| --------------------------- | ------------------ | ----- | --------------- |
| **단순 CLI** (`docker run …`) | 적을 때는 빠름           | 낮음    | 사람이 스크립트 관리     |
| **Compose**                 | 한 줄 (`compose up`) | 중     | 설정 코드화·버전 관리 용이 |

규모가 커지면 **Compose → 쿠버네티스(Helm, Kustomize)** 로 확장할 수 있습니다.

---

## 맺음말

> Dockerfile로 **이미지**를 표준화하고, Docker Compose로 **다중 서비스**를 선언적으로 관리하면
> “어디서나 같은 방식으로” 애플리케이션을 실행할 수 있습니다.

> 도움이 되었기를 바랍니다.

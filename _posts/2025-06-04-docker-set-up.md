---
title: Docker Set Up
date: 2025-06-04 09:37:20 +09:00
categories: ['docker']
tags: ['docker', 'dockerfile', 'harbor']
---


GPU 기반 딥러닝 프로젝트를 위해서는 **CUDA**, **cuDNN**, **PyTorch GPU wheel**, 그리고 프로젝트 환경을 손쉽게 관리할 수 있는 **Conda**가 필요합니다.
아래에서는 NVIDIA 공식 CUDA 런타임 이미지를 베이스로, Conda 환경을 만들고 GPU용 PyTorch를 설치한 뒤 Harbor 레지스트리에 푸시하여 리눅스 GPU 서버에서 실행하는 과정을 정리합니다.

---

## Dockerfile 구조

```dockerfile
# 1) CUDA 런타임 베이스 이미지
FROM --platform=linux/amd64 nvidia/cuda:12.1.1-cudnn8-runtime-ubuntu22.04

ARG ENV_NAME=coding
ARG PYTHON_VERSION=3.10

# 2) 시스템 패키지 설치
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      git build-essential cargo rustc wget bzip2 && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 3) Miniconda 설치
ENV CONDA_DIR=/opt/conda
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
      -O /tmp/conda.sh && \
    bash /tmp/conda.sh -b -p $CONDA_DIR && rm /tmp/conda.sh
ENV PATH=$CONDA_DIR/bin:$PATH

# 4) Conda env 생성
RUN conda create -y -n $ENV_NAME python=$PYTHON_VERSION && conda clean -afy

# 5) SHELL 을 coding env 로 설정
SHELL ["conda", "run", "-n", "coding", "/bin/bash", "-c"]

WORKDIR /home/tabular/shchoi
COPY . .

# 6) pip 업데이트 및 GPU용 PyTorch + 필수 패키지 설치
RUN pip install --upgrade pip && \
    pip install --no-cache-dir \
      torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu121 && \
    pip install --no-cache-dir \
      hydra-core omegaconf wandb scikit-learn scipy

# 7) 로컬 패키지 설치
RUN pip install -e .
```

---

## 단계별 설명

1. **CUDA 런타임**

  * `nvidia/cuda:12.1.1-cudnn8-runtime-ubuntu22.04` 이미지를 사용해 CUDA + cuDNN 환경을 준비합니다.
  * `--platform=linux/amd64` 옵션으로 x86\_64 아키텍처를 명시합니다. (리눅스 GPU 서버 기준)

2. **시스템 패키지 설치**

  * `git`, `build-essential`, `cargo`, `rustc` 등 기본 개발 툴을 설치합니다.
  * 불필요한 캐시는 제거해 이미지 크기를 줄입니다.

3. **Miniconda 설치**

  * `/opt/conda` 경로에 설치하고 PATH에 추가합니다.

4. **Conda 환경 생성**

  * `$ENV_NAME` (`coding`), `$PYTHON_VERSION` (3.10)을 기준으로 새 환경을 생성합니다.

5. **SHELL 전환**

  * `SHELL ["conda", "run", "-n", "coding", "/bin/bash", "-c"]`를 통해 이후 모든 명령이 지정된 Conda env 안에서 실행되도록 합니다.

6. **PyTorch + 필수 패키지 설치**

  * `--extra-index-url https://download.pytorch.org/whl/cu121`를 사용해 **CUDA 12.1 대응 PyTorch wheel**을 가져옵니다.
  * `hydra-core`, `omegaconf`, `wandb`, `scikit-learn`, `scipy` 등 실험 관리와 학습에 필요한 필수 라이브러리를 추가합니다.

7. **프로젝트 패키지 설치**

  * `pip install -e .`로 로컬 프로젝트를 개발 모드로 설치합니다.

---

## 빌드 & 푸시

1. **Harbor 로그인**

```bash
docker login registry.XXXX.ai
```

2. **buildx로 amd64 빌드 & 푸시**

```bash
docker buildx build \
  --platform linux/amd64 \
  -t registry.XXXX.ai/medical-coding/icd-coding:latest \
  --push .
```

---

## 실행 & 검증 (GPU 서버)

1. **GPU 노출 확인**

```bash
nvidia-smi
```

2. **컨테이너 실행**

```bash
docker run --rm --gpus all \
  registry.XXXX.ai/medical-coding/icd-coding:latest \
  python - <<'PY'
import torch
print("CUDA:", torch.version.cuda)
print("is_available:", torch.cuda.is_available())
print("device_count:", torch.cuda.device_count())
PY
```

* 기대 출력: `CUDA: 12.1`, `is_available: True`, `device_count >= 1`

---

## 자주 발생하는 문제

* **CUDA 인식 안 됨**

  * `--gpus all` 누락 → 옵션 추가
  * NVIDIA Container Toolkit 미설치 → 설치 후 Docker 재시작
  * CPU wheel 설치됨 → `--index-url`로 GPU wheel 고정
  * Mac에서 실행 시 → GPU 미지원 (리눅스 서버에서 실행 필요)

---

## 맺음말

> 이 Dockerfile은 **GPU 학습 환경을 표준화**하고, Harbor에 올립니다. 
> 로컬에서는 단순히 빌드, 푸시만 하고, 실제 학습에서는 GPU 서버에서 `--gpus all` 옵션으로 실행하면 됩니다. 
> 도움이 되었기를 바랍니다.

---

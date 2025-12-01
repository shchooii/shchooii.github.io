---
title: Kubernetes Set Up
date: 2025-06-14 09:40:20 +09:00
categories: ['docker']
tags: ['docker', 'kubernetes', 'manifest']
---

`ICD-Coding` 프로젝트를 쿠버네티스 환경에서 실행하기 위한 Deployment, Service, Job manifest 예시와 설정 방법을 정리하였습니다.

---

## 1. Deployment

GPU 자원을 활용하여 SSH 접속 가능한 컨테이너를 띄우기 위한 설정입니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ssh-csh # 서비스 라벨, 자유롭게 변경 가능
  name: ssh-csh-deployment # Deployment 이름, -deployment 형태 권장
spec:
  replicas: 1 # Pod 개수
  selector:
    matchLabels:
      app: ssh-csh # 위의 라벨과 동일해야 함
  template:
    metadata:
      labels:
        app: ssh-csh # 동일 label 유지
    spec:
      imagePullSecrets:
        - name: harbor # 프라이빗 레지스트리 인증 시 필요

      securityContext:
        runAsUser: 0 # UID (root 대신 계정에 맞게 변경 권장)
        runAsGroup: 0 # GID (계정 그룹에 맞게 변경)
        fsGroup: 1007 # 파일 시스템 그룹 ID, 계정에 맞게 수정
      affinity:
        nodeAffinity: # 특정 노드에 스케줄링 강제
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: nvidia.com/gpu.product
                    operator: In
                    values:
                      - NVIDIA-RTX-4000-Ada-Generation # 원하는 GPU 모델
      restartPolicy: Always # Pod 재시작 정책
      volumes:
        - name: shmdir
          emptyDir:
            medium: Memory # 메모리 기반 공유 디렉토리
        - name: pvc-volume
          persistentVolumeClaim:
            claimName: XXXX-nfs # PVC 이름, 환경에 맞게 수정
      containers:
        - name: gpu-container # 컨테이너 이름
          image: registry.XXXX.ai/model-medical/icd-coding # 컨테이너 이미지
          volumeMounts:
            - mountPath: /dev/shm
              name: shmdir
            - mountPath: /home/ # 실제 NFS 경로 필요시 변경
              name: pvc-volume
          env:
            - name: GIT_TOKEN
              valueFrom:
                secretKeyRef:
                  name: gitlab # Git 인증 시크릿
                  key: gitlab-token
            - name: HOME
              value: /workspace/
            - name: PYTHONUSERBASE
              value: /workspace/.local
          command: ["/bin/sh", "-c"]
          args:
            - |
              set -x
              groupadd -g 1007 XXXX
              useradd -m -d /workspace -s /bin/bash -u 1007 -g XXXX XXXX
              usermod -aG sudo XXXX
              echo "XXXX:password" | chpasswd

              chown XXXX:XXXX /workspace
              chown XXXX:XXXX /home/tabular/icd-coding

              apt-get update
              apt-get install -y --reinstall ca-certificates git openssh-server

              pip install -U scikit-learn scipy

              cd /workspace
              runuser -u XXXX -- git clone -b binary-simple https://XXXX:$GIT_TOKEN@git.XXXX.ai/ai/sleep-apnea/ecgdualnet.git dualnet
              service ssh start
              sleep infinity
          resources:
            requests:
              nvidia.com/gpu: 1 # 최소 GPU 요구
            limits:
              nvidia.com/gpu: 1 # 최대 GPU 사용 제한
          livenessProbe:
            exec:
              command: ["nvidia-smi"] # GPU 상태 체크
            initialDelaySeconds: 10
            periodSeconds: 30
```

---

## 2. Service

외부에서 Pod에 SSH 접속할 수 있도록 노출하는 설정입니다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ssh-csh-service # 서비스 이름
  labels:
    app: ssh-csh # 라벨, Deployment와 동일해야 함
spec:
  selector:
    app: ssh-csh # Deployment와 동일해야 함
  ports:
    - protocol: TCP
      name: ssh
      port: 22 # 외부에 노출될 포트
      targetPort: 22 # 컨테이너 내부 포트
  type: LoadBalancer # 외부 접근 허용
```

---

## 3. Job

모델 학습/실험 실행을 위한 일회성 잡(Job) 설정입니다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: XXXX-icdcoding-job # Job 이름
  namespace: XXXX # 네임스페이스
spec:
  ttlSecondsAfterFinished: 300 # 완료 후 300초 뒤 삭제
  template:
    spec:
      restartPolicy: Never # 실패 시 재시작하지 않음
      securityContext:
        fsGroup: 1007 # 그룹 ID
        runAsUser: 1007 # 사용자 UID
        runAsGroup: 1007 # 사용자 GID
      imagePullSecrets:
        - name: harbor # 프라이빗 레지스트리 인증
      volumes:
        - name: shmdir
          emptyDir:
            medium: Memory
        - name: pvc-volume
          persistentVolumeClaim:
            claimName: XXXX-nfs # PVC 이름
      containers:
        - name: gpu-container
          image: registry.XXXX.ai/model-medical/icd-coding
          command: ["/bin/sh", "-c"]
          volumeMounts:
            - mountPath: /dev/shm
              name: shmdir
            - mountPath: /home/
              name: pvc-volume
          env:
            - name: WANDB_API_KEY
              valueFrom:
                secretKeyRef:
                  name: wandb
                  key: wandb-api-key
            - name: HOME
              value: /home/XXXX/tabular/icd-coding
          args:
            - |
              set -x
              export PATH=/opt/conda/envs/coding/bin:$PATH
              cd "$HOME"
              which python
              python -c "import torch; print('torch module at', torch.__file__)"
              python -c "import torch; print('cuda version:', torch.version.cuda, 'available:', torch.cuda.is_available())"
              exec python main.py experiment=mimiciv_icd10/plm_icd gpu=0 load_model=/home/XXXX/tabular/icd-coding/experiments/ylgrf0rl trainer.epochs=0
          resources:
            requests:
              nvidia.com/gpu: 1
            limits:
              nvidia.com/gpu: 1
      nodeSelector:
        role: train # 특정 role의 노드에만 스케줄링
  backoffLimit: 3 # 재시도 횟수 제한
```

---

## 주요 변경 포인트

- `app`, `name`, `claimName`, `namespace` → 환경 맞게 수정 필요
- `image` → 실제 사용하는 레지스트리 경로로 변경
- `UID`, `GID` → 계정에 맞게 조정
- `mountPath` → 실제 NFS 마운트 경로 확인 필요
- SSH 기본 비밀번호(`password`) → 접속 후 반드시 변경

---

## 맺음말

> 위 manifest는 ICD-Coding 프로젝트를 GPU 클러스터 환경에서 안정적으로 실행하기 위해 설계되었습니다.
> Deployment는 SSH 접근용, Service는 외부 노출, Job은 학습 실행을 담당합니다.
> 실제 사용 시에는 계정 및 저장소 경로, 보안 설정(비밀번호·토큰 등)을 반드시 환경에 맞게 수정해야 합니다.

> 안정적인 연구 및 개발 환경 구축에 참고하시기 바랍니다. 🚀


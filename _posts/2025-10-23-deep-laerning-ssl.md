---
title: Deep Learning Self-supervised Learning
date: 2025-10-23 12:52:58 +09:00
categories: ['deep learning']
tags: ['deep learning', 'self-supervised learning']
math: true
---

## Pretext Task의 등장
대규모 딥러닝 학습에는 막대한 양의 레이블링 된 데이터가 필요합니다. SSL은 레이블이 없는 데이터(Unlabeled data)로부터 스스로 문제를 정의하고 정답(Label)을 생성하여 학습하는 Pretext Task를 통해 이 문제를 해결하고자 합니다.

### 주요 Image Transformation 기반 Pretext Tasks
초기 연구들은 이미지를 변형시키고, 모델이 원본을 예측하거나 변형된 속성을 맞추도록 설계되었습니다.

* Rotation Prediction (회전 예측): 이미지를 0도, 90도, 180도, 270도로 회전시킨 후, 모델이 회전 각도를 예측(4-way classification)하게 합니다. 
  * 가설: 객체의 올바른 회전을 예측하려면 객체의 시각적 상식(Visual commonsense)을 이해해야 합니다. 
* Jigsaw Puzzles: 이미지를 여러 패치로 나누고 섞은 뒤, 원래의 배열 순서를 예측합니다. 
* Inpainting (Context Encoders): 이미지의 일부를 지우고(Masking), 주변 문맥을 보고 지워진 픽셀을 복원합니다. 
  * Loss Function: 재구성 손실($$L_{recon}$$)과 적대적 손실($$L_{adv}$$)을 결합하여 사용합니다. 
  * $$L(x) = L_{recon}(x) + L_{adv}(x)$$
  * $$L_{recon}(x) = || M * (x - F_{\theta}((1-M)*x)) ||_2^2$$
    * (여기서 $$M$$은 마스크, $$F$$는 모델입니다.)

## Contrastive Representation Learning (대조 학습)

Pretext Task를 일일이 설계하는 것의 한계를 극복하기 위해 등장했습니다. 데이터 간의 유사성을 학습하는 것이 핵심입니다.
* 핵심 원리: 같은 이미지에서 나온 뷰(Positive pair)는 특징 공간에서 가깝게(Attract), 다른 이미지(Negative pair)는 멀어지게(Repel) 학습합니다.

### InfoNCE Loss (Information Noise-Contrastive Estimation)
대조 학습의 표준이 되는 손실 함수입니다. $$N$$개의 샘플 중 1개의 Positive sample을 찾아내는 $$N$$-way classification 문제로 볼 수 있습니다.

$$L = -\mathbb{E}_{x} \left[ \log \frac{\exp(s(f(x), f(x^+)) / \tau)}{\exp(s(f(x), f(x^+)) / \tau) + \sum_{j=1}^{N-1} \exp(s(f(x), f(x_j^-)) / \tau)} \right]$$

* $$f(x)$$: 인코더를 통과한 표현 벡터
* $$x^+$$: Positive sample (데이터 증강된 동일 이미지)
* $$x^-$$: Negative sample (다른 이미지)
* $$s(u, v)$$: 코사인 유사도
* $$\tau$$: Temperature 파라미터

### SimCLR (Simple Framework for Contrastive Learning)

SimCLR는 간단한 구조로 강력한 성능을 달성했습니다.
* 데이터 증강 (Data Augmentation): Random Cropping과 Color Distortion이 매우 중요합니다.
* Projection Head: 인코더 $$f(\cdot)$$ 뒤에 비선형 변환 $$g(\cdot)$$ (MLP)를 추가하여 대조 학습을 수행하고, 이후 $$g(\cdot)$$는 버립니다.
* Large Batch Size: Negative sample의 수를 늘리기 위해 매우 큰 배치 크기(예: 4096, 8192)가 필요합니다.

### MoCo (Momentum Contrast)

SimCLR의 대규모 배치 사이즈 요구(메모리 문제)를 해결하기 위해 Queue(큐)와 Momentum Encoder를 도입했습니다.
* Queue: 미니 배치 크기와 상관없이 많은 수의 Negative sample을 저장하는 딕셔너리 역할을 합니다.
* Momentum Update: Key Encoder($$\theta_k$$)는 역전파(Gradient)로 업데이트하지 않고, Query Encoder($$\theta_q$$)의 파라미터를 천천히 따라가도록(Moving Average) 업데이트합니다.
  * $$\theta_k \leftarrow m\theta_k + (1-m)\theta_q$$
* MoCo v2: SimCLR의 장점인 MLP Projection head와 강력한 데이터 증강을 MoCo에 적용하여 성능을 높였습니다.

## Reconstruction & Self-Distillation

### Masked Autoencoders (MAE)

ViT(Vision Transformer) 구조를 활용한 생성적(Generative) 접근 방식입니다. BERT의 MLM(Masked Language Modeling)을 이미지에 적용했습니다.
* 높은 마스킹 비율 (High Masking Ratio): 이미지 패치의 약 75%를 마스킹하여 문제를 어렵게 만듭니다. 이는 모델이 전체적인 문맥과 의미를 학습하게 유도합니다.
* 비대칭 구조 (Asymmetrical Architecture):
  * Encoder: 마스킹되지 않은(Visible) 패치(약 25%)만 입력으로 받습니다. 연산량이 크게 감소합니다.
  * Decoder: 인코더 출력과 마스크 토큰을 결합하여 원본 이미지를 복원합니다. 학습 시에만 사용됩니다.
* Loss: 마스킹된 패치 부분의 픽셀 값에 대한 MSE(Mean Squared Error)를 계산합니다.

### DINO (Self-Distillation with No Labels)

레이블 없이 지식 증류(Knowledge Distillation) 방식을 사용합니다.
* 구조: Teacher 네트워크와 Student 네트워크가 동일한 구조를 가집니다.
* 업데이트: Student는 역전파로 학습하고, Teacher는 Student의 파라미터를 지수 이동 평균(EMA)으로 업데이트합니다.
  * $$\theta_{teacher} \leftarrow \tau \cdot \theta_{teacher} + (1-\tau) \cdot \theta_{student}$$
* Centering & Sharpening: Negative sample을 사용하지 않으므로, 모델 붕괴(Collapse)를 막기 위해 Teacher의 출력에 Centering(평균 빼기)과 Sharpening(Temperature 조절)을 적용합니다.
* 특징: DINO로 학습된 ViT의 어텐션 맵(Attention Map)을 시각화하면, 지도 학습 없이도 객체의 형태를 정교하게 분할(Segmentation)하는 특성이 나타납니다.

## 맺음말 
> Self-supervised Learning(SSL)은 레이블링 비용이 높은 지도 학습의 한계를 극복하기 위해, 초기에는 회전 예측이나 직소 퍼즐과 같이 사람이 설계한 Pretext Task를 통해 시각적 상식을 학습하는 방식에서 시작했습니다. 
> 이후 데이터 간의 유사성을 구별하는 Contrastive Learning(SimCLR, MoCo)으로 발전하며 지도 학습에 버금가는 성능을 입증했고, 최근에는 MAE의 이미지 복원 방식이나 DINO의 자기 증류 기법이 도입되어 객체 분할과 같은 고차원적인 특징까지 스스로 학습하는 단계로 진화했습니다. 
> SSL은 대규모 비지도 데이터로부터 풍부한 표현(Representation)을 효과적으로 추출하는 핵심 방법론으로 자리 잡았습니다.

---
title: Deep Learning Optimization
date: 2025-08-18 16:49:20 +09:00
categories: ['deep learning']
tags: ['deep learning', 'optimization', 'adam']
math: true
---

오늘 수업에서는 **이미지 분류(Image Classification)** 문제를 예시로 하여,  
모델 학습에서 중요한 세 가지 핵심 요소를 다루었습니다.

1. **Score Function**
  - 입력된 이미지 픽셀을 클래스 점수로 변환합니다.
  - 예시: **선형 함수**
    $$
    f(x_i, W) = W x_i
    $$

2. **Loss Function**
  - 모델의 파라미터의 성능을 측정합니다.
  - 예측과 정답 레이블의 일치 정도를 평가합니다.
  - 대표적 예시는 다음과 같습니다.
    - **Softmax Loss**  
      $$
      L_i = -\log \frac{e^{s_{y_i}}}{\sum_j e^{s_j}}
      $$
    - **SVM Loss**  
      $$
      L_i = \sum_{j \neq y_i} \max(0, s_j - s_{y_i} + \Delta)
      $$

3. **Optimization**
  - Loss를 최소화하는 최적의 파라미터 $W$를 찾는 과정입니다.
    $$
    W^* = \arg\min_W L(W)
    $$

---

## Loss Function 시각화
- **고차원 파라미터 공간**에서는 Loss 함수 전체를 직접 시각화하기 어렵습니다.  
  (예: CIFAR-10 → $10 \times 3073 = 30,730$ 파라미터)
- 따라서 **1D/2D 슬라이스 기법**으로 단면을 관찰합니다.
  - 1D: $L(W + aW_1)$
  - 2D: $L(W + aW_1 + bW_2)$
- Loss는 **piecewise-linear 구조**를 가지며, 각 데이터 샘플의 손실이 0 기준으로 선형 합성되는 형태입니다.

---

## Optimization 기법
1. **Random Search**
  - 여러 $W$를 무작위로 시도한 후, 가장 좋은 것을 선택합니다.
  - CIFAR-10 기준: 약 15.5% 정확도 → 거의 랜덤 추측 수준입니다.

2. **Random Local Search**
  - 무작위 $W$에서 시작하여 작은 perturbation(교란) $\delta W$로 이동합니다.
  - 만약
    $$
    L(W + \delta W) < L(W)
    $$
    이라면 업데이트합니다.
    $$
    W \leftarrow W + \delta W
    $$

3. **Gradient 기반 방법**
  - 무작위가 아닌, **기울기(gradient)** 방향으로 이동합니다.
  - 경사하강법(Gradient Descent)의 핵심 아이디어는 다음과 같습니다.
    - 1D 함수:  
      $$
      f'(x) = \frac{df}{dx}
      $$
    - 다변수 함수:  
      $$
      \nabla f(x) =
      \begin{bmatrix}
      \frac{\partial f}{\partial x_1},
      \frac{\partial f}{\partial x_2},
      \dots,
      \frac{\partial f}{\partial x_n}
      \end{bmatrix}
      $$

---

## Gradient 계산법
1. **Numerical Gradient**
  - 유한 차분(Finite Difference)을 이용합니다.
  - 근사식은 다음과 같습니다.
    $$
    \frac{\partial f}{\partial x_i} \approx
    \frac{f(x+h) - f(x-h)}{2h}
    $$
  - 구현은 간단하지만 느리고 근사치입니다.

2. **Analytical Gradient**
  - 수식을 미분하여 직접 계산합니다.
  - 예: **SVM Loss Gradient**
    - 정답 클래스 $y_i$에 대해:  
      $$
      \frac{\partial L_i}{\partial w_{y_i}} = - \sum_{j \neq y_i} \mathbf{1}[s_j - s_{y_i} + \Delta > 0] \cdot x_i
      $$
    - 다른 클래스 $j \neq y_i$:  
      $$
      \frac{\partial L_i}{\partial w_j} = \mathbf{1}[s_j - s_{y_i} + \Delta > 0] \cdot x_i
      $$

---

## Gradient Descent
- **Vanilla GD**: 전체 데이터셋 기준으로 기울기를 계산한 후 업데이트합니다.
  $$
  W \leftarrow W - \eta \nabla L(W)
  $$
  - $\eta$: 학습률(Learning Rate)

- **Mini-batch GD**: 일부 데이터(batch)로 근사하여 연산 효율을 높입니다.

- **SGD (Stochastic GD)**: 배치 크기가 1인 경우를 의미합니다.  
  실제로는 Mini-batch GD도 흔히 SGD라고 부릅니다.

---

## Gradient Descent 개선 기법
1. **Momentum**
  - 관성 효과를 추가합니다.
  - $$
    v_t = \rho v_{t-1} - \eta \nabla L(W)
    $$
  - $$
    W \leftarrow W + v_t
    $$
  - $$\rho$$: 모멘텀 계수 (보통 0.9 ~ 0.99)

2. **Nesterov Accelerated Momentum (NAG)**
  - 업데이트 전 미리 "앞을 내다보고" 보정합니다.
  - $$
    v_t = \rho v_{t-1} - \eta \nabla L(W_t + \rho v_{t-1})
    $$
  - $$ W_{t+1} = W_t + v_t $$

3. **AdaGrad**
  - 파라미터별 학습률을 조정합니다.
  - $$
    g_t = \nabla L(W_t)
    $$
  - $$
    W_{t+1} = W_t - \frac{\eta}{\sqrt{\sum_{\tau=1}^t g_\tau^2} + \epsilon} g_t
    $$

4. **RMSProp**
  - AdaGrad의 문제(학습률이 너무 빨리 줄어듦)를 개선한 기법입니다.
  - $$
    E[g^2]_t = \rho E[g^2]_{t-1} + (1-\rho) g_t^2
    $$
  - $$
    W_{t+1} = W_t - \frac{\eta}{\sqrt{E[g^2]_t} + \epsilon} g_t
    $$

5. **Adam (Adaptive Moment Estimation)**
  - Momentum과 RMSProp을 결합한 기법입니다.
  - $$
    m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t
    $$
  - $$
    v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2
    $$
    - **Bias Correction**:
    - $$
      \hat{m}_t = \frac{m_t}{1-\beta_1^t}, \quad
      \hat{v}_t = \frac{v_t}{1-\beta_2^t}
      $$
    - 최종 업데이트:
    - $$
      W_{t+1} = W_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t
      $$

---

## 맺음말
> **Loss Function의 개념과 수식, Gradient 기반 최적화 원리, 그리고 다양한 개선 기법**을 다루었습니다.  
> 경사하강법(Gradient Descent)에는 Momentum, AdaGrad, RMSProp, Adam이 있습니다.

> Loss Function과 Optimization은 딥러닝 학습의 기본입니다. 도움이 되었기를 바랍니다.

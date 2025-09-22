---
title: Deep Learning Networks
date: 2025-09-03 19:49:58 +09:00
categories: ['deep learning']
tags: ['deep learning', 'neural networks', 'activation function']
math: true
---

-   **Linear Classification**은 입력 데이터와 weight의 곱으로 점수를
    계산하는 방식입니다. 공식은 s = Wx 입니다.
-   예를 들어 CIFAR-10 이미지 분류 문제에서는 입력 x 가 \[3072 ×
    1\] 크기의 벡터이고, weight W 는 \[10 × 3072\] 크기를 가지며,
    출력 s 는 10개의 class score로 나타납니다.
-   **Simple Neural Network**는 다음과 같은 계산을 수행합니다: <br>
    $$ s = W_2 \times \max(0, W_1x) $$
  -   여기서 W1은 입력을 100차원 벡터로 변환하고, W2는
      10개의 class score를 출력합니다.
  -   학습은 Stochastic Gradient Descent (SGD)로 진행하며,
      **Backpropagation**을 통해 기울기를 계산합니다.
-   Non-linearity (ReLU)는 음수를 0으로 잘라내어 모델이 단순한 선형
    결합으로 환원되지 않도록 합니다.
-   Three-Layer Neural Network: <br/>
    $$ s = W_3 \times \max(0, W_2 \times \max(0, W_1x)) $$로 표현됩니다. <br/>
    Hidden layer의 크기는 hyperparameter로 조정할 수 있습니다.

------------------------------------------------------------------------

## Modeling One Neuron

-   **Biological Motivation**: 실제 뇌의 뉴런을 단순화하여 수학적 모델로
    나타낸 것이 Neural Network의 기본 아이디어입니다. 인간의 뇌에는 약
    860억 개의 뉴런이 있으며, $$10^{14} \sim 10^{15}$$개의 시냅스로
    연결되어 있습니다.
-   생물학적 뉴런은 **Dendrite**를 통해 입력을 받고, **Axon**을 통해
    출력을 내보내며, **Synapse**를 통해 다른 뉴런과 연결됩니다.
-   뉴런은 입력 신호의 합이 일정 threshold를 넘으면 발화(spike)를
    발생시킵니다. 이를 수학적으로는 입력 xi와 weight wi의
    곱을 모두 더한 값에 bias를 더한 후 **Activation Function**을
    적용하여 표현합니다.
-   weight는 시냅스 강도를 의미하며, 양수이면 흥분성(excitatory),
    음수이면 억제성(inhibitory) 영향을 줍니다.

### Activation Functions

1.  **Sigmoid**
  -   정의: $$ \sigma(x) = \frac{1}{1 + e^{-x}} $$
  -   출력 범위를 \[0,1\]로 squash하여 뉴런의 firing rate를 해석하기
      좋습니다.
  -   단점으로는 gradient vanishing 문제가 있으며, 특히 0 또는 1
      근처에서 gradient가 0에 가까워 학습이 멈출 수 있습니다. 또한
      출력이 zero-centered가 아니어서 weight 업데이트가 지그재그
      형태로 비효율적으로 진행될 수 있습니다.
2.  **Tanh**
  -   정의: $$ \tanh(x) $$
  -   출력 범위는 \[-1,1\]이며, zero-centered 특성을 가집니다.
  -   Sigmoid보다 학습이 안정적이지만 여전히 saturation 문제가
      존재합니다. 또한 $$ \tanh(x) = 2\sigma(2x) - 1 $$로 Sigmoid와
      밀접한 관계를 가집니다.
3.  **ReLU (Rectified Linear Unit)**
  -   정의: $$ f(x) = \max(0, x) $$
  -   계산이 단순하고, gradient가 포화되지 않아 학습 속도가 빠릅니다.
  -   하지만 **Dying ReLU** 문제가 발생할 수 있으며, 학습 도중 특정
      뉴런이 항상 0을 출력하게 되어 gradient가 영구적으로 0이 될 수
      있습니다. 적절한 learning rate 설정으로 이를 완화할 수 있습니다.
4.  **Leaky ReLU**
  -   정의: $$ f(x) =
      \begin{cases}
      \alpha x & x < 0 \\
      x & x \geq 0
      \end{cases} $$
  -   여기서 alpha 는 보통 0.01입니다.
  -   음수 입력에도 작은 gradient를 남겨 dying ReLU 문제를
      완화합니다.
  -   변형으로는 alpha를 학습 가능한 파라미터로 두는
      **PReLU**가 있습니다.
5.  **Maxout**
  -   정의: $$ f(x) = \max(w_1^Tx+b_1, w_2^Tx+b_2) $$
  -   ReLU와 Leaky ReLU를 일반화한 형태로, saturation과 dying 문제를
      모두 피할 수 있습니다.
  -   단점은 뉴런마다 파라미터 수가 두 배로 증가한다는 점입니다.

------------------------------------------------------------------------

## Neural Network Architectures

-   Neural Network는 일반적으로 acyclic graph로 표현되며, layer 단위로
    구성됩니다.
-   **Fully-connected layer**가 가장 기본적인 형태이며, 각 layer의
    출력이 다음 layer의 입력으로 전달됩니다.
-   **Single-layer Neural Network**는 hidden layer가 없는 구조로,
    Logistic Regression이나 SVM과 동일한 역할을 합니다.
-   **Output Layer**는 activation function을 사용하지 않고, 분류
    문제에서는 class score를, 회귀 문제에서는 실수 값을 출력합니다.

### Network Sizing

-   네트워크의 크기는 뉴런 개수와 파라미터 수로 측정됩니다.
-   예시: 6개의 뉴런과 20개의 weight, 6개의 bias → 총 26개의 학습
    파라미터.
-   현대 신경망은 약 1억 개의 파라미터를 가지며, 10\~20개의 layer를
    갖습니다.

### Representational Power

-   **Universal Approximation Theorem**에 따르면, 하나의 hidden layer만
    있어도 임의의 연속 함수를 원하는 정확도로 근사할 수 있습니다.
-   그러나 깊이가 깊어질수록 데이터의 계층적 구조(예: edge → shape →
    object)를 잘 학습할 수 있어 성능이 향상됩니다.
-   특히 이미지 인식에서는 깊은 **Convolutional Network**가 효과적이며,
    10개 이상의 layer가 필요합니다.

### Capacity vs. Complexity

-   큰 네트워크는 더 복잡한 함수를 학습할 수 있는 capacity를 가지지만,
    그만큼 overfitting 위험도 커집니다.
-   작은 네트워크는 overfitting 위험은 적지만, 학습이 잘 되지 않고 local
    minima에 갇힐 수 있습니다.
-   따라서 **Regularization** 기법을 활용하여 overfitting을 제어하는
    것이 중요합니다.
  -   예: **L2 regularization**, **Dropout**, **Input Noise**.
-   결론적으로는 가능한 한 큰 네트워크를 사용하고, 정규화 기법으로
    일반화 성능을 확보하는 것이 권장됩니다.

------------------------------------------------------------------------

## 맺음말

Neural Networks는 단순한 선형 모델을 확장하여 **Non-linearity**와
**Multi-layer 구조**를 추가함으로써 매우 복잡한 함수도 근사할 수 있습니다.

> 적절한 **Activation Function** 선택, **Network Depth** 설정, 그리고
> **Regularization** 기법의 조합이 모델의 성능과 안정성을 결정합니다.

> Neural Networks를 설계할 때에는 이 세 가지 요소를 신중히 고려해야
하며, 이를 통해 효율적이고 일반화 능력이 뛰어난 모델을 구축할 수
있습니다.

---
title: Deep Learning Training Neural Networks
date: 2025-09-30 12:52:58 +09:00
categories: ['deep learning']
tags: ['deep learning', 'training']
math: true
---

시각적 인공지능(Visual AI)을 위한 신경망 학습 과정에서 고려해야 할 활성화 함수, 초기화, 정규화, 하이퍼파라미터 튜닝 및 전이 학습에 대한 상세 내용을 다룹니다.

## 활성화 함수 (Activation Functions)

뉴런의 활성화 여부를 결정하는 비선형 함수들의 특징과 문제점, 그리고 권장 사항입니다.

### Sigmoid
* 수식: $$\sigma(x) = 1/(1+e^{-x})$$
* 특징: 입력을 $$[0, 1]$$ 사이의 값으로 압축(Squash)하며, 뉴런의 발화율(Firing rate)로 해석하기 좋습니다.
* 치명적 단점:
  1.  기울기 소실 (Gradient Killing): 입력값이 아주 크거나 작으면 기울기가 0에 수렴하여, 역전파 시 하위 레이어로 그래디언트가 전달되지 않습니다.
  2.  Not Zero-centered: 출력이 항상 양수이므로, 가중치 업데이트 시 그래디언트의 부호가 모두 같아져(전부 양수거나 음수) 지그재그(Zig-zag) 형태로 비효율적인 학습이 일어납니다.
  3.  연산 비용: `exp()` 함수는 계산 비용이 비쌉니다.

### Tanh (Hyperbolic Tangent)
* 특징: 입력을 $$[-1, 1]$$로 압축하며, 0 중심(Zero-centered) 출력을 가집니다.
* 단점: 여전히 양 끝단에서 기울기 소실 문제가 발생합니다.

### 1.3 ReLU (Rectified Linear Unit)
* 수식: $$f(x) = max(0, x)$$
* 장점: 양수 영역에서 포화(Saturate)되지 않아 수렴 속도가 Sigmoid/Tanh보다 훨씬 빠릅니다(약 6배). 연산이 매우 효율적입니다.
* 단점: 출력이 0 중심이 아닙니다. 또한, 입력이 음수일 경우 기울기가 0이 되어 해당 뉴런이 영구적으로 비활성화되는 'Dead ReLU' 현상이 발생할 수 있습니다.



### ReLU의 다른 버전
* Leaky ReLU: $$f(x) = max(0.01x, x)$$. 음수 영역에 작은 기울기를 주어 Dead ReLU 문제를 해결합니다.
* PReLU (Parametric ReLU): 음수 영역의 기울기 $$\alpha$$를 파라미터로 학습합니다.
* ELU (Exponential Linear Units): 음수 영역에서 $$\alpha(e^x-1)$$을 사용하여 0 평균에 가까운 출력을 내고 노이즈에 강인하지만, `exp()` 연산이 필요합니다.
* SELU (Scaled ELU): 깊은 네트워크에서도 별도의 배치 정규화(Batch Norm) 없이 자기 정규화(Self-normalizing)되는 특성이 있습니다.
* Maxout: $$max(w_1^Tx+b_1, w_2^Tx+b_2)$$ 형태로 ReLU와 Leaky ReLU를 일반화한 것이지만, 파라미터 수가 2배로 늘어납니다.

💡 먼저 ReLU를 사용하십시오. 성능 향상을 위해 Leaky ReLU, ELU 등을 시도해 볼 수 있으며, Sigmoid는 사용하지 않는 것이 좋습니다.

## 데이터 전처리 (Data Preprocessing)

데이터를 신경망에 입력하기 좋은 형태로 가공하는 과정입니다.

* Zero-centering (평균 제거): 데이터의 중심을 0으로 맞춥니다. ($$X -= mean(X)$$).
* Normalization (정규화): 데이터의 스케일을 동일하게 맞춥니다. ($$X /= std(X)$$). 이를 통해 손실 함수가 덜 민감해져 최적화가 쉬워집니다.
* PCA / Whitening: 데이터의 상관관계를 제거(Decorrelate)하고 스케일을 맞추는 기법이지만, 이미지 처리에서는 잘 사용되지 않습니다.
* 이미지 전처리 실무: 이미지에서는 보통 Zero-centering만 수행합니다. 전체 평균 이미지를 빼거나(AlexNet), 채널별 평균을 빼는 방식(VGGNet)을 사용합니다.

## 가중치 초기화 (Weight Initialization)

초기 가중치 설정은 그래디언트 흐름과 학습 수렴에 결정적인 역할을 합니다.

### 잘못된 초기화의 문제점
* 가중치가 너무 작을 때 (Small random): 깊은 네트워크를 통과할수록 활성값(Activation)이 0으로 수렴하고, 이에 따라 그래디언트도 0이 되어 학습이 되지 않습니다.
* 가중치가 너무 클 때 (Large std): 활성값이 -1 또는 1로 포화(Saturate)되어 그래디언트가 0이 되고 학습이 멈춥니다.

### Xavier (Glorot) 초기화
* 개념: 입출력 뉴런의 분산(Variance)을 일정하게 유지하려는 아이디어입니다.
* 수식: 표준편차를 $$1/\sqrt{Din}$$ 으로 설정합니다.
* 적용: Sigmoid나 Tanh를 사용할 때 효과적입니다. 하지만 ReLU 사용 시 활성값이 0으로 수렴하여 학습이 잘 되지 않는 문제가 있습니다.

### He (Kaiming) 초기화
* 개념: ReLU와 같이 절반의 입력이 0이 되는 비선형 함수를 고려하여 분산을 보정한 방식입니다.
* 수식: 표준편차를 $$\sqrt{2/Din}$$ 으로 설정합니다.
* 결과: ReLU 사용 시에도 깊은 레이어까지 활성값의 분포가 고르게 유지됩니다.

## 정규화 (Regularization)

모델이 학습 데이터에 과적합(Overfitting)되는 것을 막고 일반화 성능을 높이는 기법들입니다.

### 기본 정규화 기법
* L2 Regularization (Weight Decay): 가중치 제곱의 합을 손실 함수에 더해 큰 가중치에 페널티를 부여합니다.
* L1 Regularization: 가중치 절대값의 합을 더하여 희소한(Sparse) 가중치를 유도합니다.

### 드롭아웃 (Dropout)
* 원리: 학습(Forward pass) 중에 임의의 뉴런을 확률 $$p$$(보통 0.5)로 0으로 만듭니다.
* 효과: 특징들 간의 상호 의존(Co-adaptation)을 막고, 거대한 앙상블 모델을 학습하는 효과를 냅니다.
* Test Time: 테스트 시에는 모든 뉴런을 사용하되, 학습 때 켜져 있던 비율만큼 출력을 스케일링(Scaling)해줘야 합니다. (Inverted Dropout을 사용하면 학습 시에 미리 스케일링하여 테스트 시 연산을 줄일 수 있습니다).

### 데이터 증강 (Data Augmentation)
* 개념: 학습 이미지를 변형하여 데이터의 다양성을 늘립니다.
* 기법: 좌우 반전(Horizontal Flips), 무작위 크롭(Random Crops/Scales), 색상 변형(Color Jitter), 렌즈 왜곡 등.
* ResNet 예시: 학습 시 $$[256, 480]$$ 범위에서 크기를 조절하고 224x224 패치를 무작위로 자릅니다. 테스트 시에는 여러 스케일과 코너/중앙 크롭을 평균 내어 사용합니다.

### 기타 정규화 기법
* DropConnect: 뉴런 대신 연결 웨이트를 임의로 끊습니다.
* Fractional Max Pooling: 풀링 영역을 무작위로 설정합니다.
* Stochastic Depth: 학습 시 일부 레이어를 통째로 건너뜁니다(Skip).
* Cutout: 이미지의 특정 영역을 무작위로 가립니다.
* Mixup: 두 이미지를 임의의 비율로 섞어 학습합니다(예: 40% 고양이 + 60% 강아지).

## 하이퍼파라미터 튜닝 (Hyperparameter Tuning)

효율적인 모델 학습을 위한 단계별 접근법입니다.

1.  초기 손실 확인: 가중치 감쇠(Weight decay)를 끄고 초기 손실값이 예상치(예: $$log(Class 수)$$)와 맞는지 확인합니다.
2.  소규모 데이터 과적합: 아주 적은 데이터로 학습하여 손실을 0으로 만들 수 있는지(모델 용량 확인) 테스트합니다.
3.  학습률(LR) 탐색: 손실이 줄어드는 적절한 학습률을 찾습니다. (추천: 1e-1 ~ 1e-4).
4.  Coarse Grid 탐색: 넓은 범위의 LR과 Weight decay로 1~5 에폭 정도 짧게 학습합니다.
5.  Refine Grid 탐색: 좋은 성능을 보인 구간을 좁혀 더 길게 학습합니다.
6.  곡선 분석: 학습(Train)과 검증(Val) 정확도의 차이를 분석합니다. 차이가 크면 과적합(정규화 필요), 차이가 없으면 과소적합(모델 크기 증대 필요)입니다.

## 전이 학습 (Transfer Learning)

ImageNet과 같은 대규모 데이터셋으로 사전 학습된(Pre-trained) 모델을 활용하는 강력한 방법입니다.

* 데이터가 적을 때 (Small Dataset): 사전 학습된 CNN의 가중치는 고정(Freeze)하고, 마지막 분류기(Linear Classifier) 층만 자신의 데이터에 맞게 재학습합니다.
* 데이터가 많을 때 (Bigger Dataset): 모델 전체 혹은 상위 레이어의 가중치를 미세 조정(Fine-tuning)합니다. 이때 학습률은 기존보다 낮게(예: 1/10) 설정하여 사전 학습된 특징이 망가지지 않도록 합니다.

## 맺음말

> 이러한 기법들을 적절히 조합하여 적용한다면, 다양한 시각적 인공지능 문제에서 안정적이고 높은 성능을 내는 신경망 모델을 구축할 수 있습니다.

---
title: Image Classification Basic
date: 2025-08-13 16:47:20 +09:00
categories: ['deep learning']
tags: ['deep learning', 'image classification', 'loss function']
math: true
---


이미지 분류(Image Classification)는 컴퓨터 비전의 핵심 과제 중 하나입니다. 
단순히 이미지를 보고 무엇인지 맞히는 것처럼 보이지만, 컴퓨터 입장에서는 수십만 개의 픽셀 숫자를 해석해야 합니다.

1. **기본적인 이미지 분류 개념과 kNN → 5-NN 확장**
2. **선형 분류기(Linear Classifier)와 Bias Trick, 전처리 기법**
3. **SVM과 Softmax 손실 함수 비교**

를 중심으로 이미지 분류의 기초부터 심화 내용을 정리합니다.

---

## 1. Image Classification

### 1.1 정의

* 입력 이미지를 사전에 정의된 카테고리 중 하나로 분류.
* **예시:** {cat, dog, plane, truck, …}
* 컴퓨터는 이미지를 \*\*3차원 배열(Width × Height × RGB)\*\*로 인식.

---

### 1.2 주요 도전 과제

* **시점 변화(Viewpoint variation)**: 다른 각도에서 찍힘
* **크기 변화(Scale variation)**: 객체 크기 차이
* **변형(Deformation)**: 비강체 물체의 다양한 형태
* **가림(Occlusion)**: 일부만 보이는 경우
* **조명 변화(Illumination)**: 빛의 강도와 방향에 따라 픽셀 값이 크게 변함
* **배경 잡음(Background clutter)**: 배경과 객체의 구분 어려움
* **클래스 내 다양성(Intra-class variation)**: 같은 클래스라도 형태/색 다양

---

### 1.3 데이터 기반 접근

* 직접 규칙을 작성하기보다 **라벨링된 데이터셋** 기반 학습.
* 파이프라인: **입력 → 학습(Training) → 평가(Evaluation)**.

---

### 1.4 최근접 이웃 분류기 (kNN → 5-NN)

* **kNN 기본 (k=1):** 모든 학습 데이터를 저장, 입력과 가장 가까운 이미지의 라벨을 예측.
* **한계:** 1-NN은 노이즈나 특이값에 매우 취약.
* **5-NN 확장:**

  * 가장 가까운 5개 이웃을 찾아 **다수결 투표**로 예측.
  * 장점: 평활화(smoothing) 효과 → 이상치(outlier) 영향을 줄임.
  * 단점: 여전히 고차원 데이터에서 차원의 저주 문제 발생.

---

## 2. Linear Classification

### 2.1 점수 함수 (Score Function)

* kNN의 비효율성을 극복하기 위해 **파라메트릭 접근**을 사용.
* 점수 함수 정의:

$$
f(x_i, W, b) = W x_i + b
$$

여기서,

* $x_i$: 입력 벡터

* $W$: 가중치 행렬

* $b$: 편향 벡터

* 한 번의 행렬 연산으로 모든 클래스 점수 계산 가능.

---

### 2.2 Bias Trick

* 기존 식:

  $$
  f(x_i, W, b) = W x_i + b
  $$
* Bias Trick 적용: $x_i$에 항상 1을 추가 → $W$의 마지막 열이 bias 역할.
* 새로운 식:

  $$
  f(x_i, W) = W x_i
  $$
* 장점: 수식과 구현이 단순화.

---

### 2.3 데이터 전처리 (Preprocessing)

* 원본 픽셀 범위: \[0,255]. 그대로 쓰면 학습이 불안정.
* 주요 방법:

  1. **Zero-mean 정규화:** 평균 이미지를 빼서 데이터를 원점 근처로 이동.
  2. **스케일링:** \[-1,1] 범위로 맞춤 → 특징 크기 균일화.
* 효과: 경사하강법 안정성과 수렴 속도 향상.

---

## 3. 손실 함수 (Loss Function)

손실 함수는 모델이 얼마나 잘 분류하는지를 평가하며, 가중치 업데이트 방향을 결정합니다.

---

### 3.1 Multiclass SVM Loss (Hinge Loss)

#### 정의

$$
L_i = \sum_{j \neq y_i} \max(0, f(x_i; W)_j - f(x_i; W)_{y_i} + \Delta)
$$

* $f(x_i; W)_j = w_j^T x_i$: 클래스 $j$의 점수
* $\Delta$: 마진 (일반적으로 1)

#### 정규화 추가

* 가중치가 과도하게 커지는 것을 방지하기 위해 L2 정규화 추가:

$$
R(W) = \sum_k \sum_l W_{k,l}^2
$$

* 최종 SVM Loss:

$$
L = \frac{1}{N} \sum_i L_i + \lambda R(W)
$$

* $N$: 샘플 개수
* $\lambda$: 데이터 손실 vs. 정규화 손실 균형

#### Binary SVM과의 관계

$$
L_i = C \max(0, 1 - y_i w^T x_i) + R(W), \quad y_i \in \{-1,1\}
$$

* $C \propto \frac{1}{\lambda}$.

---

### 3.2 Softmax + Cross-Entropy Loss

#### 확률 해석

* 점수 함수는 동일:

  $$
  f(x_i; W) = W x_i
  $$
* Softmax를 통해 확률 변환:

  $$
  p_j = \frac{e^{f_j}}{\sum_k e^{f_k}}
  $$

#### 손실 함수

* 올바른 클래스 $y_i$에 대한 Cross-Entropy Loss:

$$
L_i = -\log \frac{e^{f_{y_i}}}{\sum_j e^{f_j}}
$$

* 올바른 클래스의 확률이 높아질수록 손실이 줄어듦.

---

### 3.3 SVM vs. Softmax 비교

| 구분     | SVM (Hinge Loss)     | Softmax (Cross-Entropy) |
| ------ | -------------------- | ----------------------- |
| 출력     | 점수 (uncalibrated)    | 확률 (normalized)         |
| 목표     | 올바른 클래스 점수가 Δ 이상 크도록 | 올바른 클래스 확률 최대화          |
| 최적화 성향 | 마진 만족 시 추가 최적화 없음    | 항상 더 높은 확률 분포를 만들려 함    |
| 해석 용이성 | 점수 크기 비교만 가능         | 확률로 직관적 해석 가능           |
| 비유     | “60점 넘으면 합격”         | “100점에 가까울수록 더 좋다”      |

---

## 맺음말

> 이미지 분류는 단순히 "이미지를 보고 맞히는 문제"가 아니라, **데이터 기반 학습, 모델 설계, 손실 함수 선택**이 어우러진 복합적인 문제입니다.

> kNN은 직관적이지만 확장성에 한계가 있고,
> 선형 분류기는 빠르고 효율적이지만 복잡한 패턴을 다루기에는 제한적입니다.
> 손실 함수에서는 SVM이 **마진 확보**라는 단순한 기준을, Softmax가 **확률 기반 해석**을 제공하며 딥러닝의 기반이 됩니다.

---


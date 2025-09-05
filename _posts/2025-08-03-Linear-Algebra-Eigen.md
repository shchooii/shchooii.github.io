---
title: Linear Algebra Eigen
date: 2025-08-23 16:45:20 +09:00
categories: ['linear algebra']
tags: ['linear algebra', 'eigen vector', 'eigen value']
math: true
---

행렬을 벡터에 곱하면 보통 방향이 바뀝니다.
하지만 **특별한 벡터**들은 방향이 그대로 유지되면서, **길이만 일정 비율로 변합니다.**
이 벡터를 **고유벡터(eigenvector)**, 그 비율을 **고유값(eigenvalue)** 라고 합니다.

---

## 1. 기본 정의

$$
Ax = \lambda x
$$

* $$A$$: $$n \times n$$ 정방행렬
* $$x$$: eigenvector ($$x \neq 0$$)
* $$\lambda$$: eigenvalue

즉, $$Ax$$가 $$x$$와 같은 직선 위에 있을 때.

---

## 2. 고유값 구하는 방법

$$
(A - \lambda I)x = 0
$$

→ $A - \lambda I$가 singular해야 하므로,

$$
\det(A - \lambda I) = 0
$$

을 풀면 eigenvalue $$\lambda$$를 얻을 수 있습니다.
이를 **특성방정식(characteristic equation)** 이라 합니다.

---

## 3. 중요한 성질

1. $$n \times n$$ 행렬은 보통 $$n$$개의 eigenvalue를 가짐 (중복 허용).

2. Eigenvalue의 합 = 행렬의 trace

   $$
   \sum \lambda_i = \text{trace}(A)
   $$

3. Eigenvalue의 곱 = 행렬의 determinant

   $$
   \prod \lambda_i = \det(A)
   $$

4. 삼각행렬(triangular)의 eigenvalue = 대각선 원소.

5. 대칭행렬(symmetric)의 eigenvalue는 항상 실수.

6. Singular 행렬은 반드시 $$\lambda=0$$ 포함.

---

## 4. 예시

* **Projection**: eigenvalue = 0, 1
* **Reflection**: eigenvalue = 1, -1
* **Rotation (90°)**: eigenvalue = $$i, -i$$ (복소수)

---

## 5. "방향은 그대로, 크기만 변한다"의 의미

* 대부분의 벡터는 행렬을 곱하면 방향이 바뀝니다.
* 하지만 eigenvector는 방향이 그대로 → 행렬의 **본질적인 작용**을 보여줌.
* Eigenvalue는 그때 얼마나 **늘어나는지/줄어드는지/뒤집히는지**를 알려줍니다.

👉 **왜 중요한가?**

1. **복잡한 문제 단순화**
   2. 미분방정식: $$\frac{du}{dt} = Au$$의 해가 eigenvalue/eigenvector로 표현됨.
2. **데이터 분석**
   3. PCA에서 가장 큰 eigenvalue 방향 = 데이터가 가장 퍼져 있는 주축.
3. **시스템 해석**
   4. Eigenvalue 부호에 따라 시스템의 안정/불안정 판단 가능.
4. **응용**
   5. 구글 PageRank, 양자역학의 에너지 값 등도 모두 eigenvalue 문제.

---

## 맺음말

Eigenvalue와 Eigenvector는

* **행렬이 벡터 공간을 어떻게 바꾸는지**
* **그중 변하지 않는 “특별한 축”이 무엇인지**

를 알려주는 핵심 도구입니다.

> 방향은 그대로, 크기만 변하는 벡터를 찾는 것입니다.

---

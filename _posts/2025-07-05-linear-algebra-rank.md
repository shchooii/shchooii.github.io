---
title: Linear Algebra Rank
date: 2025-07-05 11:40:20 +09:00
categories: ['linear alegbra']
tags: ['linear alegbra', 'rank']
math: true
---

Gilbert Strang의 선형대수학 11강 강의 내용을 바탕으로, 정리한 글입니다. 
**행렬 공간**의 개념을 확장하고, **부분 공간**의 차원 계산, **랭크 1 행렬**의 의미, 그리고 **스몰 월드 그래프**를 다룹니다.

---

### Matrix as a Vector Space 📐

**행렬(matrix)도 벡터(vector)가 될 수 있다**는 아이디어에서 시작합니다. 
벡터 공간은 덧셈과 스칼라 곱셈 연산에 대해 '닫혀'있는 집합을 의미합니다. 
3x3 행렬의 집합 `M`을 생각해봅시다.

* **덧셈**: 두 3x3 행렬을 더해도 결과는 여전히 3x3 행렬입니다.
* **스칼라 곱셈**: 3x3 행렬에 어떤 상수(스칼라)를 곱해도 결과는 3x3 행렬입니다.

이 두 가지 조건을 만족하므로, **모든 3x3 행렬의 집합 `M`은 그 자체로 하나의 거대한 벡터 공간**이 됩니다. 이 공간의 차원(dimension)은 행렬의 원소 개수인 9입니다.

이 공간의 **기저(basis)**는 이 공간을 생성하는 9개의 선형 독립적인 벡터(행렬)들로 이루어집니다. 가장 간단한 표준 기저는 특정 위치에만 1을 갖고 나머지는 모두 0인 9개의 행렬입니다.

$$
\text{Basis for } M =
\left\{
\begin{array}{l}
\begin{pmatrix} 1 & 0 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 0 \end{pmatrix},\\[6pt]
\begin{pmatrix} 0 & 1 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 0 \end{pmatrix},\\[2pt]
\vdots\\[2pt]
\begin{pmatrix} 0 & 0 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 1 \end{pmatrix}
\end{array}
\right\}
$$


---

### Subspaces of Matrix Space

벡터 공간 `M` 안에는 더 작은 벡터 공간, 즉 **부분 공간(subspace)**이 존재합니다.

* **대칭 행렬 (Symmetric Matrices, S)**: 전치(transpose)해도 자기 자신과 같은 행렬 ($A^T = A$)들의 집합입니다. 대칭 행렬끼리 더하거나 스칼라를 곱해도 여전히 대칭 행렬이므로 부분 공간입니다. 3x3 대칭 행렬의 차원은 6입니다.
  $$
  \begin{pmatrix} a & b & c \\ b & d & e \\ c & e & f \end{pmatrix} \quad \rightarrow \quad \dim(S) = 6
  $$
* **상삼각 행렬 (Upper Triangular Matrices, U)**: 주대각선 아래 원소들이 모두 0인 행렬들의 집합입니다. 이 역시 부분 공간이며, 3x3의 경우 차원은 6입니다.
  $$
  \begin{pmatrix} a & b & c \\ 0 & d & e \\ 0 & 0 & f \end{pmatrix} \quad \rightarrow \quad \dim(U) = 6
  $$
* **대각 행렬 (Diagonal Matrices, D)**: 주대각선 이외의 모든 원소가 0인 행렬들의 집합입니다. 이는 **대칭 행렬과 상삼각 행렬의 교집합**($S \cap U$)에 해당하며, 차원은 3입니다.
  $$
  \begin{pmatrix} a & 0 & 0 \\ 0 & d & 0 \\ 0 & 0 & f \end{pmatrix} \quad \rightarrow \quad \dim(S \cap U) = 3
  $$

#### 부분 공간의 차원 공식
두 부분 공간의 합과 교집합의 차원 사이에는 중요한 관계식이 성립합니다.
$$\dim(S) + \dim(U) = \dim(S \cap U) + \dim(S + U)$$
위의 예시에 대입해보면,
$$6 + 6 = 3 + \dim(S + U) \quad \therefore \quad \dim(S + U) = 9$$
이는 대칭 행렬과 상삼각 행렬의 합으로 모든 3x3 행렬을 표현할 수 있음을 의미합니다 (`S + U = M`).

---

### The Building Blocks: Rank 1 Matrices 🧱

**랭크 1 행렬**은 모든 행렬을 구성하는 가장 기본적인 "벽돌"과 같습니다. 이는 하나의 열벡터와 하나의 행벡터의 곱으로 표현됩니다.
$$A = \mathbf{u}\mathbf{v}^T$$
예를 들어,
$$\begin{pmatrix} 1 \\ 2 \\ 3 \end{pmatrix} \begin{pmatrix} 1 & 4 & 5 \end{pmatrix} = \begin{pmatrix} 1 & 4 & 5 \\ 2 & 8 & 10 \\ 3 & 12 & 15 \end{pmatrix}$$
이 행렬의 모든 행은 (1, 4, 5)의 배수이고, 모든 열은 (1, 2, 3)의 배수입니다. 이것이 랭크가 1인 이유입니다.

**랭크가 R인 행렬은 R개의 랭크 1 행렬의 합으로 분해될 수 있습니다.** 
랭크 1 행렬들의 집합 자체는 부분 공간이 아닙니다. 두 랭크 1 행렬을 더했을 때 랭크가 2 이상이 될 수 있어, 덧셈에 대해 닫혀있지 않기 때문입니다.

---

### Small-World Graphs 🌐

강의 마지막 부분에서는 그래프 이론의 **스몰 월드 그래프** 개념을 소개합니다. 이는 세상이 얼마나 긴밀하게 연결되어 있는지를 보여주는 모델입니다.

* **그래프**: 노드(점)와 엣지(선)로 구성된 네트워크입니다.
* **스몰 월드**: "6단계 분리 법칙"처럼, 대부분의 노드들이 적은 수의 단계를 거쳐 서로 연결될 수 있는 네트워크를 의미합니다.

### 맺음말
> 행렬을 벡터 공간으로 분석하고, 스몰 월드 그래프 예시를 통해 선형대수학이 어떻게 현실 문제에 적용되는지 알아보았습니다.
> 예를 들어 그래프의 연결성은 adjacency matrix로 표현하고 분석할 수 있습니다.

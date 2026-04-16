---
title: Probability Markov Chain
date: 2026-03-03 18:20:00 +09:00
categories: ['probability']
tags: ['markov-chain', 'transition-matrix', 'chapman-kolmogorov', 'state-classification']
math: true
---


본 문서는 Markov Chain의 기본 개념부터 전이확률, n-step 확률, 상태 분류(Classification), 주기성(Periodicity)까지 핵심 내용을 정리했습니다.
Markov Chain은 확률적 상태 변화 과정을 모델링하는 데 사용되며, 현재 상태만으로 미래를 설명할 수 있는 구조를 가집니다.

## Markov Chain 정의

### 기본 정의

확률변수 열 $X_0, X_1, X_2, \dots$가 상태공간 $S = {1,2,\dots,m}$에서 값을 가지며 다음 조건을 만족하면 Markov Chain이라고 합니다.

$$
P(X_{n+1}=j \mid X_n=i) = P(X_{n+1}=j \mid X_n=i, X_{n-1}, \dots, X_0)
$$

현재 상태 $X_n$만 알면 미래 상태가 결정된다는 의미입니다.



### 직관적 의미

과거 정보는 현재 상태에 반영되어 있으며 미래는 현재 상태만으로 결정됩니다.

현재가 과거의 정보를 요약한 상태로 이해할 수 있습니다.



### Time-homogeneous Markov Chain

$$
p_{ij} = P(X_{n+1}=j \mid X_n=i)
$$

이 값이 시간 (n)에 의존하지 않으면 time-homogeneous라고 합니다.



## 상태(State)와 상태공간

상태(state)는 $X_n$이 가질 수 있는 값이며 상태공간(state space)은 가능한 상태들의 집합 $S$입니다.

예시로 기계 상태 {정상, 고장}, 날씨 상태 {맑음, 흐림, 비}와 같이 정의할 수 있습니다.



## 상태 선택의 중요성

COVID 예시에서 상태 정의에 따라 Markov Chain 여부가 달라집니다.

* $X_n$: 감염자 수
* $Y_n$: 비감염자 수

$X_n$만으로는 Markov Chain이 아니고 $Y_n$만으로도 아닙니다.
$(X_n, Y_n)$를 함께 상태로 정의하면 Markov Chain이 됩니다.

Markov Chain 성립 여부는 상태 정의에 의해 결정됩니다.



## Sample Path 확률

특정 경로의 확률은 다음과 같습니다.

$$
P(X_0=i_0, X_1=i_1, \dots, X_n=i_n)
$$

Markov property를 적용하면

$$
= P(X_0=i_0)\cdot p_{i_0 i_1}\cdot p_{i_1 i_2} \cdots p_{i_{n-1} i_n}
$$

초기 확률과 각 단계의 전이확률을 곱하여 계산합니다.



## n-step 전이확률

$$
r_{ij}(n) = P(X_n=j \mid X_0=i)
$$

n step 이후 상태 확률을 의미합니다.



### Chapman-Kolmogorov 식

$$
r_{ij}(n) = \sum_{k=1}^m r_{ik}(n-1),p_{kj}
$$

중간 상태 $k$를 거치는 모든 경우를 합산합니다.



### 일반화된 형태

$$
r_{ij}(n+l) = \sum_{k=1}^m r_{ik}(n),r_{kj}(l)
$$

## 전이확률행렬

전이확률행렬은

$$
P = [p_{ij}]
$$

로 정의됩니다.

n-step 전이확률행렬은

$$
P^{(n)} = [r_{ij}(n)]
$$

입니다.

결과는

$$
P^{(n)} = P^n
$$

입니다.

n-step 확률은 행렬을 n번 곱해서 계산합니다.



## Example: Urn 문제

### 설정

공 2개가 있으며 색은 빨강과 파랑입니다.

매 단계에서 공 하나를 선택하고 같은 색으로 바뀔 확률은 0.8, 다른 색으로 바뀔 확률은 0.2입니다.



### 상태 정의

$$
X_n = \text{빨간 공 개수}, \quad S={0,1,2}
$$



### 전이확률행렬

$$
P =
\begin{pmatrix}
0.8 & 0.2 & 0 \
0.1 & 0.8 & 0.1 \
0 & 0.2 & 0.8
\end{pmatrix}
$$



### 문제 해결

다섯 번째 공이 빨강일 확률은

$$
P(A) = \sum_{i=0}^2 P(A|X_4=i),P(X_4=i|X_0=2)
$$

$$
= 0 \cdot r_{2,0}(4) + 0.5 \cdot r_{2,1}(4) + 1 \cdot r_{2,2}(4)
$$

결과는

$$
P(A) = 0.7048
$$

입니다.



## 상태 분류 (Classification of States)

### Accessible

$$
i \to j
$$

어떤 시점에 $i$에서 $j$로 갈 수 있음을 의미합니다.



### Communication

$$
i \leftrightarrow j
$$

서로 왕복 가능함을 의미합니다.



### Class

서로 communicate하는 상태들의 집합입니다.

예시는 ${1,2}, {3}, {4,5,6}$입니다.



## Recurrent vs Transient

### Recurrent

떠난 후 다시 돌아올 수 있는 상태입니다.
충분한 시간이 지나면 다시 방문됩니다.



### Transient

떠난 후 다시 돌아오지 못할 수 있는 상태입니다.
일시적으로만 방문됩니다.



### 직관

Recurrent는 반복적으로 방문되는 상태이고 Transient는 잠시 거치는 상태입니다.



## Markov Chain 구조

Markov Chain은 하나 이상의 recurrent class와 경우에 따라 transient states로 구성됩니다.

특징은 다음과 같습니다.

* transient 상태에서 recurrent 상태로 이동 가능
* recurrent 상태에서 transient 상태로 이동 불가



### Irreducible

recurrent class가 하나만 존재하는 경우 irreducible이라고 합니다.

모든 상태가 하나의 연결된 구조를 이룹니다.



## Periodicity

### 정의

상태들이 (d)개의 그룹으로 나뉘고

$$
S_1 \to S_2 \to \dots \to S_d \to S_1
$$

형태를 가지면 period $d$입니다.



### 특징

$$
r_{ii}(n)=0 \quad (n \not\equiv 0 \mod d)
$$

특정 step에서만 상태로 돌아올 수 있습니다.



### Aperiodic

period가 1이면 aperiodic입니다.
복귀 시점에 제한이 없습니다.



### 중요한 성질

self-transition $(p_{ii} > 0)$이 존재하면 aperiodic입니다.



## 맺음말

> Markov Chain은 현재 상태만으로 미래를 설명하는 확률 모델입니다. 상태 정의에 따라 Markov property 성립 여부가 달라지므로 상태 설정이 중요합니다. 
> 전이확률행렬과 Chapman-Kolmogorov 식을 통해 장기적인 상태 확률을 계산할 수 있으며 상태 분류를 통해 구조를 이해할 수 있습니다.

> 이 내용은 이후 stationary distribution과 steady-state behavior 분석의 기초가 됩니다.

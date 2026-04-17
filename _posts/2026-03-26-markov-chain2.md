---
title: Probability Markov Chain 2
date: 2026-03-26 20:20:00 +09:00
categories: ['probability']
tags: ['markov-chain', 'steady-state', 'stationary distribution', 'birth-death process', 'absorbing state', 'first passage time', 'first recurrence time']
math: true
---


지금까지 살펴본 Markov chain의 long-term behavior과 transient behavior를 하나의 흐름으로 정리했습니다.  
앞부분에서는 steady-state behavior와 stationary distribution을 중심으로, 시간이 충분히 지났을 때 마코프체인이 어떤 상태에 얼마나 자주 머무는지를 정리합니다.
뒷부분에서는 여러 recurrent class와 transient state가 함께 존재하는 경우를 다루며, 결국 어느 recurrent class에 들어가는지, 그때까지 몇 단계가 걸리는지, 특정 상태에 처음 도달하거나 다시 돌아오기까지의 평균 시간이 무엇을 의미하는지 정리합니다.

## Steady-state behavior의 의미

마코프체인에서  
$$
r_{ij}^{(n)}
$$
는 상태 $$i$$에서 시작하여 $$n$$단계 후 상태 $$j$$에 있을 확률을 의미합니다.

우리가 관심을 두는 것은 $$n$$이 매우 커졌을 때 이 값이 어떤 한 값으로 안정되는지입니다.  

$$
r_{ij}^{(n)} \to \pi_j
\quad (n \to \infty)
$$
와 같이 수렴하는지를 봅니다.

이때 $$\pi_j$$는 장기적으로 상태 $$j$$에 있을 확률입니다.  
쉽게 말하면, 마코프체인을 아주 오래 관찰했을 때 전체 시간 중 상태 $$j$$에 머무는 비율이라고 이해할 수 있습니다.

예를 들어 상태가 두 개뿐인 시스템에서

- 상태 1: working
- 상태 2: broken

이라면,

$$
\pi_{\text{working}} = \alpha, \qquad
\pi_{\text{broken}} = \beta
$$

는 장기적으로 기계가 정상 상태와 고장 상태에 머무는 비율을 뜻합니다.

steady-state behavior는 시스템의 순간적인 모습이 아니라, 오래 보았을 때의 평균적인 모습을 설명해 줍니다.

## steady-state convergence가 일어나기 위한 조건

limiting distribution가 출발 상태와 무관하게 하나의 분포로 수렴하려면 중요한 조건이 필요합니다.

### 단 하나의 recurrent class

먼저 recurrent class가 하나뿐이어야 합니다.  
recurrent class는 한 번 들어가면 그 안의 상태들을 계속 방문하게 되는 상태들의 묶음입니다.

만약 recurrent class가 여러 개라면, 처음 어느 곳에서 시작했는지에 따라 나중에 들어가게 되는 class가 달라질 수 있습니다.  
그러면 장기적인 거동이 출발 상태에 의존하게 되므로 하나의 공통된 limiting distribution으로 정리하기 어렵습니다.

recurrent class가 여러 개이면

- 어떤 class에 들어갈지는 시작 상태에 따라 달라지고
- 한번 들어간 class 밖의 다른 recurrent class는 더 이상 방문하지 않게 됩니다.

### aperiodic해야 함

두 번째 조건은 recurrent class가 aperiodic해야 한다는 것입니다.

periodic한 경우에는 확률이 주기적으로 흔들리면서 하나의 값으로 수렴하지 않을 수 있습니다.  
가장 쉬운 예는 두 상태가 번갈아 가는 경우입니다.

예를 들어
$$
P =
\begin{pmatrix}
0 & 1 \\
1 & 0
\end{pmatrix}
$$
이면, 상태 1에서 시작했을 때

- 1단계 후에는 반드시 2
- 2단계 후에는 반드시 1
- 3단계 후에는 반드시 2

가 되어 확률이 한 값으로 수렴하지 않고 계속 진동합니다.

이런 경우에는 steady-state limit가 존재하지 않을 수 있습니다.  
반대로 aperiodic하면 이러한 강한 주기성이 없어서 장기적으로 안정된 분포에 수렴할 가능성이 생깁니다.

## steady-state probability $$\pi$$를 구하는 방법

limiting distribution이 존재한다고 가정하면, 이를 직접 극한으로 구하지 않고 balance equation과 normalization equation을 이용해 계산할 수 있습니다.

### Balance equation

장기확률 $$\pi_j$$는 다음 식을 만족합니다.

$$
\pi_j = \sum_{k=1}^{m} \pi_k p_{kj}
$$

이 식의 의미는 다음과 같습니다.

상태 $$j$$에 장기적으로 머무는 비율은, 다른 상태들에서 $$j$$로 들어오는 장기적인 흐름의 총합과 같아야 합니다.

steady-state에서는 어떤 상태로 들어오는 평균적인 비율과 그 상태에 머무는 평균적인 비율이 서로 균형을 이룹니다.

### Normalization equation

확률들의 전체 합은 1이어야 하므로

$$
\sum_{i=1}^{m} \pi_i = 1
$$

도 함께 만족해야 합니다.

따라서 실제 계산은

- balance equation들
- normalization equation

을 함께 풀어 $$\pi_1, \pi_2, \dots, \pi_m$$를 구하는 방식으로 진행합니다.

## Long-term frequency 해석

steady-state probability $$\pi_j$$는 단순한 기호가 아니라 장기적인 방문 빈도를 의미합니다.

$$
\pi_j
$$
는 마코프체인을 오랫동안 관찰했을 때 상태 $$j$$에 있는 시간의 비율입니다.

$$
\pi_j p_{jk}
$$
는 장기적으로 $$j \to k$$ 전이가 일어나는 비율을 뜻합니다.

그래서 balance equation은 결국

“상태 $$j$$로 들어오는 장기 전이 빈도의 합이 상태 $$j$$에 머무는 장기 비율과 일치한다”

는 뜻으로 해석할 수 있습니다.

## Stationary distribution의 의미

steady-state probability $$\{\pi_j\}$$는 stationary distribution이라고도 부릅니다.

그 이유는 처음 상태를 이미 $$\pi$$ 분포에 따라 선택하면, 한 단계 뒤에도 분포가 그대로 유지되기 때문입니다.

$$
P(X_0 = j) = \pi_j
$$
로 시작하면,
$$
P(X_1 = j)
= \sum_{k=1}^{m} P(X_0 = k)p_{kj}
= \sum_{k=1}^{m} \pi_k p_{kj}
= \pi_j
$$
가 됩니다.

따라서 한 단계 뒤에도 분포가 바뀌지 않고, 더 일반적으로 모든 $$n$$에 대해
$$
P(X_n = j) = \pi_j
$$
가 됩니다.

그래서 stationary distribution은 “시간이 지나도 형태가 변하지 않는 분포”라고 이해할 수 있습니다.

행렬식으로 쓰면
$$
\pi = \pi P
$$
입니다.

## 2상태 예제 해석

Transition matrix가
$$
P =
\begin{pmatrix}
0.8 & 0.2 \\
0.6 & 0.4
\end{pmatrix}
$$
라고 하겠습니다.

이때 balance equation은

$$
\pi_1 = 0.8\pi_1 + 0.6\pi_2
$$

$$
\pi_2 = 0.2\pi_1 + 0.4\pi_2
$$

이고, normalization equation은

$$
\pi_1 + \pi_2 = 1
$$

입니다.

이 식을 풀면
$$
\pi_1 = 0.75, \qquad \pi_2 = 0.25
$$
가 됩니다.

시간이 충분히 지나면 장기적으로

- 상태 1에 75%
- 상태 2에 25%

정도 머문다고 해석할 수 있습니다.

## 우산 예제

교수가 집과 사무실을 오가는데, 비가 오고 현재 있는 곳에 우산이 있으면 우산을 들고 갑니다.  
비가 오지 않으면 우산을 가져가지 않습니다.  
출근 또는 퇴근할 때 비가 올 확률은 매번 $$p$$이고, 서로 독립이라고 가정합니다.

이 문제를 Markov chain으로 모델링하려면 상태를 “현재 있는 위치에 우산이 몇 개 있는가”로 정의합니다.

상태는

- 0: 현재 있는 곳에 우산 0개
- 1: 현재 있는 곳에 우산 1개
- 2: 현재 있는 곳에 우산 2개

입니다.

이때 balance equation과 normalization equation을 세우면

$$
\pi_0 = (1-p)\pi_2
$$

$$
\pi_1 = (1-p)\pi_1 + p\pi_2
$$

$$
\pi_2 = \pi_0 + p\pi_1
$$

$$
\pi_0 + \pi_1 + \pi_2 = 1
$$

이 되고, 이를 풀면

$$
\pi_0 = \frac{1-p}{3-p}, \qquad
\pi_1 = \frac{1}{3-p}, \qquad
\pi_2 = \frac{1}{3-p}
$$

를 얻습니다.

이제 장기적으로 통근 한 번당 비를 맞을 확률을 구하려면

- 현재 있는 곳에 우산이 0개여야 하고
- 그 순간 비가 와야 합니다.

따라서 비를 맞을 장기 확률은

$$
p\pi_0
$$

입니다.

이 예제는 steady-state probability를 구하면 단지 상태 비율뿐 아니라, 장기적인 성능 지표도 계산할 수 있음을 보여줍니다.

## Birth-Death Process

birth-death process는 상태들이 일렬로 배열되어 있고, 한 번에 이웃한 상태로만 이동하는 Markov chain입니다.

상태 $$i$$에서 가능한 이동은 보통

- $$i \to i+1$$
- $$i \to i-1$$

또는 제자리뿐입니다.

여기서

$$
b_i = P(X_{n+1} = i+1 \mid X_n = i)
$$

$$
d_i = P(X_{n+1} = i-1 \mid X_n = i)
$$

를 각각 birth probability, death probability라고 합니다.

이 구조에서는 일반적인 balance equation보다 더 단순한 식이 나옵니다.  
바로 local balance equation입니다.

$$
\pi_i b_i = \pi_{i+1} d_{i+1}, \qquad i=0,1,\dots,m-1
$$

이 식은 장기적으로

- $$i \to i+1$$로 가는 흐름
- $$i+1 \to i$$로 오는 흐름

이 서로 같다는 뜻입니다.

따라서 이를 반복적으로 이용하면

$$
\pi_i
=
\pi_0
\frac{b_0 b_1 \cdots b_{i-1}}{d_1 d_2 \cdots d_i},
\qquad i=1,\dots,m
$$

를 얻을 수 있고, 마지막으로
$$
\sum_{i=0}^{m}\pi_i = 1
$$
을 이용하여 전체 steady-state probability를 구할 수 있습니다.

## 여러 recurrent class와 transient state가 있는 경우

이제부터는 Markov chain에

- 여러 개의 recurrent class
- transient state들의 집합 $$T$$

가 함께 존재하는 경우를 생각합니다.

이 경우 transient state에서 출발하면 transient state들은 유한 번만 방문되고, 결국 어느 하나의 recurrent class에 들어가게 됩니다.  
한번 어떤 recurrent class에 들어가면 그 class의 상태들은 무한히 방문되지만, 다른 recurrent class의 상태들은 더 이상 방문되지 않습니다.

이번에는 “장기적으로 어느 상태에 얼마나 오래 머무는가”보다 먼저

- 결국 어느 recurrent class에 들어가는가
- 거기에 들어가기까지 시간이 얼마나 걸리는가

를 보는 것이 핵심이 됩니다.

## Absorbing state와 absorbing Markov chain

분석을 쉽게 하기 위해 먼저 모든 recurrent state가 absorbing state인 경우를 생각합니다.

상태 $$k$$가 absorbing이라는 것은

$$
p_{kk} = 1, \qquad p_{kj}=0 \quad (j \neq k)
$$

를 의미합니다.

absorbing state는 한 번 들어가면 절대 빠져나올 수 없는 상태입니다.

이 경우 문제는

- 특정 absorbing state에 도달할 확률
- absorbing state에 도달하기까지 걸리는 기대시간

을 구하는 것으로 바뀝니다.

## 특정 absorbing state에 도달할 확률

고정된 absorbing state $$s$$에 대해

$$
a_i = a_i(s)
$$

를 상태 $$i$$에서 출발해서 결국 $$s$$에 도달할 확률이라고 정의합니다.

예를 들어 목표 상태가 $$s=6$$일 때,

- $$a_1$$: 1에서 출발해 결국 6에 도달할 확률
- $$a_6$$: 6에서 출발해 결국 6에 도달할 확률

입니다.

만약 1과 6이 absorbing state라면

$$
a_1 = 0, \qquad a_6 = 1
$$

입니다.

- 1은 다른 absorbing state이므로 거기 있으면 6에 갈 수 없고
- 6은 이미 목표 상태이기 때문입니다.

이제 transient state들에 대해서는 한 단계 먼저 이동한 뒤의 경우를 따져 식을 세웁니다.

예를 들어

$$
a_2 = 0.2a_1 + 0.3a_2 + 0.4a_3 + 0.1a_6
$$

$$
a_3 = 0.2a_2 + 0.8a_6
$$

와 같은 식이 주어졌다면, 이는 각각

- 상태 2에서 출발해서 한 걸음 뒤에 도달하는 상태별 성공확률의 평균
- 상태 3에서 출발해서 한 걸음 뒤에 도달하는 상태별 성공확률의 평균

을 뜻합니다.

도달확률 문제의 핵심은  
“한 번 이동한 뒤의 상태들에서 다시 같은 문제를 푼다”  
는 first-step analysis입니다.

위 식에 $$a_1=0$$, $$a_6=1$$을 대입하면

$$
a_2 = \frac{21}{31}, \qquad a_3 = \frac{29}{31}
$$

를 얻습니다.

이는 상태 3에서 시작하는 쪽이 상태 6에 더 가까운 구조이므로 6에 도달할 확률이 더 크다는 뜻입니다.

## 일반적인 recurrent class 문제를 absorbing 문제로 바꾸는 이유

일반 Markov chain에서는 recurrent class가 단일 상태가 아니라 여러 상태의 묶음일 수 있습니다.  
예를 들어 recurrent class가

$$
\{1\}, \qquad \{4,5\}
$$

라고 하겠습니다.

이때 “결국 recurrent class $$\{4,5\}$$에 들어갈 확률”을 알고 싶다면, class 내부에서 4와 5 사이를 어떻게 오가는지는 본질적으로 중요하지 않습니다.

왜냐하면 관심 있는 것은

- 4와 5 내부에서의 세부 이동이 아니라
- 결국 $$\{4,5\}$$라는 class에 들어가느냐

이기 때문입니다.

따라서 이런 문제는 recurrent class 전체를 하나의 absorbing target처럼 보고, absorbing chain 문제로 변환하여 다룰 수 있습니다.

## absorption time: recurrent state에 처음 들어가기까지의 기대시간

이제 확률이 아니라 시간을 생각합니다.

$$
\mu_i
$$
를 transient state $$i$$에서 출발하여 처음 recurrent state에 들어가기까지 걸리는 기대 전이 횟수라고 정의합니다.

이미 recurrent state에서 시작하면 목표를 이미 달성한 상태이므로

$$
\mu_i = 0
$$

입니다.

예를 들어 recurrent state가 1과 4라면

$$
\mu_1 = \mu_4 = 0
$$

입니다.

반면 transient state에서는 first-step analysis를 사용하여 식을 세웁니다.  
예를 들어

$$
\mu_2 = 1 + 0.4\mu_2 + 0.3\mu_3
$$

$$
\mu_3 = 1 + 0.3\mu_2 + 0.4\mu_3
$$

와 같이 나타낼 수 있습니다.

여기서 앞의 $$1$$은 한 번 이동하는 데 이미 1 step을 사용했다는 뜻입니다.  
그 이후에는 도착한 상태에서 남은 기대시간을 더합니다.

기대시간 문제의 기본 구조는

$$
\text{기대시간} = 1 + \text{다음 상태들에서의 기대시간의 가중평균}
$$

입니다.

일반 Markov chain에서도 마찬가지로 recurrent class를 absorbing처럼 바꾸면 같은 방식으로 흡수시간을 계산할 수 있습니다.

## mean first passage time

이제 특정 recurrent state $$s$$에 처음 도달하기까지 걸리는 기대시간을 생각합니다.

$$
t_i = t_i(s)
$$

를 상태 $$i$$에서 출발하여 상태 $$s$$에 처음 도달하기까지 걸리는 기대 전이 횟수라고 정의합니다.

예를 들어 $$s=1$$이라면

$$
t_1 = 0
$$

입니다. 이미 1에 있으므로 1에 도달하는 데 시간이 필요하지 않기 때문입니다.

상태 2에서 출발하는 경우, 예를 들어

$$
t_2 = 1 + p_{21}t_1 + p_{22}t_2
$$

와 같이 쓸 수 있습니다.

만약
$$
p_{21}=0.6, \qquad p_{22}=0.4
$$
라면

$$
t_2 = 1 + 0.6\cdot 0 + 0.4t_2 = 1 + 0.4t_2
$$

이고, 따라서

$$
t_2 = \frac{5}{3}
$$

을 얻습니다.

이 값은 상태 2에서 출발해서 상태 1에 처음 도달하기까지 평균적으로 $$5/3$$단계가 걸린다는 의미입니다.

## mean first recurrence time

특정 상태 $$s$$에서 출발해서 다시 $$s$$로 처음 돌아오기까지 걸리는 기대시간을  
mean first recurrence time이라고 합니다.

이를
$$
t_s^\star
$$
로 나타냅니다.

이는 $$t_s=0$$과는 다른 개념입니다.  
$$t_s$$는 이미 상태 $$s$$에 있으므로 0이지만,  
$$t_s^\star$$는 일단 한 번 떠난 뒤 다시 돌아오는 시간을 의미합니다.

예를 들어 상태 1에서 시작할 때

$$
t_1^\star = 1 + p_{11}t_1 + p_{12}t_2
$$

와 같이 쓸 수 있습니다.

만약
$$
p_{11}=0.8, \qquad p_{12}=0.2, \qquad t_2=\frac{5}{3}
$$
라면

$$
t_1^\star
= 1 + 0.8\cdot 0 + 0.2 \cdot \frac{5}{3}
= \frac{4}{3}
$$

입니다.

이는 상태 1에서 출발하여 다시 1로 돌아오기까지 평균적으로 $$4/3$$단계가 걸린다는 뜻입니다.

## 정리

지금까지 나온 식들은 서로 달라 보이지만, 대부분 같은 아이디어에서 나옵니다.

### 도달확률 문제
도달확률 $$a_i$$는
“한 걸음 뒤 도착할 수 있는 상태들에서의 도달확률의 가중평균”
으로 표현됩니다.

$$
a_i = \sum_j p_{ij} a_j
$$
형태를 가집니다.

### 기대시간 문제
흡수시간 $$\mu_i$$, first passage time $$t_i$$, first recurrence time $$t_s^\star$$는 모두
“한 번 이동하는 데 걸린 1 step”
과
“그 다음 상태에서의 남은 기대시간”
을 더하는 구조를 가집니다.

전형적으로
$$
\text{기대시간} = 1 + \sum_j p_{ij}(\text{다음 상태에서의 기대시간})
$$
형태입니다.

따라서 이 강의 내용은 서로 다른 공식을 외우기보다,  
first-step analysis라는 공통 원리를 이해하는 것이 훨씬 중요합니다.

## 맺음말

> 지금까지 Markov chain의 steady-state behavior와 stationary distribution, birth-death process, absorbing state, absorbing probability, absorption time, mean first passage time, mean first recurrence time을 하나의 흐름으로 정리하였습니다. 
> 앞부분에서는 마코프체인을 오래 관찰했을 때 각 상태에 머무는 비율이 어떻게 결정되는지를 보았고, 뒷부분에서는 transient state에서 출발해 결국 어떤 recurrent class에 들어가는지, 그리고 그 과정에 평균적으로 얼마나 시간이 걸리는지를 살펴보았습니다.
> 모든 문제가 제각기 다른 공식처럼 보이더라도, 대부분은 “한 번 먼저 이동한 뒤 남은 문제를 다시 쓴다”는 first-step analysis에서 출발한다는 점입니다.

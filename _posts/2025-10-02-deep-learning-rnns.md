---
title: Deep Learning Recurrent Neural Networks
date: 2025-10-02 13:52:58 +09:00
categories: ['deep learning']
tags: ['deep learning', 'RNNs']
math: true
---

시각 인공지능(Visual Artificial Intelligence) 강의 중 순환 신경망(RNN, Recurrent Neural Networks)에 대한 핵심 내용을 다루고 있습니다.
기존의 'Vanilla' 신경망이 고정된 크기의 입력과 출력을 처리하는 데 그쳤다면, RNN은 가변적인 길이를 가진 시퀀스(Sequence) 데이터를 처리하는 데 특화된 모델입니다.
RNN의 기본 구조, 다양한 처리 방식, 학습 알고리즘, 그리고 문자열 생성 및 이미지 캡셔닝과 같은 구체적인 응용 사례를 상세히 다룹니다.

## RNN의 기본 개념 및 구조
### 기존 신경망과의 차이점
* Vanilla Neural Network: 고정된 크기의 입력(Input)을 받아 고정된 크기의 출력(Output)을 내보내는 일대일(One-to-One) 구조로 제한됩니다.
* Recurrent Neural Network (RNN): 시퀀스 데이터를 처리하며, 다음과 같은 다양한 입력-출력 형태를 지원합니다.
  * One-to-Many: 하나의 입력(예: 이미지)에서 시퀀스 출력(예: 이미지 캡션)을 생성.
  * Many-to-One: 시퀀스 입력(예: 비디오 프레임, 문장)에서 하나의 출력(예: 행동 분류, 감정 분석)을 생성.
  * Many-to-Many: 시퀀스 입력에서 시퀀스 출력을 생성 (예: 기계 번역, 비디오 캡셔닝).

### RNN의 작동 원리 (Process Sequences)
RNN의 핵심 아이디어는 시퀀스를 처리할 때 업데이트되는 '내부 상태(Internal State)'를 가진다는 것입니다.
* 재귀 수식 (Recurrence Formula): 매 시간 단계(time step)마다 동일한 함수와 파라미터를 사용하여 상태를 업데이트합니다.
  $$h_t = f_W(h_{t-1}, x_t)$$
  * $$h_t$$: 현재의 새로운 상태 (new state)
  * $$h_{t-1}$$: 이전 상태 (old state)
  * $$x_t$$: 현재 시점의 입력 벡터
  * $$f_W$$: 파라미터 $$W$$를 가진 함수.

* Vanilla RNN (Elman RNN) 수식:
  * 히든 스테이트 업데이트: $$h_t = \tanh(W_{hh}h_{t-1} + W_{xh}x_t)$$
  * 출력 계산: $$y_t = W_{hy}h_t$$
  * 여기서 $$W_{hh}, W_{xh}, W_{hy}$$는 모든 시점에서 공유되는 가중치 행렬입니다.

### 계산 그래프 (Computational Graph)
RNN은 시간 순서대로 펼쳐진(Unrolled) 계산 그래프로 표현할 수 있습니다.
* Many-to-Many: 매 스텝마다 입력을 받고 출력을 생성하며, 각 단계에서 손실(Loss)을 계산하고 이를 합산하여 전체 손실을 구합니다.
* Many-to-One: 시퀀스를 모두 처리한 후 마지막 히든 스테이트에서만 출력을 생성합니다163].
* One-to-Many: 초기 입력을 받고 이후 시퀀스를 생성합니다190].
* Sequence-to-Sequence: Many-to-One(Encoder)과 One-to-Many(Decoder)가 결합된 형태로, 입력을 벡터로 인코딩한 후 출력을 생성합니다 (예: 번역).

## 학습 및 역전파 (Training & Backpropagation)
* BPTT (Backpropagation Through Time): 전체 시퀀스를 순전파(Forward)하여 손실을 계산한 후, 전체 시퀀스를 역방향으로 거슬러 올라가며 그래디언트(Gradient)를 계산합니다.
* Truncated BPTT: 시퀀스가 매우 긴 경우, 전체를 한 번에 역전파하는 것은 메모리와 연산 비용이 큽니다. 따라서 일정 단위(Chunk)로 잘라서 순전파와 역전파를 수행하는 방식을 사용합니다.

## 응용 사례 1: 문자 단위 언어 모델 (Character-Level Language Model)
RNN을 사용하여 문자열을 학습하고 생성하는 예제입니다.
* 학습 과정: 'hello'라는 단어를 학습할 때, 입력 'h'에 대해 다음 글자 'e'를 예측하도록 학습합니다. 각 글자는 원-핫 벡터(One-hot vector)로 표현됩니다.
* 샘플링 (Sampling): 학습된 모델을 테스트할 때, 모델이 예측한 확률분포에서 문자를 하나씩 샘플링하고, 이를 다시 다음 단계의 입력으로 사용하여 텍스트를 생성합니다.
* 생성 예시:
  * 셰익스피어 (Shakespeare): 셰익스피어 희곡을 학습시킨 후, 그와 유사한 문체의 텍스트를 생성함.
  * 수학 논문 (Algebraic Geometry): LaTeX 소스 코드를 학습시켜, 실제 논문처럼 보이지만 내용은 무의미한 수학 공식을 포함한 문서를 생성함.
  * 리눅스 소스 코드 (Linux Source Code): C 언어 코드를 학습시켜, 변수 선언, 주석, 들여쓰기, 괄호 닫기 등의 문법적 구조를 갖춘 코드를 생성함.

## RNN의 해석 및 시각화 (Visualizing RNNs)
Karpathy 등의 연구(2015)에서는 RNN 내부의 특정 셀(Cell)들이 어떤 역할을 하는지 시각화했습니다.
* 인용구 탐지 (Quote Detection): 따옴표 안의 문장이 시작되면 활성화되고 끝나면 비활성화되는 셀이 발견됨.
* 줄 길이 추적 (Line Length Tracking): 문장이 길어질수록 점진적으로 값이 변하여 줄바꿈 시점을 예측하는 셀.
* 조건문/코드 깊이 추적: `if` 문 내에 있을 때 활성화되거나, 코드 블록의 중첩 깊이에 따라 활성화 정도가 달라지는 셀들이 확인됨.

## 응용 사례 2: 이미지 캡셔닝 (Image Captioning)
이미지를 입력받아 그 내용을 설명하는 문장을 생성하는 기술입니다1043].
* 구조: CNN(Convolutional Neural Network)과 RNN을 결합한 형태입니다.
  * Encoder (CNN): 이미지를 처리하여 특징 벡터(Feature Vector)를 추출합니다 (주로 분류 레이어 직전의 FC layer 사용).
  * Decoder (RNN): CNN에서 추출한 이미지 정보를 초기 상태 또는 입력으로 받아 텍스트 시퀀스를 생성합니다.
* 결과: "밀짚모자를 쓴 사람", "서핑 보드를 든 사람들"과 같이 이미지를 정확하게 묘사하는 캡션을 생성합니다.
* 실패 사례: 학습 데이터의 편향 등으로 인해, 칫솔을 들고 있는 아기를 야구 방망이를 든 야구 선수로 잘못 인식하는 등의 오류가 발생할 수 있습니다.

## RNN의 장단점 (Tradeoffs)
### 장점 (Advantages)
* 입력 데이터의 길이에 제한이 없습니다 (Any length input).
* 이론적으로는 아주 오래전(many steps back)의 정보도 현재 단계의 계산에 활용할 수 있습니다.
* 입력이 길어져도 모델의 크기(파라미터 수)는 증가하지 않습니다.
* 모든 시간 단계에 동일한 가중치를 적용하므로 대칭성(symmetry)이 있습니다.

### 단점 (Disadvantages)
* 순차적으로 계산해야 하므로 연산 속도가 느립니다 (Recurrent computation is slow).
* 실제로는(in practice) 아주 오래전의 정보를 기억하고 활용하는 것이 어렵습니다 (Vanishing Gradient 문제 등 내포).

## 맺음말
> RNN은 가변 길이의 시퀀스 데이터를 처리할 수 있는 강력한 모델로, 텍스트 생성, 코드 생성, 이미지 캡셔닝 등 다양한 분야에서 놀라운 성과를 보여주었습니다. 
> 내부 상태를 통해 과거의 정보를 기억한다는 점과 CNN과 결합하여 멀티모달(Multimodal) 태스크를 수행할 수 있다는 점이 핵심입니다. 

> 연산 속도와 장기 의존성(Long-term dependency) 학습의 어려움이라는 단점이 존재하지만, 이는 LSTM이나 GRU, 그리고 Transformer와 같은 발전된 모델들의 기반이 됩니다.

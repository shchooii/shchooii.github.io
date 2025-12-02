---
title: Deep Learning CNN Architectures
date: 2025-09-25 10:52:58 +09:00
categories: ['deep learning']
tags: ['deep learning', 'CNNs', 'ImageNet']
math: true
---

컴퓨터 비전 분야의 발전, 특히 ImageNet Large Scale Visual Recognition Challenge (ILSVRC) 우승 모델들을 중심으로 CNN 아키텍처의 발전 과정을 다룹니다.
초기 LeNet-5부터 시작하여 네트워크가 점차 깊어지고(Deeper), 효율적으로(Efficient) 변화하는 과정을 확인할 수 있습니다.


## 주요 아키텍처별 특징

### LeNet-5 (1998)
* 특징: 초창기 CNN 모델로, 우편 번호와 같은 손글씨 숫자 인식을 위해 개발되었습니다.
* 구조: [Conv - Pool - Conv - Pool - FC - FC] 형태의 단순한 구조를 가집니다.
* 필터: $$5 \times 5$$ 크기의 컨볼루션 필터를 사용했습니다.


### AlexNet (2012) - 1차 혁명
* 의의: 최초의 CNN 기반 ILSVRC 우승 모델로, 딥러닝 붐을 일으킨 시발점입니다. Top-5 에러율을 16.4%로 획기적으로 낮췄습니다.
* 주요 기법:
  * ReLU 사용: Sigmoid 대신 ReLU 활성화 함수를 처음 도입하여 학습 속도를 높였습니다.
  * Dropout: 과적합(Overfitting) 방지를 위해 0.5 비율의 Dropout을 사용했습니다.
  * Data Augmentation: 데이터 증강 기법을 적극적으로 활용했습니다.
  * GPU 활용: 당시 메모리 제약으로 인해 네트워크를 두 개의 GPU로 나누어 학습시켰습니다.
* 구조: 8개의 레이어(5 Conv + 3 FC)로 구성되어 있습니다.


### ZFNet (2013)
* 특징: AlexNet의 하이퍼파라미터를 개선한 모델입니다.
* 개선점:
  * 첫 번째 레이어의 필터 크기를 $$11 \times 11$$에서 $$7 \times 7$$로 줄이고, Stride를 4에서 2로 줄여 정보 손실을 최소화했습니다.
  * 중간 레이어(Conv3, 4, 5)의 필터 수를 늘려 성능을 향상했습니다.

### VGGNet (2014) - 깊이(Depth)의 미학
* 핵심 아이디어: "작은 필터를 깊게 쌓자"는 철학으로 설계되었습니다.
* 구조: $$3 \times 3$$ 크기의 작은 필터만을 사용하여 16~19층까지 깊이를 늘렸습니다 (VGG16, VGG19).
  * $$3 \times 3$$ 필터의 장점: $$3 \times 3$$ 필터를 3번 쌓으면 $$7 \times 7$$ 필터 1번과 동일한 수용 영역(Receptive Field)을 가지면서도, 파라미터 수는 줄어들고 비선형성(Non-linearity)은 증가합니다.
* 단점: FC(Fully Connected) 레이어 비중이 높아 파라미터 수가 매우 많고(약 1억 3,800만 개), 메모리 사용량이 큽니다.


### GoogLeNet (Inception V1) (2014) - 효율성(Efficiency)
* 특징: ILSVRC 2014 우승 모델로, 22개의 레이어를 가지면서도 파라미터 수는 AlexNet의 1/12 수준(500만 개)으로 매우 효율적입니다.
* Inception 모듈: $$1 \times 1$$, $$3 \times 3$$, $$5 \times 5$$ 컨볼루션과 $$3 \times 3$$ 풀링을 병렬로 수행하고 결과를 합치는(Concatenate) 구조입니다.
* Bottleneck 구조 ($$1 \times 1$$ Conv): 연산량 감소를 위해 $$3 \times 3$$이나 $$5 \times 5$$ 컨볼루션 전에 $$1 \times 1$$ 컨볼루션을 배치하여 채널 수(Depth)를 줄였습니다.
* Global Average Pooling: 마지막에 FC 레이어 대신 Global Average Pooling을 사용하여 파라미터 수를 획기적으로 줄였습니다.


### ResNet (2015) - 잔차 학습(Residual Learning)
* 혁신: 네트워크가 너무 깊어지면 오히려 성능이 떨어지는(Degradation) 문제를 해결하여 152층이라는 초심층 모델을 구현했습니다.
* Residual Block: 입력을 출력에 더해주는 Skip Connection ($$F(x) + x$$) 구조를 도입했습니다. 이는 네트워크가 입력을 그대로 보존하는 'Identity Mapping'을 쉽게 학습하도록 도와줍니다.
* 성과: 인간의 인식 능력(에러율 약 5.1%)을 뛰어넘는 3.57%의 에러율을 기록했습니다.


### 이후 발전 모델
* Inception-v4: ResNet의 잔차 연결(Skip Connection)과 Inception 모듈을 결합한 형태입니다.
* SENet (2017): Squeeze-and-Excitation 모듈을 도입했습니다. 이는 특징 맵(Feature map)의 채널별 중요도를 학습하여 가중치를 재조정(Recalibration)하는 방식입니다.

## 비교 및 분석 (Comparison)
* 정확도 vs 연산량:
  * VGGNet은 정확도는 높지만 파라미터가 가장 많고 연산량이 큽니다(비효율적).
  * GoogLeNet은 매우 효율적이며 적은 파라미터로 높은 성능을 냅니다.
  * ResNet은 깊이에 비해 효율적이며 가장 높은 정확도를 보여줍니다.
* 트렌드: AlexNet 이후 네트워크는 더 깊어지고(Deep), 파라미터 효율성(Efficiency)을 추구하는 방향으로 발전했습니다.

## 맺음말

> CNN 아키텍처가 단순한 구조에서 시작하여, 깊이와 효율성을 동시에 잡기 위해 Inception Module ($$1 \times 1$$ Conv)이나 Residual Connection 같은 구조로 발전해 온 역사를 보여줍니다.

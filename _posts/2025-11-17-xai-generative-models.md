---
title: XAI Generative Models
date: 2025-11-17 18:13:12 +09:00
categories: ['xai']
tags: ['xai', 'gan', 'vae']
math: true
---

생성 모델은 데이터 분포를 학습하여 새로운 샘플을 생성하는 모델로, 최근 인공지능 연구와 응용 전반에서 핵심적인 역할을 수행하고 있습니다. 이러한 생성 모델은 단순히 데이터를 만들어내는 데 그치지 않고, 설명가능 인공지능 관점에서 두 가지 중요한 역할을 가집니다. 
첫째, 생성 모델은 기존 설명 기법을 보완하는 설명자, Explainer로 활용될 수 있습니다. 
둘째, 생성 모델 자체가 복잡한 블랙박스 모델이므로, 그 내부 동작을 이해하기 위한 설명 대상, Explainee가 됩니다.
생성 모델의 기본 개념을 출발점으로, 생성 모델을 활용한 XAI 방법과 생성 모델 자체에 대한 XAI 접근을 수식과 함께 체계적으로 정리합니다.

## Generative Models 개요

생성 모델은 학습 데이터가 따르는 분포 $$p_{\text{data}}(x)$$를 근사하는 모델로, 이 분포로부터 새로운 샘플 $$x \sim p_\theta(x)$$를 생성하는 것을 목표로 합니다. 
분류 모델이 조건부 확률 $$p(y|x)$$를 학습하는 것과 달리, 생성 모델은 데이터 자체의 구조와 다양성을 학습한다는 점에서 근본적인 차이를 가집니다.

생성 모델은 이미지, 텍스트, 음성, 분자 구조, 시계열 등 다양한 도메인에 적용될 수 있습니다.

### GAN

Generative Adversarial Network는 생성기 $$G$$와 판별기 $$D$$의 적대적 학습을 통해 데이터 분포를 학습합니다. 생성기는 잠재 변수 $$z \sim p(z)$$로부터 가짜 샘플 $$x' = G(z)$$를 생성하고, 판별기는 입력 $$x$$가 실제 데이터인지 생성된 데이터인지를 판별합니다.

GAN의 기본 목적 함수는 다음과 같이 정의됩니다.

$$
\min_G \max_D
\mathbb{E}_{x \sim p_{\text{data}}}[\log D(x)] + \mathbb{E}_{z \sim p(z)}[\log(1 - D(G(z)))]
$$

GAN은 매우 선명한 이미지를 생성할 수 있다는 장점이 있으나, 일부 모드만 생성하는 mode collapse 문제가 발생할 수 있습니다.

### VAE

Variational Autoencoder는 확률적 인코더 $$q_\phi(z|x)$$ 와 디코더 $$p_\theta(x|z)$$ 를 학습하여 잠재 공간을 명시적으로 모델링합니다. 
학습 목표는 Evidence Lower Bound를 최대화하는 것입니다.

$$
\mathcal{L}_{\text{ELBO}}
=
\mathbb{E}_{q_\phi(z|x)}[\log p_\theta(x|z)] - \mathrm{KL}(q_\phi(z|x)\|p(z))
$$

VAE는 사용이 비교적 간단하고 안정적이지만, 재구성 이미지가 흐릿해지는 경향이 있습니다.

### Flow-based Models

Flow 기반 모델은 가역 함수 $$f$$를 학습하여 데이터 공간과 잠재 공간을 정확히 연결합니다.

$$
z = f(x), \quad x = f^{-1}(z)
$$

확률 밀도는 변수 변환 공식으로 계산됩니다.

$$
\log p(x) = \log p(z) + \log \left| \det \frac{\partial f}{\partial x} \right|
$$

Flow 모델은 정보 손실이 없지만 계산 비용이 매우 높다는 단점이 있습니다.

## Generative Models as Explainers

### 기존 Attribution Baseline의 한계

Integrated Gradients와 같은 gradient 기반 설명 방법은 baseline 선택에 크게 의존합니다. 일반적으로 사용되는 baseline은 영벡터 이미지나 블러 처리된 이미지입니다. 그러나 이러한 baseline은 실제 데이터 분포와 동떨어져 있으며, 모델의 결정 경계를 정확히 반영하지 못합니다.

이로 인해 특정 픽셀이 중요하게 강조되었을 때, 그것이 실제로 중요한 특징이어서인지 단순히 baseline과의 값 차이 때문인지를 구분하기 어렵습니다.

### 좋은 Baseline의 조건

강의에서는 좋은 baseline $$B$$가 다음 조건을 만족해야 한다고 정의합니다.

- 입력 $$x$$와 거리가 가까워야 합니다.
- 데이터 분포 내부에 존재해야 합니다.
- 타겟 클래스에 속하는 반사실적 샘플이어야 합니다.

이를 수식으로 표현하면 다음과 같습니다.

$$
B_{c_t}(x) = \arg\min_{\tilde{x} \in \mathcal{R}^N}
\left(
\|x - \tilde{x}\| - \log R(\tilde{x}) - \log S_{c_t}(\tilde{x})\right)
$$

여기서 $$R(\tilde{x})$$는 현실적인 이미지일 확률을, $$S_{c_t}(\tilde{x})$$는 타겟 클래스 점수를 의미합니다.

### GANMEX

GANMEX는 StarGAN을 활용하여 이상적인 baseline을 생성하는 방법입니다. StarGAN은 하나의 생성기로 여러 도메인 간 변환을 학습할 수 있는 모델입니다.

StarGAN 생성기의 손실 함수는 다음 항들의 결합으로 구성됩니다.

$$
\mathcal{L}_G
=
\mathcal{L}_{adv} + \lambda_{cls}\mathcal{L}_{cls} + \lambda_{rec}\mathcal{L}_{rec}
$$

GANMEX에서는 판별기 대신, 설명 대상 분류기 $$S(x)$$를 사용하여 다음과 같은 손실을 정의합니다.

$$
\mathcal{L}_G^{GANMEX}
=
\mathbb{E}_x[\log(1 - D(G(x,c)))] - \lambda_{cls} \log S_{c_t}(G(x,c)) + \lambda_{rec}\|x - G(x,c)\|
$$

이를 통해 입력과 유사하면서도 다른 클래스에 속하는 반사실적 baseline을 생성합니다. 이 baseline을 사용한 IG 결과는 노이즈가 적고, 실제 변별적 특징만을 강조합니다.

## Generative Models as Explainees

### 생성 모델에서 XAI의 필요성

생성 모델은 자동 평가가 어렵고, 출력이 다양하며, 고위험 응용에 사용될 수 있기 때문에 설명가능성이 특히 중요합니다. 강의에서는 생성 모델에서 XAI가 필요한 이유로 출력 조절, 검증, 안전성, 보안, 그리고 잠재적 오용 방지를 제시합니다.

### Latent Space와 Controllability

생성 모델의 핵심은 잠재 공간 $$Z$$입니다. 좋은 생성 모델이란, 잠재 공간의 이동이 의미 있는 시각적 변화로 이어지는 모델입니다. 따라서 XAI의 핵심 역시 잠재 공간을 이해하고 제어하는 데 있습니다.

### Inversion

Inversion은 실제 이미지 $$x$$에 대응하는 잠재 벡터 $$z^*$$를 찾는 과정입니다.

$$
z^* = \arg\min_z \ell(G(z), x)
$$

Optimization 기반 inversion은 높은 재현도를 가지지만 매우 느리며, Encoder 기반 inversion은 빠르지만 세부 정보를 놓치는 경향이 있습니다. Hybrid 방식은 두 방법의 절충안입니다.

### Latent Space Manipulation

잠재 코드 $$z^*$$에 방향 벡터 $$n$$을 더함으로써 속성을 편집할 수 있습니다.

$$
z^{*'} = z^* + \alpha n
$$

여기서 $$\alpha$$는 변화의 강도를 조절합니다. 방향 벡터는 지도 학습 기반 분리 평면의 법선 벡터나, CLIP 임베딩 차이를 통해 얻을 수 있습니다.

CLIP 기반 방식은 다음과 같이 정의됩니다.

$$
z^{*'} = z^* + \alpha (z_t^{target} - z_t^{source})
$$

또는 그래디언트 기반으로 정의됩니다.

$$
z^{*'} = z^* + \alpha \nabla_z \cos(CLIP(G(z)), t)
$$

### 응용과 한계

이러한 조작은 이미지 편집, 복원, 스타일 전이 등에 활용됩니다. 그러나 OOD 입력에 취약하며, 속성 간 완전한 분리가 어렵고, 속도와 품질 간 트레이드오프가 존재합니다.

## 맺음말

> 생성 모델은 XAI에서 설명의 도구이자 설명의 대상이라는 이중적 역할을 수행합니다. 
> GANMEX와 같은 방법은 기존 설명 기법의 한계를 보완하며, inversion과 latent manipulation은 생성 모델을 해석 가능하고 제어 가능한 시스템으로 전환시킵니다. 
> 향후에는 GAN을 넘어 Diffusion 모델 기반 inversion과 설명 방법이 XAI 연구의 중심으로 자리 잡을 것으로 전망됩니다.

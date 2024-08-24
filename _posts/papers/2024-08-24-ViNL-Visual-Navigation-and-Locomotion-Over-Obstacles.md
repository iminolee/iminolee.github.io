---
layout: post
title: "ViNL: Visual Navigation and Locomotion Over Obstacles"
date: 2024-08-24 22:00:00 +0900
categories: [papers]
tags: [RL, Locomotion, Navigation]
toc: true
comments: true
published: true
---

> Kareer, Simar, et al. "Vinl: Visual navigation and locomotion over obstacles." 2023 IEEE International Conference on Robotics and Automation (ICRA). IEEE, 2023. [[Paper]](https://ieeexplore.ieee.org/abstract/document/10160612/) [[Project Page]](https://www.joannetruong.com/projects/vinl.html)

![Figure1](/assets/img/papers/2024-08-24/Figure1.png){: width="100%" }

## Abstract 
이 논문에서는 *unseen environments*를 탐색하면서 경로에 놓인 작은 장애물들을 넘는 사족로봇을 위한 *Visual Navigation*과 *Locomotion*을 결합한 **ViNL**을 제안하고 있다. 이는 사람이나 동물이 걷는 동안 물체를 넘을 때 발을 들어 올리는 방식과 유사하다.

구체적으로, ViNL은 (1)미지의 실내 환경에서 로봇을 목표 지점으로 가는 선형 및 각속도 명령을 출력하는 *Visual Navigation Policy*와, (2)제공된 속도 명령을 따르면서 장애물에 발이 닿지 않도록 로봇의 관절을 제어하는 *Visual Locomotion Policy*로 구성된다. 이 두 가지 정책은 모두 *model-free*하게 *end-to-end* 방식으로 학습된다. 두 정책은 완전히 다른 시뮬레이터, *Navigation Policy*는 Habitat에서 *Locomotion*은 Isaac Gym에서 독립적으로 학습된 후, 내비게이션의 속도 명령을 locomotor에 전달되어 통합된다.

이 논문의 저자는 시각 정보를 활용하여 미지의 환경에서의 지능형 내비게이션과 장애물에 방해받지 않고 복잡한 환경을 통과할 수 있는 지능형 로코모션을 모두 달성하는 *fully learned approach*는 이전 연구에서는 없던 최초의 시도임을 시사하고 있다.

실험 결과에서는 미지의 환경에서 목표 지점으로 이동하는 작업에서 ViNL은 *egocentric vision*을 사용하여 *privileged terrain maps*를 활용한 이전 연구들보다 성공률이 32.8% 높고, 미터당 충돌 빈도가 4.42회 낮았음을 보여주고 있다. 또한, *Locomotion Policy*를 분석하여 이 접근 방식의 각 측면이 장애물 충돌을 줄이는 데 기여함을 보인다.

## 1. Introduction
Real-world에서 모바일 로봇은 이전에 본 적이 없는 환경에서 능숙하게 탐색할 수 있어야 한다. 이는 특히 실내 환경에서 중요한데, 실내 환경은 가구 배치의 변경이나 임시적인 혼잡과 같은 변동에 자주 노출되기 때문이다. 최근 몇 년간 *Deep Reinforcement Learning*의 발전, 실제 실내 스캔의 방대한 데이터셋, 그리고 확장 가능한 *photorealistic simulator* 덕분에 학습된 에이전트를 활용한 실내 내비게이션이 큰 진전을 이루었다. 이전의 연구들에서는 훈련된 에이전트가 사전에 구축된 지도를 사용하지 않고도 이전에 본 적이 없는 환경에서도 *deploy*가 가능함을 보여주고 있다.

그러나, 현재 미지의 환경에서 비주얼 실내 내비게이션의 발전은 대부분 깨끗하고 평평한 지형을 갖춘 가정에서 wheeled-base robot을 사용하는 것으로 제한되어 있다. 일반적인 wheeled robot은 실내 가정 환경에서도 maneuverability의 제약이 있어 장애물, 문턱이 있는 출입구, 계단, 두꺼운 카펫 등을 넘는 데 어려움을 겪는다. 반면에 legged robot은 이러한 조건에서 탐색하는 데 더 적합하며 또한 장애물을 방해하거나 손상시키지 않고 넘을 수 있다. Legged robot의 locomotion을 학습하는 연구는 이전에 여럿 있었지만, 주로 실외 환경을 위한 blind 및 reactive policy를 사용하며 rough terrain에서의 안정성과 강인한 회복을 강조하고 있기 때문에, 본 연구에서는 미지의 실내 환경에서 흔히 발견되는 작은 장애물들을 넘으며 탐색할 수 있는 legged robot의 ViNL 프레임워크를 제안하고 있다.

제안된 프레임워크에서 *low-level visual locomotion policy*는 NVIDIA의 Isaac Gym 시뮬레이션 환경을 사용하여 세 단계로 학습된다. 첫 번째로, 사전 훈련 없이 대규모 심층 강화학습을 통해 다양한 도전적인 지형에서 걷는 법을 학습하게 된다. 여기서 로봇은 주변의 고도 샘플에 대한 privileged terrain map을 활용하여 걷고 속도 명령을 따르는 법을 배운다. 두 번째로, 새로운 지형에서 이전 정책을 미세 조정하여 지면에 작은 장애물이 있는 상황에서 걷는 법을 학습한다. 이 단계에서는 로봇이 privileged terrain map을 사용하여 장애물을 피하는 법을 학습한다. 마지막으로, egocentric vision을 사용하여 오직 vision을 통해 privileged terrain map을 재구성하는 지도학습을 통해 장애물 위를 걷는 법을 학습한다. 이전 단계들과 달리, 이 visual policy는 *LSTM*을 사용하여 이전의 시각적 관찰과 proprioceptive states의 기억을 유지한다. 이는 로봇이 장애물이 시야에서 벗어났을 때도 장애물을 넘을 수 있게 해준다.

또한, real-world의 실내 환경을 photorealistic 3D 스캔한 데이터를 사용하여 Habitat Simulator에서 *high-level visual navigation policy*를 학습한다. 이전 연구와 유사하게, 이 내비게이션 정책은 시뮬레이션에서 kinematic control을 사용하여 로봇의 질량 중심에 대한 선형 및 각속도를 명령하는 방식으로 학습된다. Locomotion과 Navigation Polic는 서로 다른 시뮬레이터에서 학습되지만, 두 정책은 사전 훈련 없이도 장애물 위를 이동하는 visual navigation task에 결합된다.

![Demo](/assets/img/papers/2024-08-24/demo.gif){: width="100%" }

## 2. Related Work
본 논문에서는 세 가지 관련 연구 키워드를 제시하고 있다. 1) Legged Locomotion; 2) Visual Locomotion; 3) Visual Navigation.

**Legged Locomotion**에서 특히 다양한 보행(gait)에 대한 모델 기반 제어 설계와 관련된 이전 연구와 관련성이 높다. 여기에는 동물의 민첩한 행동을 모방하는 *imitation learning*과 *locomotion controller*를 직접 학습하는 심층 강화 학습 접근법이 포함된다. 이러한 locomotion 연구들은 일반적으로 시각 정보 없이 안정성과 회복력을 강조하는데, 실제 거친 지형이나 미끄러운 표면 그리고 경사에서 걷는 로봇의 높은 안정성을 보여준다. 본 연구는 이러한 안정성과 회복력에 초점을 맞추기보다는 위험이 존재하는 발판이나 장애물과의 충돌을 피하면서 목표 지점으로 가는 경로를 탐색하는 데 중점을 두고 있다.

**Visual Locomotion**의 일부 연구들은 시각 정보 없이 locomotion을 수행하는 것을 넘어, 로봇 주변의 지형을 미리 파악하기 위해 LiDAR 센서를 사용하여 발판을 선택하는 방법을 제안하였다. 그러나 이러한 LiDAR 센서는 고가이며, 많은 legged robot에 기본적으로 포함되어 있지 않다. 최근 연구들은 LiDAR 대신 카메라를 활용하는 방법을 탐구하고 있으며, 이는 비용 효율적이고 확장 가능한 솔루션이다. 여러 연구들은 불연속적인 지형에서 '발판 위치'를 안내하기 위해 시각 정보를 사용하고 있지만, 저자는 실내 내비게이션에서 로봇이 발판을 건너서 가는 것보다 '이러한 장애물을 넘는 것'이 더 중요하고 어려운 과제라고 이야기한다.

**Visual Navigation**은 일반적으로 정적, 사회적 또는 상호작용 환경에서 연구되며 wheeled robot에 대해 실제 환경으로의 전이가 가능함이 입증되었다. 정적 내비게이션에서는 로봇이 가구 등의 고정적인 장애물이 있는 실내 환경을 탐색해야 하며, 사회적 내비게이션에서는 로봇이 보행자와 충돌하지 않도록 목표 지점으로 탐색해야 한다. 정적 및 사회적 내비게이션은 궁극적으로 내비게이션 정책이 큰 장애물(가구 또는 보행자)을 피하는 것을 학습하도록 하며, legged robot의 세밀한 로코모션과 제어가 필요하지 않을 수 있다. 하지만, 본 연구에서 다루는 상호작용 내비게이션에서는 장애물과의 상호작용을 피하고 환경에 대한 방해를 최소화하며 내비게이션을 수행하는 것을 목표로 하고 있다.

## 3. Method
제안된 ViNL(Visual Navigation and Locomotion)은 실내 환경에서 장애물을 넘으면서 탐색할 수 있는 계층적 구조를 갖고 있다. 이 구조는 (1)작은 장애물을 피하도록 학습된 Visual Locomotion Policy와 (2)선형 및 각속도를 명령하는 Visual Navigation Policy로 구성되어 있다. 이 Locomotion Policy는 50Hz로 작동하며, Navigation Policy는 2Hz로 작동한다. 두 정책은 병렬로 독립적으로 학습된다.

### 3.1. Visual Locomotion

* Stage 1: Learning to Walk

장애물을 넘는 법을 학습시키기 전에, *Isaac Gym* 벤치마크 환경을 사용하여 Deep Reinforcement Learning(PPO)을 통해 사족 로봇에게 명령된 선형 및 각소도를 따르면서 다양한 도전적인 지형에서 강건하게 걷는 법을 학습시킨다. 이 단계에서 로봇은 고유 감각(proprioceptive) 상태와 1.6m x 1.0m 격자 범위로 정의된 로봇 주변의 높이 샘플로 이루어진 privileged height map(*H*)를 입력으로 받는다. Timestep(*t*)에서의 관찰 공간은 관절 위치, 관절 속도, 이전 관절 명령, 기본 선형 속도, 기본 각속도, 투영 중력 벡터, 명령된 속도 및 지형 지도 *H*로 구성된다. Locomotion policy의 출력은 PD motor controllers의 목표 위치로 사용되는 관절 각도이다. 이때, 로봇이 명령된 속도를 따르면서 부드럽고 자연스러운 걸음을 유지하도록 보상 시스템을 적용한다.

*Actor & Critic Networks*는 각각 3-layer의 MLP로 구성되어 있으며, 각 층은 256개의 hidden unit을 가지고 있다. Actor network는 diagonal Gaussian distribution을 매개하는 평균과 표준 편차를 출력하며, 이 분포로부터 행동이 샘플링된다. 특히 로봇 지형의 로컬 맵을 다른 관찰 값들과 직접 연결하는 것과 달리, 먼저 로컬 맵을 128,64,32개의 hidden unit을 가진 3-layer MLP 인코더 ϕ로 별도로 인코딩한다. 이 맵 인코딩은 나머지 관찰 값들과 함께 정책에 전달되며, 이는 아래 **[그림]** 과 같다.

![Figure2](/assets/img/papers/2024-08-24/Figure2.png){: width="100%" }

 이를 통해 *Learning by Cheating* 방법을 접근 방식의 3단계에서 사용할 수 있게 되며, <U>여기에서 *privileged map MLP encoder*의 knowledge를 egocentric depth observations를 사용하는 encoder τ로 증류(distill)할 수 있다.</U> 본 논문에서 정책은 *Proximal Polic Optimization(PPO)* 알고리즘을 사용하여, Isaac Gym 시뮬레이션 환경에서 50개의 병령 환경과 500의 배치 크기로 훈련하였다고 한다.

* Stage 2: Learning to Walk Over Clutter

앞선 1단계의 Locomotion Policy는 이미 걷는 데 있어서 견고하지만, 이제 로봇이 장애물을 밟지 않도록 학습시켜야 한다.

![Figure3](/assets/img/papers/2024-08-24/Figure3.png){: width="100%" }

먼저, 지면에 작은 블록들이 놓여진 장애물 지형을 구성한다. 학습에 사용된 환경에서 블록의 크기는 너비 0.2m에서 0.5m, 길이는 0.2m로 고정, 높이 0.1m에서 0.2m 사이에서 무작위로 결정된다. 로봇은 장애물이 없는 지형에서 시작해, 임의로 샘플링된 선형 및 각속도를 따르면서 장애물이 있는 지형으로 진입한다. 점진적으로 장애물 밀도를 증가시키는 커리큘럼을 사용하여 로봇이 장애물을 넘는 법을 학습하도록 한다. 로봇이 지면에 있는 장애물과 다리가 접촉할 때마다 페널티(*-k*)를 부과하여, 로봇이 장애물을 피하는 법을 학습하도록 유도한다. 이 단계에서 로봇이 자연스러운 걸음걸이를 유지하면서 장애물을 피하는 법을 학습하며, pretraining 없이 장애물을 넘는 법을 학습시켰을 때는 로봇이 다리를 너무 높이 들어 올려 부자연스러운 걸음걸이를 보였으나, pretraining을 거친 후 fine tuning을 하였을 때 로봇이 자연스러운 걸음걸이를 유지하면서 장애물을 효과적으로 피할 수 있음을 검증하였다.

* Stage 3: Learning to Walk Over Clutter with Vision

마지막 단계에서는 privileged terrain map에 접근할 필요 없이 egocentric vision(*d*)을 사용해 장애물을 넘는 법을 학습한다. 이 단계에서는 로봇이 30도 아래로 기울어진 전방 depth camera에서 수집한 egocentric vision을 사용해 privileged terrain map의 인코딩을 예측하도록 학습시킨다. 아래 그림은 vision을 사용하여 로봇이 경로 상의 장애물을 피하기 위해 발을 들어올리는 법을 가이드하는 locomotion policy의 예시를 보여준다.

![Figure4](/assets/img/papers/2024-08-24/Figure4.png){: width="100%" }

*[Learning by Cheating(Chen et al., CoRL, 2019)](https://par.nsf.gov/biblio/10145645)* 을 사용하여 로봇의 정책이 고정된 상태에서, egocentric depth observations와 로봇의 proprioceptive states를 입력으로 받아 privileged terrain map *ϕ(H)* 의 인코딩을 예측하는 인코더를 학습한다. *LSTM* τ를 사용해 정책에 시계열 데이터를 제공하며, timestep *t*에서, *LSTM*은 CNN *ψ(dt)* 에서 얻은 깊이 이미지의 시각적 인코딩, 로봇의 고유 감각 상태 *p_t*, 그리고 이전 hidden state *h_t-1*을 입력으로 받아 현재 지형 맵 *ϕ(H_t)* 의 인코딩을 예측한다. 이렇게 3-layer CNN과 2-layer LSTM을 supervised learning으로 학습하며, 정책을 고정한 채로 MSE(Mean Squared Error) 손실을 최소화하는 과정을 통해 로봇은 이전의 시각 정보와 고유 감각 상태를 기억하고 이를 바탕으로 현재 상태를 예측함으로써 장애물이 시야에서 사라져도 이전에 얻은 정보를 바탕으로 계속해서 장애물을 피할 수 있게 된다.

### 3.2. Visual Navigation

* Task: PointGoal Navigation

*[PointGoal Navigation(Anderson, Peter, et al., arXiv, 2018)](https://arxiv.org/abs/1807.06757)* 에서는 로봇이 사전에 구축된 환경 지도를 받지 않고 실내 환경에서 목표 지점(보통 5~30m 거리)에 도달해야 한다. 이때, 로봇은 egocentric depth camera와 출발 위치에서의 상대 위치와 방향을 나타내는 gyroscope 데이터에 접근할 수 있다. 모굪 위치는 로봇의 출발 위치를 기준으로 polar coordinates로 지정되며 에피소드는 로봇이 목표에 0.325m 이내(*AlienGo* 로봇의 half the length)에 도달하면 성공으로 간주된다.

* Dataset: Habitat-Matterport(HM3D) & Gibson 3D

본 연구는 Habitat-Matterport(HM3D) 및 Gibson 3D 데이터셋을 사용하며, 이 데이터셋에는 다양하고 현실적인 1000개 이상의 실내 환경(가정, 사무실 등)의 스캔이 포함되어 있다.

![Figure6](/assets/img/papers/2024-08-24/Figure6.png){: width="100%" }

* Kinematic Visual Navigation

Navigation Policy는 egocentric depth image와 로봇의 목표 지점까지의 거리 및 방향을 입력으로 받는다. 이를 통해 egomotion 추정과 goal coordinate를 계산할 수 있으며, 둘 다 로봇의 시작 위치를 기준으로 지정된다. Navigation policy의 출력은 로봇의 질량 중심이 원하는 선형 속도와 각속도를 나타내는 2차원 벡터이며, egocentric depth image를 처리하기 위해 *ResNet-18 visual encoder* 가 사용된다. 목표 벡터는 단일 선형 레이어를 사용해 인코딩되고, 그 출력은 시각 인코더의 출력과 결합되어 *downstream 2-layer LSTM policy* 로 전달된다. Locomotion policy와 유사하게, 정책의 최종 레이어는 diagonal Gaussian distribution을 매개하는 평균과 표준 편차를 출력하며, 이 분포로부터 actions이 샘플링된다. 이 Navigation Policy의 전체 아키텍처는 아래 그림에 나와있다. 본 프레임워크에서 경로 효율성을 높이고 환경과의 충돌을 방지하기 위해 해당 연구가 제안한 [Reward Function(Truong, Joanne, et al., CoRL, 2022)](https://openreview.net/forum?id=BxHcg_Zlpxj&utm_campaign=The%20Batch&utm_source=hs_email&utm_medium=email&_hsenc=p2ANqtz-_ngka00E370d0Z7YmBJbLw9wGM9AGwUbDtM--kZsGHjrjUxtlu_1M7uZ3SwDDBZFaJ0J7w)를 차용하였다.

Navigation Policy를 학습하기 위해, 로봇의 움직임에 대한 근사로서 운동학적 제어를 사용한다. 각 단계에서 명령된 속도를 2Hz로 통합하여 (완전한 강체 동역학 시뮬레이션 없이) 로봇을 직접 이동시킨다. 만약 결과 위치가 환경과 충돌하면, 유효한 속도 명령이 주어질 때까지 로봇은 현재 위치에 그대로 유지된다. 이 공식은 저수준 물리적 상호작용을 추상화하여 더 빠른 시뮬레이션을 통해 더 많은 경험을 축적함으로써, 더 나은 sim-to-real transfer를 이끄는 것으로 나타났다.

그러나 내비게이션 학습 중 로봇이 운동학적으로 제어되기 때문에, 로봇의 깊이 관찰은 완전히 지면과 평행하게 유지되어 움직이는 사족 로봇에 장착될 때 발생하는 카메라 흔들림을 정확하게 반영하지 못한다. 따라서 내비게이션 정책의 강건성을 향상시키기 위해, 학습 중 각 단계에서 전방 카메라의 roll과 pitch를 ±30도까지 무작위로 변경한다.

정책은 분산 강화학습 방법인 *DD-PPO*를 사용하여 Habitat 시뮬레이터에서 훈련된다. Habitat-Sim은 photorealistic 3D 시뮬레이션 환경의 빠른 렌더링을 지원하며, 이전 연구에서 wheeled robot과 legged robot 모두에 대해 real world로 성공적으로 전이될 수 있음을 보여주었다. 본 연구에서는 8개의 GPU를 사용하여 3일동안(500M steps of experience) 정책을 훈련해 수렴을 보장하였다.

## 4. Results
결과 섹션에서는 다음과 같은 질문을 다루고 있다.

* 제안하고 있는 *Visual Navigation Policy*는 다른 시뮬레이터와 다른 저수준 컨트롤러에서 얼마나 잘 작동할 수 있는가?
* *ViNL*은 이전 연구들과 비교하여 장애물을 넘으면서 얼마나 잘 탐색할 수 있는가?
* *Locomotion Policy*에 있어서 외부 센서(exteroception), pre-training, 혹은 LSTM layers의 사용이 얼마나 중요한가?

본 연구는 Isaac Gym에서 여러 가지 복잡한 환경에서 *PointGoal Navigation* 작업을 통해 평가하고 있다. 각 환경에서 로봇의 시작 지점과 목표 지점을 임의로 샘플링하여 약 7m 떨어진 거리를 설정한다. 환경은 미로와 같은 레이아웃으로 방들이 배치되어 있으며, 탐색 과정에서 탐색과 되돌아가기(backtracking)가 필요할 수 있다. 각 방법별로 5개의 시드에 대해 250번의 실험을 수행한 결과를 보고하며, 평가 중 Navigation Policy는 위로 15도 기울어진 전방 깊이 카메라를 사용하고, Locomotion Policy는 아래로 30도 기울어진 전방 깊이 카메라를 사용한다.

먼저 ViNL의 성능 상한값을 얻기 위해 장애물이 없는 환경에서 *PointGoal Navigation*의 성능을 평가하였으며, 이를 통해 Navigation Policy가 '도메인 내'(Habitat에서 운동학적 저수준 제어와 함께)와 '도메인 외'(Isaac Gym에서 ViNL을 사용하여 저수준 제어와 함께)에서 어떻게 성능이 나타나는지 비교하였다. 도메인 내에서는 비주얼 네비게이션 정책이 HM3D 데이터셋의 검증 세트에서 1000개의 에피소드를 사용해 평가되었으며, SR(Success Rate)은 89.70%를 기록하였다. 도메인 외에서는 ViNL이 86.8%의 SR을 기록하였으며, 이는 시뮬레이터와 저수준 로코모션 컨트롤러가 다름에도 불구하고 성능 차이가 2.9%에 불과함을 나타낸다. 이는 내비게이션 정책과 로코모션 정책을 서로 다른 시뮬레이터와 추상화 수준에서 독립적으로 훈련했음에도 불구하고, 두 정책이 거의 매끄럽게 결합되어 내비게이션 성능이 최소한으로 떨어졌다는 것을 입증한다. ViNL은 장애물이 없는 환경에서 평가되었기 때문에, 아래 [표1]의 6번째 줄에 있는 미터당 2.9번의 발 충돌은 환경 내의 벽과의 충돌에서 발생한 것이다.

![Table1](/assets/img/papers/2024-08-24/Table1.png){: width="100%" }

다음으로, ViNL을 다른 baselines 및 ablations과 비교한다. 실험에서, 동일한 고수준 내비게이션 정책을 다음의 로코모션 정책들과 결합한다:

* **Blind**: Rough terrain에서 학습된 locomotion policy로 다리 접촉 페널티와 함께 장애물 지형에서 미세 조정되었다. 이 정책은 고유 감각 정보만을 사용하여 장애물을 피한다. 이 baseline은 고유 감각 정보와 외부 센서의 value를 측정한다.
* **Rough**: Rough terrain에서 학습된 locomotion policy로 다리 접촉 페널티는 적용되지 않았다. 이 정책은 로코모션을 위해 privileged terrain map에 접근할 수 있다. vision 정보를사용하여 장애물을 넘는 것과 복잡한 지형에서의 강인한 회복을 위한 privileged terrain map을 사용하는 것의 value를 평가하여 비교한다.
* **ViNL (no pre-training)**: 모든 훈련이 장애물 지형에서 수행되었다. 이 로코모션 정책은 egocentric depth observations을 사용하며, 이 실험은 자연스러운 보행을 학습하고 강인성을 개선하기 위한 pre-training 단계의 중요성을 정량화한다.
* **ViNL (MLP)**: 인코더가 *LSTM* 대신 *MLP*를 사용한다. 이 로코모션 정책은 egocentric depth observations을 사용하며, 이 실험은 시계열 정보의 value를 측정한다.

모든 baseline은 동일한 경험량(500M steps)를 기준으로 학습되었다. 성공률(SR), 이동거리, 미터당 발 충돌 횟수를 평가 지표로 사용한다. 미터당 발 충돌 횟수는 로봇이 최소 1m 이상 탐색한 에피소드에서 계산된다. 이는 로봇이 시작 지점 근처에서 넘어지는 짧은 에피소드에서 발 충돌이 인위적으로 낮아지는 것을 방지하기 위함이다. 또한, 저수준 컨트롤러는 50Hz로 작동하므로, 발 접촉이 1초 동안 지속될 경우 50번의 발 충돌로 계산된다. 위의 [표1]에 모든 지표의 평균과 표준 편차를 확인할 수 있다.

[표1]의 지표에서 보이듯이, **Blind**는 매우 낮은 성공률을 나타내는데, 이는 복잡한 환경에서 외부 센서를 사용하는 것이 중요함을 보여준다. **Rough**와 비교했을 때, 제안된 방법은 평균 32.8%의 성공률 증가와 평균 4.42개의 미터당 발 충돌 감소를 보였다.(2번 줄과 5번 줄)

다음으로, **ViNL(no-pretraining)** 을 **Rough**와 비교했을 때, ViNL(no-pretraining)은 약간 높은 성공률을 보였고, 미터당 발 충돌 수도 감소하였다. 그러나 이 방법은 **ViNL**에 비해 낮은 성공률과 높은 발 충돌 수를 보인다. 이는 제안된 접근 방식에서 pre-training의 중요함을 강조한다.

**ViNL(MLP)** 은 거친 지형에서 사전 학습을 거쳤고, 장애물 지형에서 미세 조정되었으며, 다리 접촉 페널티를 사용하였다. 이 방법은 ViNL과 거의 동일한 발 충돌 횟수를 보였지만, 성공률에서 ViNL이 +6.8% 더 높은 결과를 보였다. 이는 시계열 정보를 활용하는 *LSTM*의 이점을 강조한다. ViNL(MLP)은 로봇의 시야에서 벗어난 물체의 위치에 대한 정보를 갖고 있지 않기 때문에, 장애물을 넘는 동안 더 자주 걸려 넘어지는 경향을 보인다. 이로 인해 에피소드 실패가 더 많아졌다고 볼 수 있다.

## 5. Conclusion
이 논문은 egocentric vision 정보를 사용해 복잡한 실내 환경을 효과적으로 탐색할 수 있는 로봇을 위한 접근 방식을 제시하였다. 이전 연구들에서 불안정성에서 회복하거나, 장애물 주변으로 경로를 계획하는 데 초점을 맞춘 반면, 본 연구는 로봇의 다리를 정밀하게 제어하여 이러한 장애물을 피하는 *locomotion policy*를 제안한다. 이 연구는 *navigation policy*와 *locomotion policy*를 모두 학습하는 계층적 접근 방식을 제시하고 있으며, 두 모듈을 다른 시뮬레이터에서 별도로 학습한 후 추가적인 fine-tuning 없이 매끄럽게 통합이 가능함을 보여주고 있다.

## Comments
이 연구에서 제안하는 **ViNL**은 로봇이 복잡한 실내 환경에서 장애물을 피하면서 목표 지점까지 도달하는 접근 방식을 제시하고 있다. 현재 내가 관심을 갖고 있는 *Vision-and-Language Navigation (VLN)* 연구에서도 "로봇이 어떻게 효율적으로 장애물을 회피하고 목표 지점까지 안전하게 도달할 수 있는가"는 우선적으로 고려해야할 문제라고 생각한다. 특히, 이 프레임워크에서 활용된 시계열 모델인 *Long Short-Term Memory (LSTM)* 은 *VLN* 연구에서도 활용한다면, 텍스트 지시와 관련된 시각적 단서를 기억함으로써 더욱 정밀한 동작 계획과 수행이 가능한 시스템을 구축하는 데 매우 유용할 것이라고 생각한다. *(찾아보니, 실제로 LSTM을 최근 VLN 연구에서도 많이 활용하고 있었다.)* 이외에도 본 연구에서 다루고 있는 계층적 학습, 시뮬레이션 활용, 시각적 정보의 활용 등의 요소들을 향후 나의 연구에 적용해 볼 수 있기를 기대하며 글을 마친다.
---
layout: post
title: "Foundation Models of and for Navigation"
date: 2025-02-04
category:
  - Study
image: /assets/img/thumbnails/blog_1.png
author: Minho Lee
tags: 
  - ICRA2024
  - Workshop
  - Foundation Model
  - Robotics
  - Navigation
comments: true
toc: true
---

📌 ICRA 2024 Workshop에서 열린 Vision-Language Models for Navigation and Manipulation (VLMNM)를 주제로 한 invited talk 중에서 ViNT, GNM, NoMaD 논문의 저자인 Dhruv Shah 박사님의 <strong>"Foundation Models of and for Navigation"</strong> 발표를 정리해보려고 한다.

워크숍에 대한 정보와 전체 발표 영상은 아래 링크를 통해 확인할 수 있다. <br>
* <a href="https://vlmnm-workshop.github.io/"> ICRA-24 VLMNM Workshop</a>
* <a href="https://www.youtube.com/playlist?list=PLvYJV1Xnj1YVZdTrP2i8EqJW7Ai_pIMV7"> Recordings of Invited Talks</a>

### 1. Foundation Models
로봇 분야의 최근의 발전을 보면, 중요한 핵심은 대량의 데이터를 활용한 학습에 있다. 다양한 작업에서 수집된 데이터를 통해 모델이 작업의 패턴을 학습하고 이를 활용하여 여러 작업을 수행할 수 있도록 하는 것이다. 이러한 모델들을 <strong>"Foundation Models"</strong>라고 한다. 일반적으로 이러한 Foundation Model들은 광범위한 데이터셋을 사용하여 자가 지도 학습을 통해 학습되며, 최소한의 지도 학습만으로도 뛰어난 일반화 및 적응 능력을 갖는 것이 특징이다.

이러한 모델들은 "인터넷 기반 모델 (Internet Foundation Models)"이라고 할 수도 있는데, 그 이유는 이런 모델들은 능동적으로 데이터를 수집하는 것이 아니라 인터넷에 존재하는 다양한 데이터들을 활용해 훈련되기 때문이다. 이러한 데이터와 모델의 통합은 data-driven robotics에서 중요한 역할을 할 수 있다.

모델을 훈련할 때 단순히 데이터의 규모만으로 충분한 것은 아니다. 모델의 성능을 높이기 위해서는 데이터의 품질과 다양성에 집중하는 데이터 중심적 접근(Data-centric perspective)이 중요하다. 그러나 이러한 고품질의 그리고 다양한 데이터를 수집하는 것은 매우 어렵고 비용이 많이 드는 문제이다.

이러한 관점에서 Dhruv Shah 박사는 <strong>기존에 존재하는 데이터를 활용하여 데이터 기반 로보틱스를 어떻게 실현할 것인지</strong>에 집중해 왔다. 기존 데이터들은 서로 다른 환경과 다양한 작업에서 수집되어 겉보기에는 매우 이질적으로 보일 수 있다. 그러나 적절한 목표와 구조를 정의하면 이러한 데이터를 효과적으로 활용하여 로봇 간 전이 가능한 유용한 표현을 학습할 수 있음을 보여주었다.

<p align="center"><img src ="/assets/img/blog/20250204/fig_1.png"></p>

### 2. Robot Foundation Model
Robot Foundation Model이란 용어는 많은 의미를 포함하고 있으며, 사람들마다 각자 다른 방식으로 이해할 수도 있다. Dhruv Shah 박사는 연구에서 Robot Foundation Model을 한 번 훈련되면 추가적인 지도 학습 없이도 다양한 로봇에서 바로 사용할 수 있는 모델이라 정의하였다. 이는 서로 다른 센서를 가진 로봇, 전혀 다른 환경에 놓인 로봇에서도 새로운 작업을 수행할 수 있도록 설계된 모델을 의미한다. 이를 위해서 모델이 충족해야 하는 주요 조건은 아래와 같다:

<p align="center"><img src ="/assets/img/blog/20250204/fig_2.png"></p>

이러한 모델을 만들기 위해서는 기존의 개별적인 로봇별 학습 방식에서 벗어나, 여러 로봇에서 수집한 데이터를 통합하여 단일한 거대 신경망 모델을 학습하는 방식 (Cross-Embodiment Learning)이 요구된다.

### 3. Cross-Embodiment Learning
서로 다른 로봇 데이터를 하나의 모델에서 학습시키기 위해서는 특정한 설계 원칙이 필요하다. <br>
* <strong>공통된 행동 공간(action space) 정의</strong>: 각 로봇은 서로 다른 행동 공간(rotor velocities, joint angles ...)을 가진다. 따라서, 이를 통합할 수 있는 공통된 표현이 필요하다.
* <strong>로봇의 특성을 반영하는 프롬프트 (Embodiment Prompt)</strong>: 로봇마다 구조나 이동 방식이 다르기 때문에, 모든 로봇에 동일한 방식의 제어 모델을 적용하기 어렵다. 이를 해결하기 위해 로봇의 특성을 명시적인 시스템 파라미터로 직접 학습하는 대신 로봇이 과거에 수행한 행동 데이터를 활용하여 모델이 스스로 로봇의 특성을 학습하도록 하는 접근 방식이 요구된다. 다시 말해, Embodiment Prompt란 로봇의 과거 행동 데이터를 입력으로 제공하여 모델이 해당 로봇의 특성을 자동으로 반영하는 기법을 의미한다.

개별 로봇 데이터만 학습한 모델보다, 여러 로봇의 데이터를 통합해 학습한 모델(GNM, General Navigation Model)이 일관되게 높은 성능을 기록함을 보여주었다. 또한, 개별 로봇에서 학습한 정책보다 다수의 로봇 데이터를 결합한 정책이 더욱 일반화된 성능을 보이며, 모델의 크기가 커질 때에도 성능이 더 크게 향상됨을 보였다.

* <a href="https://ieeexplore.ieee.org/abstract/document/10161227?casa_token=UShbqLF5uogAAAAA:s3h6WmqvbkyN8E8lCbVogL2PXPrGoii7FftHRSKsFySo2NEW0aLQb5dF1A2OLe2KZIligpBP"> Shah, Dhruv, et al. "Gnm: A general navigation model to drive any robot." 2023 IEEE International Conference on Robotics and Automation (ICRA). IEEE, 2023.</a>
<p align="center"><img src ="/assets/img/blog/20250204/fig_3.gif"></p>

### 4. Downstream Adaptation
<p align="center"><img src ="/assets/img/blog/20250204/fig_4.png"></p>

제안하고 있는 Robot Foundation Model을 기반으로 더 고차원의 기능을 추가하기 위한 연구가 진행되고 있다. 주요 연구 방향은 크게 세 가지로 나뉘는데, 기본적으로 학습된 모델은 단순히 목표 지점에 도달하고 충돌을 회피하는 기능만을 수행한다면 기존 모델에 약간의 추가 데이터와 적절한 보상 함수를 사용해 학습시켜 로봇이 사회적 상호작용이 이루어지는 환경에서 더 나은 행동을 학습할 수 있도록 하는 연구가 진행되고 있다.

두 번째로 기존의 모델이 이미지 기반 목표를 사용했다면, 텍스트 기반 목표 또는 GPS 기반 목표를 적용하는 것이 더 유용할 수 있다고 판단하여 연구가 수행되고 있다. 그러나, 텍스트 기반 목표를 적용하려면 모든 데이터를 텍스트로 annotation해야 하는 문제가 있다. 이를 해결하기 위해 이미지 기반 정책을 학습한 후, GPS 좌표나 명령어를 인코딩하는 새로운 모듈을 추가하는 방식을 사용했다고 한다. 이 방식은 GPT 모델에서 소프트 프롬프팅(Soft Prompting)을 사용하는 방식과 유사하며, 향후 연구에서는 In-context Prompting 방식을 활용해 더욱 효과적인 목표 설정을 탐구할 예정이라고 한다.

마지막으로 대규모의 로봇 데이터셋 구축(Open Cross-Embodiment Collaboration)이 진행 중이다. 연구 결과에서 더 다양한 로봇 데이터를 통합할 수록 Positive Transfer 현상이 발생함을 확인하였으며, 특히 manipulation과 navigation tasks를 함께 정책으로 학습할 경우에 두 작업 모두에서 성능이 개선되는 효과가 나타났다. 이러한 연구를 바탕으로 더 다양한 로봇과 작업 데이터를 동시에 학습하는 하나의 통합된 모델로의 확장 연구가 진행되고 있다고 한다.


### 5. Conclusion
발표 내용을 요약하면 다음과 같다:

- 로봇 분야에서 <u>Foundation Models</u>은 일반적으로 대량의 데이터를 활용하여 로봇이 다양한 작업의 패턴을 학습하고, 이를 통해 여러 작업을 수행 가능하도록 하는 모델을 의미한다.
- 모델을 훈련할 때 데이터의 규모만으로 충분하지 않으며, 데이터의 품질과 다양성에 집중하는 <u>Data-centric Perspective</u>가 중요하다. 하지만, 고품질 데이터를 확보하는 것은 높은 비용과 어려움이 따르는 문제이다.
- Dhruv Shah 박사는 연구에서 <u>Robot Foundation Model</u>을 추가적인 지도 학습 없이도 다양한 로봇에서 바로 사용 가능하며, 이종 센서나 새로운 환경에서도 적용될 수 있도록 설계된 모델이라 정의했다.
- 위의 모델을 설계하기 위해서는 <u>Cross-Embodiment Learning</u>의 개념이 필요하며, 이는 서로 다른 로봇의 행동 공간을 통합하고 로봇의 특성을 반영하는 프롬프트를 활용하여 모델을 학습하는 방식이다. 이를 통해 개별적인 학습보다, 보다 일관적이고 일반화된 성능을 보인다는 것이 실험적으로 입증되었다.
- 향후 연구에서는 기존 모델에 <u>사회적 보상함수 추가</u>, <u>새로운 모달리티(텍스트/GPS 기반 목표) 적용</u>, <u>더 많은 로봇 모델 및 작업을 학습한 통합 모델로의 확장</u>의 방향이 진행될 것이다.

끝으로, 앞서 언급된 모델 및 관련 프레임워크는 아래 GitHub에서 확인할 수 있다.
* <a href="https://github.com/robodhruv/visualnav-transformer"> Berkeley AI Research & Dhruv Shah Github</a>

내가 연구하는 필드에서 Dhruv Shah 박사님의 연구들은 큰 주목을 받았었고, 현재도 많은 관심을 받고 있다. 앞으로도 의미 있는 연구들이 나올 것으로 기대되기 때문에, 계속해서 살펴볼 예정이다.
그리고 GitHub에 공개된 사전 학습된 모델을 연구실의 로봇 플랫폼에 적용하고 다양한 모듈과 연계하여 실험해 볼 계획인데, 아주 흥미로운 작업이 될 것 같다!

---
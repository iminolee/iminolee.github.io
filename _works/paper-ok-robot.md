---
title: "[ICRAW 2024] OK-Robot: What Really Matters in Integrating Open-Knowledge Models for Robotics"
category: paper
category_slug: paper
type: content
image: assets/img/works/paper/ok-robot/thumb.png
button_url: https://arxiv.org/pdf/2401.12202
---

| **Author** | Peiqi Liu*, Yaswanth Orru*, Jay Vakil, Chris Paxton, Nur Muhammad Mahi Shafiullah, Lerrel Pinto  |
| **Venue** | [First Workshop on Vision-Language Models for Navigation and Manipulation at ICRA 2024](https://openreview.net/group?id=IEEE.org/2024/ICRA/Workshop/VLMNM#tab-accept)  |
| **Code** | [GitHub Repository](https://github.com/ok-robot/ok-robot)  |

### 1. Method

#### A. Object-centric Sematinc Memory (VoxelMap)

<p align="center"><img src ="/assets/img/works/paper/ok-robot/figure1.png"></p>

본 논문에서 제안하는 open-vocabulary object navigation 방식은 [**CLIP-Fields**](https://arxiv.org/pdf/2210.05663)의 접근법을 따르며, 사전 매핑 단계(pre-mapping phase)에서 환경을 스캔하는 과정을 포함한다. 이 과정에서 수집된 RGB-D 이미지, 카메라의 자세(pose) 및 위치(position) 정보가 맵 구축 모듈로 입력된다.

모든 프레임에서 open-vocabulary object detector를 실행하여 객체의 바운딩 박스를 검출하고, 이를 **Segment Anything Model (SAM)** 을 사용하여 객체 마스크로 변환한다. 이때 객체 탐지를 위한 자연어 쿼리(object queries) 입력은 **Scannet200 데이터셋**의 라벨을 기반으로 구축된 대규모 객체 쿼리 세트를 활용한다. 이와 함께 CLIP 임베딩을 생성한다.

<p align="center"><img src ="/assets/img/works/paper/ok-robot/figure2.png"></p>

이후, Depth 이미지와 카메라의 포즈 정보를 사용하여 객체 마스크를 실제 월드 좌표계(real-world coordinate)로 역투영(back-projecting)한다. 그 결과, CLIP semantic vector와 연관된 포인트 클라우드가 생성된다. 생성된 포인트 클라우드는 일정 해상도로 복셀화(voxelize)되며, 각 voxel 내의 CLIP 임베딩 값을 검출 신뢰도(detector confidence)기반 가중 평균(weighted average)으로 계산하여 최종 **VoxelMap**을 구축한다. 이 **VoxelMap**은 객체 메모리(object memory)의 기반을 형성하며, 객체 탐색 및 내비게이션을 위한 핵심 요소로 활용된다.

#### B. Querying the memory module

<p align="center"><img src ="/assets/img/works/paper/ok-robot/figure3.png"></p>

주어진 언어 쿼리(language query)는 CLIP 언어 인코더를 사용하여 semantic vector로 변환된다. 변환된 semantic vector와 각 복셀에 저장된 semantic vector 간 내적(dot product)를 계산하여, 내적 값이 최대가 되는 복셀을 찾는다. 이 과정은 <u>쿼리한 객체가 존재할 가능성이 가장 높은 위치를 찾는 것</u>을 의미한다.

언어 쿼리를 처리할 때 "A on B" 형태의 관계형 쿼리를 "A near B"로 해석하는 접근 방식을 채택하고 있다. 이 과정에서는:
* 쿼리 A에 대해 가장 가능성이 높은 10개의 복셀을 선택하고 
* 쿼리 B에 대해 가장 가능성이 높은 50개의 복셀을 선택한 뒤, 
* 이 10×50개의 복셀 간 L2 거리(pairwise L2 distance)를 계산, 가장 짧은 (A,B) 거리 값을 갖는 A 위치를 최종 선택한다.

이러한 방식은 **기존 연구들에 비해 더 낮은 해상도의 맵으로도 객체 검색이 가능**하고, **맵을 생성한 후 객체가 약간 이동하더라도 대응할 수 있다**는 장점을 갖는다.

#### C. Navigating to objects in the real world

구축된 **VoxelMap**을 **2D Obstacle Map**으로 변환하며, 이 과정에서 바닥과 천장의 높이를 설정한다. 그 다음 바닥과 천장 사이에 차지된 복셀 부피가 있으면 해당 그리드 셀은 장애물이 있는 것으로 간주하여 "non-navigable"로 표시한다. 또한, 바닥과 천장에 해당하는 복셀이 없는 경우에는 해당 그리드 셀은 "unexplored"로 간주되며 역시 "non-navigable"로 표시한다. 추가적으로, 로봇의 크기와 회전 반경을 고려하여 각 차지된 포인트 주변의 20cm 반경 내의 모든 포인트도 "non-navigable"로 표시한다.

<p align="center"><img src ="/assets/img/works/paper/ok-robot/figure4.png"></p>

A* 알고리즘을 사용하여 전역 경로(Global Path)와 지역 경로(Local Path)를 탐색하는데, 이를 위해 세 가지 주요 목표를 균형 있게 고려해야 한다.
1. 로봇은 물체를 조작할 수 있을 만큼 충분히 가까워야함
2. 로봇의 그리퍼가 물체를 조작할 수 있도록 물체와 로봇 사이에 일정한 공간이 필요함
3. 조작 중 충돌을 피하기 위해 로봇은 장애물과의 거리를 적절히 유지해야 함

위의 세 가지 목표와 관련된 서로 다른 navigation score function를 평가하여 가장 적절한 도착 위치를 찾는다. A* 알고리즘을 통해 경로를 탐색할 때 세 함수의 가중합으로 정의된 *s*를 고려하여 전역 경로(Global Path)가 계획이 되고, 이 전역 경로를 따라 이동하는 과정에서 장애물로부터 더 멀리 떨어지도록 하는 *s3* 함수를 휴리스틱 함수로 사용하여 동적으로 지역 경로(Local Path)가 업데이트 된다.

### 2. Experiments

<p align="center"><img src ="/assets/img/works/paper/ok-robot/figure5.png"></p>

<p align="center"><img src ="/assets/img/works/paper/ok-robot/ablation.png"></p>

논문의 실험 결과에서 제안된 **VoxelMap** 기반 방법이 다른 Semantic Memory 모듈과 비교하여 가장 높은 성능을 보였으며, 다양한 환경에서 적은 성능 편차(error variance)를 보이는 것을 통해 일관되게 좋은 성능을 유지함을 확인할 수 있다.

### 3. Limitations & Requested for Research

본 논문에서 제안한 **VoxelMap** 기반 open-vocabulary object navigation 방법은 다음과 같은 한계를 갖는다.

* **동적 환경에 대한 대응 부족**: 현재의 Semantic Memory 모듈과 Obstacle Map Builder는 환경을 정적(static)으로 표현하므로, 환경 변화에 대해 효율적으로 업데이트하는 방법이 부족하다.

* **시멘틱 쿼리의 애매성(ambiguity)**: 내비게이션 과정에서 Semantic Query가 애매한 경우, 의도한 객체를 Semantic Memory에서 정확히 검색하지 못하는 문제가 발생한다. 이를 해결하기 위해 사용자와의 상호작용을 통한 쿼리 명확화(query disambiguate) 과정이 필요할 수 있다.

* **모듈 간 오류 누적(multiplicative error accumulation)**: 시스템 내 개별 모듈의 오류가 누적될 경우 전체 프로세스가 실패할 가능성이 높아진다. 이를 방지하기 위해 더 정교한 오류 감지(error detection) 및 복구(retry) 알고리즘의 도입이 필요하다.

---
---
layout: post
title: "Isaac Lab Tutorial #2 - Interacting with Objects"
date: 2025-02-23
category:
  - Simulation
image: /assets/img/thumbnails/blog_3.png
author: Minho Lee
tags: 
  - Robotics
  - Isaac Lab
  - Isaac Sim
comments: true
toc: true
---

| Previous Tutorial | Description |
|------------|------------------------------------------------------------------------------------------------|
| 🔗 [Isaac Lab Tutorial #1](https://www.roboxiv.com/simulation/2025/02/08/isaaclab-tutorial-intro/) | Overview of Isaac Lab & Setting up the Docker Environment |


📌 두 번째 튜토리얼에서는 Standalone Python 스크립트를 사용하여 Isaac Sim 시뮬레이터를 실행하고, Isaac Lab의 장면에 다양한 객체 또는 프리미티브를 불러오는 방법을 살펴본다.

### 1. Creating an empty scene

#### 1.1. Launching the simulator

#### 1.2. Importing python modules

#### 1.3. Configuring the simulation context

#### 1.4. Running the simulation

#### 1.5. Exiting the simulation

### 2. Spawning prims into the scene

#### 2.1. Key concepts of USD
Omniverse의 장면 디자인은 USD(Universal Scene Description)라는 파일 형식을 중심으로 구축된다. USD 파일을 통해 계층적 방식으로 3D 장면을 구성할 수 있으며, USD에 대한 자세한 설명은 아래 페이지를 통해 확인해보는 것을 추천한다. 

* <a href="https://openusd.org/docs/index.html"> Introduction to USD</a>

본 튜토리얼을 이해하기 위해 꼭 알아야 할 USD의 개념은 다음과 같다.<br>
1. <b>Primitive (Prim)</b>: USD 장면의 기본 구성 요소로 장면 그래프의 노드(node)라고 볼 수 있다. 각 노드는 메시(mesh), 광원(light), 카메라(camera) 또는 변환(transform)일 수 있다. 
2. <b>Attribute</b>: Prim이 가지는 속성(attribute)으로 키-값(key-value) 쌍의 형태를 띤다. 예를 들어, 특정 prim이 <i>color</i>라는 속성을 가지며, 그 값이 <i>red</i>일 수도 있다.
3. <b>Relationship</b>: Prim 간의 연결을 나타낸다. 이는 다른 prim을 참조하는 포인터와 같은 개념으로, 예를 들어 메시(mesh) prim이 shading을 위한 material prim과 관계를 가질 수 있다.
4. <b>Stage</b>: 프리미티브(prim)와 속성(attribute) 및 관계(relationship)의 모음을 USD 스테이지(stage)라고 한다. 이는 장면의 모든 prim을 위한 컨테이너로 생각할 수 있다. 즉, USD 스테이지는 장면을 구성하는 모든 prim을 관리하는 공간이며, 장면을 디자인한다는 것은 곧 USD stage를 구성하는 것과 같다.

Isaac Lab에서는 USD API를 기반으로 configuration-driven 인터페이스를 제공하여 장면에 prim을 생성할 수 있다. 이는 <mark>sim.spawners</mark> 모듈에 포함되어 있다.

* <a href="https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.sim.spawners.html"> Isaac Lab API Reference: isaaclab.sim.spawners</a>

장면에 prim을 생성할 때 각 prim에는 속성과 관계를 정의하는 구성 클래스 인스턴스가 필요하다. 이 구성 클래스가 해당 함수에 전달되어 prim 이름과 변환이 지정된다. 그런 다음 함수가 장면에 prim을 생성한다.

> ⚠️ **[Attention]** <br>
모든 장면 구성은 시뮬레이션이 시작되기 전에 이루어져야 한다. 시뮬레이션이 시작되면 장면을 고정하고 prim의 속성만 변경하는 것이 좋다. 시뮬레이션 중 새로운 prim을 추가하면 GPU 물리 시뮬레이션 버퍼가 변경되어 예상치 못한 동작이 발생할 수 있기 때문이다.

#### 2.2. Reference Script Guide


이 튜토리얼과 관련된 더 자세한 내용은 아래 링크를 통해 확인할 수 있다.
* <a href="https://isaac-sim.github.io/IsaacLab/main/source/tutorials/00_sim/spawn_prims.html"> Isaac Lab Documentation: Spawning prims into the scene</a>

---
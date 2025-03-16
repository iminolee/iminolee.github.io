---
layout: post
title: "Isaac Lab Tutorial #2 - Create the Scene"
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
NVIDIA Omniverse의 기본적인 워크플로우는 크게 세 가지로 나뉜다.
* GUI: 직관적인 시각적 인터페이스를 통해 가상 세계를 구성하고 로봇 조립 및 센서 부착 등의 작업을 수행하는 데 적합하다.
* Extensions: 비동기 실행 및 실시간 변경을 지원하며, 적응형 물리 엔진을 활용해 인터랙티브 GUI와 실시간 애플리케이션 구현하는 데 유용하다.
* <b>Standalone Python</b>: 물리 및 렌더링 타이밍을 직접 제어할 수 있으며, headless 모드로 대규모 강화학습 및 동적 환경 생성을 효율적으로 수행할 수 있다.

본 섹션에서는 독립 실행형 파이썬(Standalone Python) 스크립트를 사용하여 시뮬레이션 애플리케이션을 시작하고 빈 장면을 생성하는 기본적인 방법을 소개한다. 마지막에는 <mark>argparse.ArgumentParser</mark>에 명령줄 옵션을 추가하는 방법을 설명하며, <mark>AppLauncher.add_app_launcher_args()</mark> 메서드에 파서 인스턴스를 전달하여 다양한 매개변수를 추가하는 과정을 다룬다. 여기에는 headless 모드 설정, 다양한 Livestream(해상도 및 출력) 조정, 오프스크린 렌더링 활성화 등 사용자 정의 시뮬레이션을 설정하는 옵션이 포함된다.

#### 1.1. Launching the simulator

```python
import argparse
from isaaclab.app import AppLauncher

# create argparser
parser = argparse.ArgumentParser(description="Creating an empty stage.")
# append AppLauncher CLI args
AppLauncher.add_app_launcher_args(parser)
# parse the arguments
args_cli = parser.parse_args()
# launch omniverse app
app_launcher = AppLauncher(args_cli)
simulation_app = app_launcher.app
```
<mark>isaaclab.app</mark> 모듈의 <mark>AppLauncher</mark> 클래스는 Omniverse Isaac Sim 애플리케이션을 실행하는 역할을 한다. <mark>AppLauncher.add_app_launcher_args(parser)</mark>를 호출하면 애플리케이션 실행에 필요한 기본적인 CLI 인자가 <mark>parser</mark>에 추가된다. 이후 <mark>AppLauncher(args_cli)</mark>를 호출하면 애플리케이션 실행 객체가 생성되며, <mark>app_launcher.app</mark>을 통해 시뮬레이션 애플리케이션 객체(<mark>simulation_app</mark>)를 가져올 수 있다.

#### 1.2. Importing python modules
시뮬레이션 앱이 실행되면 Isaac Sim 및 기타 라이브러리에서 다양한 Python 모듈을 가져올 수 있다. 아래에서는 다음 모듈을 가져오는 예시다.
* <mark>isaaclab.sim</mark>: 시뮬레이터와 관련된 모든 핵심 작업을 위한 Isaac Lab의 하위 패키지

```python
from isaaclab.sim import SimulationCfg, SimulationContext
```

#### 1.3. Configuring the simulation context
독립 실행형 스크립트에서 시뮬레이션을 시작할 때 <mark>sim.SimulationContext</mark> 클래스를 통해 시뮬레이터 재생, 일시 정지 및 단계별 실행 제어뿐만 아니라 다양한 타임라인 이벤트 처리 및 물리 장면 구성을 수행할 수 있다. 아래 스크립트에서는 물리 및 렌더링의 시간 간격(time step)을 0.01초로 설정하고 있으며, 이를 위해 해당 값을 <mark>sim.SimulationCfg</mark>에 전달한 후 시뮬레이션 컨텍스트의 인스턴스를 생성한다.

```python
# Initialize the simulation context
sim_cfg = SimulationCfg(dt=0.01)
sim = SimulationContext(sim_cfg)
# Set main camera
sim.set_camera_view([2.5, 2.5, 2.5], [0.0, 0.0, 0.0])
```
위 예제에서는 시뮬레이션 컨텍스트를 생성한 후, 시뮬레이션된 장면에서 작동하는 물리 엔진(physics)만 설정하였다. 여기에는 시뮬레이션에 사용할 장치(device), 중력 벡터(gravity vector), 기타 고급 솔버 매개변수(advanced solver parameters) 등이 포함된다. 이제 시뮬레이션 실행을 위해 다음 두 가지 주요 단계가 남아있다.
* <b>시뮬레이션 장면 설계</b>: 센서, 로봇 및 기타 시뮬레이션 객체 추가
* <b>시뮬레이션 루프 실행</b>: 시뮬레이터 단계별 실행 및 데이터 설정 및 가져오기

이번 튜토리얼에서는 <b>빈 장면에서의 시뮬레이션 제어(2단계)</b>만 다룬다. 다음 튜토리얼에서는 <b>객체 추가 및 상호 작용을 위한 시뮬레이션 핸들 작업(1단계)</b>을 살펴볼 예정이다.

#### 1.4. Running the simulation
시뮬레이션 장면을 설정한 후 가장 먼저 해야 할 일은 <mark>sim.SimulationContext.reset()</mark> 메서드를 호출하는 것이다. 이 메서드는 타임라인을 시작하고, 시뮬레이터에서 물리 핸들(physics handles)을 초기화하는 역할을 한다. 시뮬레이션을 실행하기 전에 반드시 처음에 호출해야 하며, 그렇지 않으면 시뮬레이션 핸들이 올바르게 초기화되지 않는다.

아래 코드 스니펫은 시뮬레이션 타임라인을 시작한 후, 애플리케이션이 실행되는 동안 반복적으로 시뮬레이션 단계를 진행하는 간단한 시뮬레이션 루프를 설정하는 예제이다. <mark>sim.SimulationContext.step()</mark> 메서드는 <mark>render</mark> 인자를 받아 실행되며, 이 값에 따라 렌더링 관련 이벤트를 포함할지 여부가 결정된다. 기본적으로 <mark>render</mark> 값은 <mark>True</mark>로 설정되어 있다.

```python
# Play the simulator
sim.reset()
print("[INFO]: Simulator Setup complete...")

# Simulate physics
while simulation_app.is_running():
    # perform step
    sim.step()
```

#### 1.5. Exiting the simulation
마지막으로, <mark>isaacsim.SimulationApp.close()</mark> 메서드를 호출하여 시뮬레이션을 중지하고 해당 창을 닫을 수 있다.

```python
# close sim app
simulation_app.close()
```

### 1.6. Test Code Execution
이제 아래 명령어를 통해 스크립트를 실행하면 시뮬레이션이 시작되고 스테이지가 렌더링되는 결과를 확인할 수 있다.
시뮬레이션을 중지하려면 창을 닫거나 터미널에서 <mark>Ctrl+C</mark>을 입력하면 된다.

```bash
$ ./isaaclab.sh -p scripts/tutorials/00_sim/create_empty.py
```

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

개략적으로 보면 작동 방식은 아래와 같다.
1. 설정 클래스 인스턴스 생성
2. 해당 spawner 함수를 사용하여 prim을 장면에 배치하거나 설정 클래스에서 직접 spawner 함수를 호출

```python
# Create a configuration class instance
cfg = MyPrimCfg()
prim_path = "/path/to/prim"

# Spawn the prim into the scene using the corresponding spawner function
spawn_my_prim(prim_path, cfg, translation=[0, 0, 0], orientation=[1, 0, 0, 0], scale=[1, 1, 1])
# OR
# Use the spawner function directly from the configuration class
cfg.func(prim_path, cfg, translation=[0, 0, 0], orientation=[1, 0, 0, 0], scale=[1, 1, 1])
```

> ⚠️ **[Attention]** <br>
모든 장면 구성은 시뮬레이션이 시작되기 전에 이루어져야 한다. 시뮬레이션이 시작되면 장면을 고정하고 prim의 속성만 변경하는 것이 좋다. 시뮬레이션 중 새로운 prim을 추가하면 GPU 물리 시뮬레이션 버퍼가 변경되어 예상치 못한 동작이 발생할 수 있기 때문이다.

#### 2.2. Reference Script Guide
이제 장면에 지면 평면(Ground Plane)과 조명, 큐브와 구, 실린더와 같은 기본 형상 및 다른 .USD, .URDF 또는 .OBJ 파일과 같은 다른 파일 형식에서 prim을 스폰하는 방법을 간단한 예시를 통해 알아보자.

1. 지면 평면 생성: <mark>GroundPlaneCfg</mark> 모양과 크기 등의 변경 가능한 속성을 갖는 격자 모양의 접지면을 구성한다.
```python
# Spawn ground-plane
cfg_ground = sim_utils.GroundPlaneCfg()
cfg_ground.func("/World/defaultGroundPlane", cfg_ground)
```

2. 조명 생성: 장면에 조명 설정을 불러온다. 여기에는 원거리 조명(distant lights), 구형 조명(sphere lights), 디스크 조명(disk lights), 원통형 조명(cylinder lights)이 포함된다.
```python
# Spawn distant light
cfg_light_distant = sim_utils.DistantLightCfg(
    intensity=3000.0,
    color=(0.75, 0.75, 0.75),
)
cfg_light_distant.func("/World/lightDistant", cfg_light_distant, translation=(1, 0, 10))
```

3. 기본 형상 생성: 
* 기본 형상을 생성하기 전에, <b>Transform Primitive(Xform)</b>의 개념을 이해해야 한다. Xform은 변환(transformation) 속성만 포함하는 기본 요소(primitive)로 다른 prim을 그 아래에 그룹화(parent-child 관계)하고 그룹 전체를 변형하는 데 사용된다. 아래는 Xform Prim을 생성하여 하위에 있는 모든 기본 형상을 그룹화하는 예제이다.

```python
# Create a new xform prim for all objects to be spawned under
prim_utils.create_prim("/World/Objects", "Xform")
```

다음으로, <mark>ConeCfg</mark> 클래스를 사용하여 원뿔의 반경, 높이, <b>[2] 물리 속성 및 재료 속성을 지정</b>하여 원뿔 형상을 생성하는 예시는 아래와 같다. (<b>[1] 기본적으로 물리 및 재료 속성은 비활성화</b>된다.)

```python
# Spawn a red cone (without physics and material properties)
cfg_cone = sim_utils.ConeCfg(
    radius=0.15,
    height=0.5,
    visual_material=sim_utils.PreviewSurfaceCfg(diffuse_color=(1.0, 0.0, 0.0)),
)
cfg_cone.func("/World/Objects/Cone1", cfg_cone, translation=(-1.0, 1.0, 1.0))
cfg_cone.func("/World/Objects/Cone2", cfg_cone, translation=(-1.0, -1.0, 1.0))
```

아래는 강체 물리(Rigid body physics)를 추가하여 원뿔의 질량, 마찰 및 복원력을 지정하는 예시다. 만약 이를 따로 지정하지 않으면 USD Physics의 기본값이 적용된다.

```python
# Spawn a green cone with colliders and rigid body (with rigid body physics)
cfg_cone_rigid = sim_utils.ConeCfg(
    radius=0.15,
    height=0.5,
    rigid_props=sim_utils.RigidBodyPropertiesCfg(),
    mass_props=sim_utils.MassPropertiesCfg(mass=1.0),
    collision_props=sim_utils.CollisionPropertiesCfg(),
    visual_material=sim_utils.PreviewSurfaceCfg(diffuse_color=(0.0, 1.0, 0.0)),
)
cfg_cone_rigid.func(
    "/World/Objects/ConeRigid", cfg_cone_rigid, translation=(-0.2, 0.0, 2.0), orientation=(0.5, 0.0, 0.5, 0.0)
)
```

마지막으로, 변형 가능한 물리 속성(Deformable body physics)을 포함하는 직육면체를 생성하는 예시다. 강체 시뮬레이션과 달리 변형 가능한 객체는 각 꼭짓점(vertex) 사이에 상대적인 움직임이 가능하다. 이는 천, 고무 또는 젤리와 같은 부드러운 재질을 갖는 객체를 시뮬레이션하는 데 유용하다. 변형 가능한 물리 속성은 GPU 시뮬레이션에서만 지원된다.

```python
# Spawn a blue cuboid with deformable body
cfg_cuboid_deformable = sim_utils.MeshCuboidCfg(
    size=(0.2, 0.5, 0.2),
    deformable_props=sim_utils.DeformableBodyPropertiesCfg(),
    visual_material=sim_utils.PreviewSurfaceCfg(diffuse_color=(0.0, 0.0, 1.0)),
    physics_material=sim_utils.DeformableBodyMaterialCfg(),
)
cfg_cuboid_deformable.func("/World/Objects/CuboidDeformable", cfg_cuboid_deformable, translation=(0.15, 0.0, 2.0))
```

4. 다른 파일에서 불러오기: USD, URDF 또는 OBJ 파일과 같은 다른 파일 형식에서 객체를 불러올 수 있다. 아래 예시에서는 테이블의 USD 파일을 장면에 추가한다. 테이블은 mesh primitive이며, 연관된 material primitive가 있다. 이 모든 정보는 USD 파일에 저장되어 있다.

```python
# Spawn a usd file of a table into the scene
cfg = sim_utils.UsdFileCfg(usd_path=f"{ISAAC_NUCLEUS_DIR}/Props/Mounts/SeattleLabTable/table_instanceable.usd")
cfg.func("/World/Objects/Table", cfg, translation=(0.0, 0.0, 1.05))
```

이 튜토리얼과 관련된 보다 자세한 내용은 아래 링크에서 확인할 수 있다.
* <a href="https://isaac-sim.github.io/IsaacLab/main/source/tutorials/00_sim/spawn_prims.html"> Isaac Lab Documentation: Spawning prims into the scene</a>

#### 2.3. Executing the Script
아래 명령어를 통해 위에서 설명한 장면 구성을 하는 예제 스크립트를 실행할 수 있다.

```bash
$ ./isaaclab.sh -p scripts/tutorials/00_sim/spawn_prims.py
```

시뮬레이션이 시작되면 아래 영상과 같이 지면 평면, 조명, 원뿔 객체, 테이블이 나타난 화면이 보여진다. 강체 물리가 활성화된 녹색 원뿔은 떨어져서 테이블과 지면 평면과 충돌해야 하며, 다른 빨간색 원뿔은 시각적 요소이므로 움직이지 않는 것을 볼 수 있다.

<video width="640" height="360" controls>
  <source src="/assets/img/blog/20250223/isaaclab_spawn_prim_demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

이번 포스팅에서는 <b>기본적인 시뮬레이션 장면 구성 방법</b>을 살펴보았다. 다음 포스팅에서는 <b>강체(rigid object), 관절(articulation), 변형 가능한 객체(deformable object)를 불러오고 상호작용</b> 하는 방법에 대해 알아보도록 하자!

---
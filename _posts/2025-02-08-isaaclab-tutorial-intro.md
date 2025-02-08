---
layout: post
title: "Isaac Lab Tutorial #1 - Introduction"
date: 2025-02-08
category:
  - Simulation
image: /assets/img/thumbnails/blog_2.png
author: Minho Lee
tags: 
  - Robotics
  - Isaac Lab
  - Isaac Sim
comments: true
toc: true
---

📌 최근 로봇 연구에서 <strong>로보틱스 시뮬레이션</strong>은 모델 성능 검증, 학습 워크플로우 구축, 대규모 합성데이터 생성 등 다양한 목적으로 활용되고 있다. 로봇 학습 및 시뮬레이션을 위한 여러 플랫폼이 존재하는 가운데, NVIDIA의 Isaac Lab은 GPU 가속, CUDA, TensorRT 등의 딥러닝 SDK와 Omniverse Isaac Sim을 기반으로 고품질 그래픽과 실시간 레이트레이싱을 지원하는 가상환경을 제공한다. 이러한 특징들은 Embodied AI를 연구하는 연구자들에게 강력하고 효과적인 솔루션이 될 수 있다.

나 역시도 Isaac Lab이 제공하는 기능들을 공부하는 것은 향후 연구에 많은 도움이 될 것이라 생각이 들었다. 그래서, <strong>Isaac Lab의 기본적인 튜토리얼부터 토이 프로젝트를 거쳐 실제 연구 적용까지의 과정을 정리하여 포스팅하려고 한다.</strong>

### 1. Before Getting Started
#### 1.1. NGC(NVIDIA GPU Cloud) Container
<p align="center"><img src ="/assets/img/blog/20250208/fig_1.png"></p>

NGC에서는 AI, 머신 러닝 및 고성능 컴퓨팅(HPC)을 위한 컨테이너을 제공하며, 사전 학습된 AI 모델과 시각화 도구를 포함한 소프트웨어 스택을 지원한다. AI 모델 구축은 복잡하고 많은 시간이 소요되는데 NGC의 통합 컨테이너 기술은 배포 워크플로우를 간소화하고 AI 프로젝트를 가속화하는 데 도움을 준다.

또한, NGC는 AI 모델 개발에 최적화된 Docker 이미지를 제공하며 TensorFlow, PyTorch 등 주요 프레임워크를 폭넓게 지원하여 GPU 관련 호환성 문제를 해결할 수 있다.

더 자세한 내용은 <a href="https://catalog.ngc.nvidia.com/containers"> NVIDIA NGC Catalog</a>에서 확인할 수 있으며, 안에서 <a href="https://catalog.ngc.nvidia.com/orgs/nvidia/containers/isaac-lab"> NVIDIA Isaac Lab</a>과 관련된 정보도 확인할 수 있다.

### 2. Build the Isaac Lab container

Issac Lab 도커 이미지를 빌드하기 위해, Dockerfile과 docker-compose.yml, .env 파일을 포함하는 Root GitHub Repository를 클론한다.

```
$ git clone https://github.com/isaac-sim/IsaacLab.git
```
클론해온 <mark>IsaacLab</mark> 저장소의 <mark>./docker</mark> 디렉토리에는 Docker 및 ROS2 개발 환경을 구축하고 실행하는 데 필요한 파일들이 위치해 있다.

<mark>container.py</mark>와 <mark>container.sh</mark>은 Isaac Lab의 Docker 컨테이너를 쉽게 실행하고 관리할 수 있도록 만들어진 Python 및 Bash 스크립트이다. 이를 활용하여 Docker 이미지를 빌드하고 컨테이너를 실행해보자.

```
$ cd IsaacLab
$ python3 docker/container.py start
```

<mark>container.py</mark>는 Docker 컨테이너를 관리하기 위한 CLI 명령어를 제공한다. 주요 명령어는 다음과 같다.
- <mark>start</mark>: 도커 이미지를 빌드하고 컨테이너를 백그라운드에서 실행
- <mark>enter</mark>: 실행 중인 Isaac Lab 컨테이너에 새 bash 프로세스로 진입
- <mark>config</mark>: 제공된 YAML 및 환경 파일을 기반으로 docker-compose.yaml 생성 또는 출력
- <mark>copy</mark>: 컨테이너에서 빌드 결과물 및 로그 파일을 호스트로 복사
- <mark>stop</mark>: 도커 컨테이너를 종료하고 삭제

이미지 빌드가 완료되면 아래 명령어를 통해 컨테이너를 실행할 수 있다. 
```
$ python3 docker/container.py enter
```

이 환경에서는 Isaac Lab 저장소의 사본을 포함하며, Isaac Sim의 디렉토리와 라이브러리에 대한 액세스가 가능하다. 또한, 컨테이너에서는 호스트 장치의 <mark>IsaacLab</mark> 디렉토리가 마운트되어 있기 때문에 호스트에서 해당 디렉토리 아래의 파일을 수정하면 Docker 이미지를 다시 빌드할 필요 없이 변경 사항이 컨테이너에 즉시 반영된다.

### 3. Training with an RL Agent

이제 컨테이너 안에서 Isaac Lab 루트 디렉토리에 포함된 PPO 에이전트를 Stable-Baseline3를 사용하여 Cartpole balancing task를 해결하는 강화학습 예제를 실행해보고 결과를 확인해보자.

이 튜토리얼과 관련된 더 자세한 내용은 아래 링크를 통해 확인할 수 있다.
* <a href="https://isaac-sim.github.io/IsaacLab/main/source/tutorials/03_envs/run_rl_training.html"> Isaac Lab Documentation: Training with an RL Agent</a>

```
$ ./isaaclab.sh -p scripts/reinforcement_learning/sb3/train.py --task Isaac-Cartpole-v0 --num_envs 64 --headless --video --max_iterations 2000

[INFO]: Time taken for scene creation : 1.056965 seconds
[INFO]: Scene manager:  <class InteractiveScene>
	Number of environments: 64
	Environment spacing   : 4.0
	Source prim name      : /World/envs/env_0
	Global prim paths     : []
	Replicate physics     : True
[INFO]: Starting the simulation. This may take a few seconds. Please wait...
```

에이전트를 훈련하는 세 가지 주요 방법이 있고, 플래그를 통해 설정할 수 있다.

1. Headless execution: <mark>--headless</mark> 플래그가 설정되면 시뮬레이션은 훈련 중에 렌더링되지 않는다. 일반적으로 물리 시뮬레이션 단계만 수행되기 때문에 훈련 프로세스가 빨라진다.

```
$ ./isaaclab.sh -p scripts/reinforcement_learning/sb3/train.py --task Isaac-Cartpole-v0 --num_envs 64 --headless
```

2. Headless execution with off-screen render: 위의 headless 실행은 시뮬레이션을 렌더링하지 않기 때문에 훈련 중에 에이전트의 행동을 시각화할 수 없다. 에이전트의 행동을 시각화하기 위해 오프스크린 렌더링을 활성화(<mark>--enable_cameras</mark>)하고 훈련 중에 에이전트의 행동을 녹화(<mark>--video</mark>)하는 플래그를 전달한다.

```
$ ./isaaclab.sh -p scripts/reinforcement_learning/sb3/train.py --task Isaac-Cartpole-v0 --num_envs 64 --headless --video
```

저장된 비디오는 <mark>logs/sb3/Isaac-Cartpole-v0/$run-dir/videos/train</mark> 디렉토리에 저장된다.

3. Interactive execution: 아래 명령어를 통해 실시간으로 에이전트가 시뮬레이션 환경과 상호 작용하며 훈련되고 있는지 확인할 수 있다. 하지만, 시뮬레이션이 화면에 렌더링되므로 훈련 프로세스가 느려질 수 있다. 해결 방법으로 화면 오른쪽 하단에 도킹된 창에서 다른 렌더링 모드 간에 전환할 수 있다.

```
$ ./isaaclab.sh -p scripts/reinforcement_learning/sb3/train.py --task Isaac-Cartpole-v0 --num_envs 64
```

별도의 터미널에서 아래 명령을 실행하여 학습 진행 상황을 모니터링할 수 있다.

```
$ ./isaaclab.sh -p -m tensorboard.main --logdir logs/sb3/Isaac-Cartpole-v0
```

훈련이 완료되면 다음 명령을 실행하여 훈련된 에이전트를 Isaac Sim에서 시각화할 수 있다.

```
$ ./isaaclab.sh -p scripts/reinforcement_learning/sb3/play.py --task Isaac-Cartpole-v0 --num_envs 32 --use_last_checkpoint
```

### 4. Trained Agent Visualization

에이전트의 훈련이 완료되면 터미널에서 다음과 같은 로그를 확인할 수 있다.

```
------------------------------------------
| rollout/                |              |
|    ep_len_mean          | 293          |
|    ep_rew_mean          | 4.82         |
| time/                   |              |
|    fps                  | 6365         |
|    iterations           | 2000         |
|    time_elapsed         | 321          |
|    total_timesteps      | 2048000      |
| train/                  |              |
|    approx_kl            | 0.0043435125 |
|    clip_fraction        | 0.0512       |
|    clip_range           | 0.2          |
|    entropy_loss         | -0.0181      |
|    explained_variance   | 0.57         |
|    learning_rate        | 0.0003       |
|    loss                 | 0.0119       |
|    n_updates            | 39980        |
|    policy_gradient_loss | -0.00246     |
|    std                  | 0.247        |
|    value_loss           | 0.017        |
------------------------------------------
Saving model checkpoint to /workspace/isaaclab/logs/sb3/Isaac-Cartpole-v0/2025-02-07_07-05-22/model_2048000_steps.zip
```

아래는 오프스크린 렌더링을 사용하여 에이전트의 훈련 중 행동을 스텝 0, 2000, 30000에서 저장된 영상과, 최종 훈련된 모델의 체크포인트를 로드하여 환경에서 에이전트를 실행한 결과를 시각화한 영상이다.

* Step 0: Initial agent behavior at the beginning of training
<video width="640" height="360" controls>
  <source src="/assets/img/blog/20250208/rl-video-step-0.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

* Step 2000: Agent's progress after 2000 steps of training
<video width="640" height="360" controls>
  <source src="/assets/img/blog/20250208/rl-video-step-2000.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

* Step 30000: Agent's behavior after 30000 steps, nearing the end of training
<video width="640" height="360" controls>
  <source src="/assets/img/blog/20250208/rl-video-step-30000.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

* Final Model Execution: Agent performance after loading the final trained model checkpoint.
<video width="640" height="360" controls>
  <source src="/assets/img/blog/20250208/isaaclab_rl_demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

다음 포스팅에서는 기본 환경 설정을 다룬 후 강체(rigid object), 관절(articulation), 변형 가능한 객체(deformable object)와의 상호작용에 대한 튜토리얼을 진행할 예정이다.

---
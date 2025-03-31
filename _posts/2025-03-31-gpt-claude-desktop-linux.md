---
layout: post
title: "Installing ChatGPT and Claude Desktop Application on Ubuntu"
date: 2025-03-31
category:
  - Ubuntu
image: /assets/img/thumbnails/blog_4.png
author: Minho Lee
tags: 
  - LLM
  - Linux
  - ChatGPT
  - Claude
comments: true
toc: true
---

💡 요즘 다양한 LLM(Large Language Model)이 생겨나면서 각 모델마다 가진 강점에 맞춰 여러 영역에서 활용하고 있다.
그 중에서 내가 일상적으로 가장 자주 사용하는 모델을 꼽자면 **ChatGPT**와 **Claude**인 것 같다.
대부분 이런 모델들을 사용할 때 Chrome과 같은 웹 브라우저를 통해 접근하는데, 작업을 계속하다 보면 브라우저 탭이 점점 많아져서 관리가 번거로워지게 된다. 이런 불편함을 해소하기 위해 나는 주로 작업하는 Ubuntu 환경에서 웹브라우저 없이 실행할 수 있는 **Desktop Application**을 설치해 사용할 수 있는 방법을 찾게 되었고, 이번 포스팅에서는 그 과정을 쉽게 따라할 수 있도록 정리해보려고 한다.

### 1. ChatGPT Desktop Application
#### 1.1. Installation via Snap
Linux의 소프트웨어 배포 시스템인 <mark>Snap</mark>을 통해 설치하는 방법이다.

```bash
$ sudo apt update
$ sudo apt install snapd
```

```bash
$ sudo snap install chatgpt-desktop
```

```bash
$ chatgpt-desktop
```
* 실행 결과
<p align="center"><img src ="/assets/img/blog/20250331/fig_1.png"></p>

* macOS나 Windows 환경의 경우 [OpenAI 공식 사이트](https://openai.com/chatgpt/desktop/)를 통해 간단하게 설치할 수 있다.

#### 1.2. Installation via .deb Package

ChatGPT Desktop Application을 제공하는 GitHub 프로젝트에서 .deb 패키지를 다운로드하여 설치하는 방법도 있다.

아래 URL로 이동하면 최근 릴리즈된 버전의 여러 패키지들을 확인할 수 있다.

> 🔗 [https://github.com/lencx/ChatGPT/releases](https://github.com/lencx/ChatGPT/releases)

<p align="center"><img src ="/assets/img/blog/20250331/fig_2.png"></p>

여기서 Ubuntu 시스템에서 설치하기 위해 <mark>ChatGPT_1.1.0_linux_x86_64.deb</mark> 파일을 다운로드한다.</br>
다운로드한 .deb 패키지 파일을 마우스 오른쪽 버튼으로 클릭하고, 다른 응용 프로그램으로 열기 옵션에서 **소프트웨어 설치 옵션**을 선택한다.</br>
소프트웨어 설치 페이지가 나타나면 설치 버튼을 눌러 설치를 진행하고, 설치가 완료되면 Desktop Application이 설치된 것을 확인할 수 있다.

### 2. Claude Desktop Application
#### 2.1. Installation via .deb Package
Linux에서 Claude Desktop을 네이티브로 실행하기 위한 작업[(k3d3/claude-desktop-linux-flake)](https://github.com/k3d3/claude-desktop-linux-flake)에서 파생된 GitHub Repository에서 제공해주는 Build Script를 통해 설치를 진행하는 방법을 소개한다.

```bash
$ git clone https://github.com/aaddrick/claude-desktop-debian.git
```

패키지를 빌드하기 전에 <mark>Node.js >= 12.0.0 and npm</mark> 버전이 요구되기 때문에 버전 업데이트를 진행해준다.
아래는 이미 설치된 버전이 요구하는 버전을 만족하는지 확인하는 방법부터 소개한다.

```bash
# Check whether the installed Node.js and npm versions meet the minimum requirement
$ node -v
v10.19.0
$ npm -v
6.14.4

# If your versions are too old, install the latest Node.js LTS version.
# npm will be installed automatically along with Node.js.
$ curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
$ sudo apt install -y nodejs

$ node -v
v18.19.0
```

```bash
$ cd claude-desktop-debian

# Run the build script to generate a Debian package
$ sudo ./build-deb.sh

✓ Package built successfully at: /home/user/claude-desktop-debian/build/claude-desktop_0.9.0_amd64.deb
🎉 Done! You can now install the package with: sudo dpkg -i /home/user/claude-desktop-debian/build/claude-desktop_0.9.0_amd64.deb

# Replace ${VERSION} with the version printed by the build script
sudo dpkg -i ./build/claude-desktop_${VERSION}_amd64.deb

$ sudo dpkg -i ./build/claude-desktop_0.9.0_amd64.deb
```

```bash
$ claude-desktop
```

* 실행 결과
<p align="center"><img src ="/assets/img/blog/20250331/fig_3.png"></p>

이 Desktop Application에서는 최근 AI 트렌드에서 주목받고 있는 키워드 중 하나인 Claude의 **MCP (Model Context Protocol**)도 지원한다.</br>
MCP의 구성 파일은 <mark>~/.config/Claude/claude_desktop_config.json</mark> 경로에 위치하고 있다. (**MCP**에 대해서는 곧 자세히 다뤄볼 예정이다.)

### 3. Coming up next...
앞서 소개한 ChatGPT와 Claude의 Desktop Application은 웹 서버를 통해 동작하기 때문에 인터넷 연결이 필수적이다.</br>
다음 글에서는 **DeepSeek, Llama, Phi, Mistral, Gemma 등**과 같은 **오픈소스 LLM**을 로컬 머신에 설치하고, 인터넷 연결 없이 사용할 수 있는 방법에 대해서 알아보도록 하자.

---
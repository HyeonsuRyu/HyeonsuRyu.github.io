---
layout: post
title: 「Expert Python Programming 4th」PyPI와 pip
subtitle: 「Expert Python Programming 4th」PyPI와 pip
categories: Python
tags: [Python, PyPI, pip, site-packages, dist-packages, 전문가를 위한 파이썬 프로그래밍]
---
## PyPI
Python은 거대한 생태계를 가져 수많은 패키지를 이용할 수 있다. `PyPI`(Python Packaging Index)는 이러한 패키지를 저장하는 공개 저장소이다. Python의 몇몇 패키지들은 여러 운영체제가 표준 컴포넌트로 내장되어 있어 `apt-get`, `rpm`, `emerge` 등으로도 설치할 수 있다. 그러나 구버전의 패키지가 있을 가능성이 크다. 파이썬 패키징 어소시티는 `pip`를 이용하여 PyPI의 패키지를 설치할 것을 권장한다.
## pip
`pip`는 하나의 독립 프로젝트이다. 모든 파이썬 릴리스에는 `ensurepip` 모듈이 포함되어있다. `ensurepip` 모듈은 패키지가 항상 사용자의 환경에서 `pip` 설치가 가능하도록 보장한다.
### 패키지 설치 경로
`pip`로 설치된 패키지들은 `site-packages` 디렉터리 중 하나에 설치된다. `site-packages` 디렉터리는 크게 `global site-packages(dist-packages)` 디렉터리 혹은 `user site-packages` 디렉터리에 설치된다. 
#### 패키지 우선 순위
설치된 패키지는 `user site-packages` 디렉터리에 설치 된 패키지가 `dist-packages` 디렉터리에 설치된 패키지보다 항상 우선한다.  
대부분의 운영체제들은 필수적인 파이썬 패키지를 `dist-packages`에 포함하고 있다. 또한 일반 사용자가 `dist-packages` 디렉터리에 패키지를 설치하도록 권한을 주지 않는다. `sudo`등을 통하여 `dist-packages`디렉터리에 패키지를 설치할 수는 있지만, 운영체제에서 의도적으로 낮은 버전의 패키지를 설치해 둔 경우 다른 버전의 패키지로 업데이트되며 문제가 될 수 있다.

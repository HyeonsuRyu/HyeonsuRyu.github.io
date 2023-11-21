---
layout: post
title: 「Expert Python Programming 4th」런타임 환경 격리
categories: Python
tags: [환경 격리, 가상 환경, venv, poetry, 머신 가상화, 컨테이너화, 도커, 디펜던시, 전문가를 위한 파이썬 프로그래밍]
---
## 환경 격리란
프로젝트가 정상적으로 작동하기 위하여 필요한 패키지를 `디펜던시`라 한다. 여러 프로젝트가 서로 다른 버전의 패키지를 필요로 하는 경우 [`pip`](https://hyeonsuryu.github.io/python/2023/11/02/Expert-Python-Programming-4th-PyPI%EC%99%80-pip.html)는 여러 버전의 패키지 설치를 허용하지 않으므로 디펜던시 문제가 발생하게 된다. `환경 격리`는 하나의 컴퓨터 시스템에서 디펜던시 문제를 해결하기 위하여 필요하다. 환경 격리는 [`애플리케이션 레벨 격리`](###애플리케이션-레벨-격리)와 [`시스템 레벨 격리`](###시스템-레벨-격리)로 나뉜다.

### 애플리케이션 레벨 격리
애플리케이션 격리는 주로 `가상 환경`을 이용한다. 가상환경은 프로젝트가 실행되는 환경의 파이썬 인터프리터와 패키지를 격리하는 것이다. 파이썬의 [`venv`](https://docs.python.org/3/library/venv.html)모듈 혹은 [`poetry`](https://python-poetry.org/)를 통해 쉽게 가상 환경을 생성할 수 있다.

#### venv
venv는 파이썬의 내장 모듈로, 시스템 셸에서 직접 호출하여 사용한다.

##### 환경 만들기
```bash
python3.10 -m venv <env-name>
```
venv를 이용하여 가상 환경을 만드는 방법은 위와 같다.   
`python3.10`대신 `python3`을 이용해도 된다. 그러나  `python3`에 `python3.12`가 있을지, `python3.10`이 있을지는 컴퓨터 시스템 환경에 따라 다르다. 따라서 이용하고자 하는 파이썬 인터프리터의 버전을 `python3.10`과 같이 명시하는 것이 좋다.

##### 가상 환경 구성
가상 환경을 만들면, 하위 디렉터리에 여러 디렉터리가 생성된다. 하위 디렉터리는 컴퓨터 시스템에 따라 각각의 컨벤션에 맞게 구성된다. 

##### 가상 환경 활성화
가상 환경을 활성화 하기 위해서는 다음과 같은 명령어를 실행한다.
<table>
<thead>
  <tr>
    <th>Platform</th>
    <th>Shell</th>
    <th>명령어</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="4">POSIX</td>
    <td>bash/zsh</td>
    <td>$  source  &lt;venv&gt/bin/activate</td>
  </tr>
  <tr>
    <td>fish</td>
    <td>$ source &lt;venv&gt;/bin/activate.fish</td>
  </tr>
  <tr>
    <td>csh/tcsh</td>
    <td>$ source &lt;venv&gt;/bin/activate.csh</td>
  </tr>
  <tr>
    <td>PowerShell</td>
    <td>$ &lt;venv&gt;/bin/Activate.ps1</td>
  </tr>
  <tr>
    <td rowspan="2">Windows</td>
    <td>cmd.exe</td>
    <td>C:\&gt; &lt;venv&gt;\Scripts\activate.bat</td>
  </tr>
  <tr>
    <td>PowerShell</td>
    <td>PS C:\&gt; &lt;venv&gt;\Scripts\Activate.ps1</td>
  </tr>
</tbody>
</table>

##### 디펜던시 관리
venv는 가상 환경 안에서 pip로 패키지를 설치하더라도 이를 추적, 관리하지 않는다. 대신 가상 환경의 절대 경로만을 사용한다. 따라서 디렉터리를 임의로 이동시키면 가상 환경이 작동하지 않는다.  
이 문제를 해결하기 위하여 디펜던시를 쉽게 갈리하는 프랙티스를 사용한다. 프랙티스는 관습적으로 디펜던시를 requirements.txt 파일에 작성하여 만든다.
``` text
# hash기호는 주석
# requirements.txt 예시

# 일반적으로 필요한 패키지의 버전을 고정한다.
eventlet==0.17.4
graceful==0.1.1

# 여러 버전에서 작동하는 것을 확인한 경우 범위를 지정할 수 있다.
falcon>=0.3.3,<0.5.0

# 버전이 없는 패키지는 지양하지만, 항상 최신 버전이 필요한 경우 버전을 명시하지 않는다.
pytz
```
pip를 통해 이 파일에 있는 모든 디펜던시를 쉽게 설치할 수 있다.
`$ pip install -r requirements.txt`
  
requirements.txt 파일은 venv에 의해 자동으로 업데이트 되지 않으므로 직접 수정해야 한다. `pip freeze`를 통해 설치된 모든 패키지를 표시할 수 있다. 그러나 중첩된 디펜던시도 표시하므로 양이 빠르게 늘어나 관리가 어렵다.

#### poetry
poetry는 파이썬의 디펜던시와 가상 환경 관리를 돕는 패키지로 Python3.8 이상에서 작동한다.  
poetry가 지원하는 기능은 다음과 같다.
* 파이썬 프로젝트와 가상 환경을 함께 생성
* 기존 프로젝트를 가상 환경에 대해 초기화
* 프로젝트 디펜던시 관리
* 라이브러리 패키징

##### 설치
poetry는 venv와 마찬가지로 가상 환경을 생성하므로 가상 환경 밖에 설치해야 한다.
PyPI에서 제공되므로 pip를 통해 설치할 수 있다. 그러나 pip는 전역적인 영향을 미치며 설치되므로 [`pipx`](https://python-poetry.org/docs/#installing-with-pipx)를 이용하여 설치하거나 [poetry의 공식 인스톨러](https://python-poetry.org/docs/#installing-with-the-official-installer)를 이용해야한다.

##### 가상 환경 생성
poetry로 새로운 프로젝트를 생성할 경우 다음 명령어를 사용한다.
```bash
$ poetry new <project-name>
```
`poetry new`를 통해 생성된 프로젝트는 아래와 같은 구조를 가진다.
```bash
<project-name>
├── pyproject.toml
├── README.md
├── <project-name>
│   └── __init__.py
└── tests
    └── __init__.py
```

poetry로 기존 프로젝트를 관리하고 싶은 경우 다음 명령어를 사용한다.
```bash
$ poetry init
```
`poetry init`은 `pyproject.toml` 파일만 생성한다.

##### pyproject.toml
pyproject.toml 파일은 환경 설정 파일이다.
구성은 다음과 같다.
* [tool.poetry]
	* 기본 프로젝트의 메타 데이터로 PyPI에 공개할 때 필요한 정보이다.
* [tool.poetry.dependencies]
	* 프로젝트 디펜던시 목록이다.
* [tool.poetry.dev-dependencies]
	* 로컬 개발시 필요한 디펜던시 목록으로 개발에만 필요하므로 분리하여 관리한다.
* [build-system]
	* poetry를 프로젝트 관리용 빌드 시스템으로 설명한다.

##### 패키지 설치
환경 안에서 패키지를 설치하기 위해서는 다음 명령어를 이용한다.
```bash
$ poetry add <name>
```
`poetry add`를 이용하여 패키지를 설치한 경우 자동으로 pyproject.toml 파일의 디펜던시 목록이 업데이트 된다.  
하나의 디펜던시가 추가될 때 poetry는 자동으로 그 디펜던시의 디펜던시(전이 디펜던시) 또한 pyproject.toml에 추가한다.  
디펜던시는 특정 버전이 아닌 범위로 지정되어 있을 수 있다. poetry는 여러 디펜던시를 비교하고 프로젝트가 작동할 수 있는 특정한 버전을 찾는다.  여전히 여러 버전이 가능할 경우 poetry가 선택하는 버전이 환경을 만들 때 마다 다를 수 있다. 이런 경우 작동이 잘 되는 환경에서 `poetry lock` 명령어를 통해 poetry가 항상 같은 버전의 디펜던시를 사용하도록 `poetry.lock`파일을 만들 수 있다.  
`poetry.lock` 또한 `poetry add`가 실행될 경우 자동으로 업데이트 된다.


### 시스템 레벨 격리
소프트웨어는 데이터베이스, 코드, HTTP 서버 등 여러 요소가 함께 사용될 수 있다. 그러나 HTTP 서버, 데이터베이스, 공유 라이브러리 등은 파이썬 패키지에 포함되어있지 않다. 이런 요소들은 파이썬 가상 환경 안에 캡슐화 할 수 없는 `외부 디펜던시`이다.  
외부 디펜던시는 시스템 종류에 따라 버전이나 특징이 바뀐다. 따라서 가상 환경을 이용하면 시스템 종류가 달라질 경우 외부 디펜던시에 의해 소프트웨어가 작동하지 않는다.  
시스템 레벨 격리는 시스템 전체를 애플리케이션 환경으로 격리하여 외부 디펜던시에 의한 문제를 해결하기 위해 사용되며 `머신 가상화`와 `운영체제 레벨 가상화(컨테이너화)` 방법이 있다.

#### 머신 가상화
머신 가상화는 컴퓨터 시스템 전체를 에뮬레이션 하는 방법이다. 시스템 전체를 에뮬레이션 하기 때문에 오버헤드가 크다. 머신 가상화를 위한 도구로는 베이그런트가 있다.  

#### 컨테이너화
컨테이너화는 운영체제 안의 사용자 공간을 격리하는 방법이다. 하드웨어 에뮬레이션이 필요하지 않기 때문에 머신 가상화보다 오버헤드가 적어 경량 가상화라고 불리기도 한다. 코드가 운영체제에 크게 의존하거나 운영 환경에서 컨테이너 사용이 불가능 하면 컨테이너화를 통해 만들어진 소프트웨어는 작동하지 않는다. 그러나 컨테이너화 만으로도 충분히 시스템이 격리될 수 있고, 머신 가상화보다 오버헤드가 적기 때문에 컨테이너화가 보통 선호된다. 

##### 도커
도커는 리눅스에서 이용 가능한 컨테이너의 구현체 중 하나이다. Windows나 macOS에서도 도커를 설치하면 리눅스 운영체제인 `중간 가상 머신`이 생성되어 머신 가상화와 컨테이너화가 함께 쓰이는 방식으로 실행된다. 중간 가상 머신이 있더라도 머신 가상화에 비해 크게 오버헤드가 커지지는 않는다.
텍스트 파일인 `도커파일`에 컨테이너 이미지가 기술되며, 기술된 이미지들은 `이미지 저장소`에 구현 및 저장된다. 이미지 저장소를 통해 이미지를 재사용할 수 있다.

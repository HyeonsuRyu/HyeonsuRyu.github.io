---
layout: post
title: Python
subtitle:
categories:
tags:
  - Poetry
  - lockfile
  - requirements
date: 2026-10-15
---
# 문제 현상
기존 `requirements.txt`에 정의된 python 패키지를 Poetry 프로젝트에 설치했지만, 그 결과가 `pyproject.toml`과 `poetry.lock`에 반영되지 않았다.

# 원인 분석
## Poetry와 pip의 의존성 관리 방식 차이
* pip는 `requirements.txt`를 단순히 현재 위치에 설치한다.
* Poetry는 `pyproject.toml`을 기준으로 프로젝트 의존성을 관리하고, `poetry.lock`을 통해 구체적인 버전을 고정한다.
* Poetry 환경에서 `requirements.txt`를 설치하여도 이는 poetry에 반영되지 않는다.

# 해결 방법
## A. requirements.txt를 poetry 의존성으로 변환하기
기존 `requirements.txt`를 poetry 의존성 관리 방식으로 전환한다.
방법은 다음과 같다.
1. 수동으로 하나씩 추가
   `poetry add <패키지명>`으로 requirements.txt에 명시된 패키지를 직접 설치하는 방식이다.
2. xargs를 활용하기(Mac/Linux)
   `cat requirements.txt | xargs poetry add` 명령어를 통해 `poetry add` 방식을 자동화한다.
3. powershell script를 이용하기(Windows)
   `Get-Content requirements.txt | ForEach-Object { poetry add $_ }` 명령어를 통해 `poetry add` 방식을 자동화한다.

## B. uv 사용하기
`poetry`는 `requirements.txt`를 내보낼 수 있지만, 직접 설치하는 방법은 제공하지 않는다.
그러나 `uv`는 `uv add -r requirements.txt` 방식을 통해 패키지들을 추가하고 동기화까지 제공한다. 
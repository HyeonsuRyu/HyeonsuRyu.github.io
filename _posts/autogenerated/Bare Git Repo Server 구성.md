---
layout: post
title: Bare Git Repo Server 구성
subtitle:
categories: Infrastructure
tags:
  - Bare-Git-Repository
date: 2026-09-26
---
이 포스트는 **Bare Git Repo**를 서버에 생성한 후,
**SSH 프로토콜**로 클라이언트에서 통신하는 예제이다.
# 환경
## 서버
| 프로그램 | 버전                 |
| ---- | ------------------ |
| OS   | Ubuntu 24.04.5 LTS |
| Git  | 2.43.0             |
## 클라이언트
| 프로그램 | 버전                  |
| ---- | ------------------- |
| OS   | Windows 11 Pro 25H2 |
| Git  | 2.51.1.windows.1    |
# 1. Bare 저장소 생성
```bash
# 저장소 디렉터리 생성 및 이동
mkdir <저장소이름>
cd <저장소이름>

# (옵션) 기본 브랜치 이름 설정
git config --global init.defaultBranch <브랜치이름>

# Bare Repository 생성
git init --bare <저장소이름>.git
```

# 2. 클라이언트 initial push
```bash

```
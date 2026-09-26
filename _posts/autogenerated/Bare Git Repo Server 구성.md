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
# repository를 저장할 디렉터리 생성 및 이동
mkdir <디렉터리이름>
cd <디렉터리이름>

# (옵션) 기본 브랜치 이름 설정
git config --global init.defaultBranch <브랜치이름>

# Bare Repository 생성
git init --bare <repository이름>.git
```

# 2. (옵션) git 전용 계정 및 git-server 디렉터리 생성
git 전용 계정은 일반 사용자 계정처럼 생성할 수 있다.
그러나 [git](https://git-scm.com/book/ko/v2/Git-%EC%84%9C%EB%B2%84-%EC%84%9C%EB%B2%84-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)에서는 사용자들이 git 작업만 실행하도록 제한하는 방법을 추천한다.
아래는 위 링크에서 소개하는 git 전용 계정 및 서비스 디렉터리 생성 방법이다.
## 서버
```bash
# 계정 생성 및 git 계정 shell로 로그인
sudo adduser git
su git

# ssh 접속용 키 관리 파일 생성
cd
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```
## 클라이언트
아래는 윈도우 클라이언트에서 ssh-keygen을 이용해 ED25519 방식 ssh 키를 발급하는 예시이다.
```powershell
ssh-keygen -t ed25519 -C "email@example.com"
```
이후 생성된 `pub` 파일을 git 서버로 옮긴다.

# 3. 클라이언트 initial push
ssh 통신이 이미 가능해야 한다. 사용자가 여러명인 경우, 서버 계정을 여러개 만들기보다, git 계정를 하나 만들고 여러 사용자들의 ssh 키를 git 계정에 등록하는 방법을 [Git 홈페이지](https://git-scm.com/book/ko/v2/Git-%EC%84%9C%EB%B2%84-%EC%84%9C%EB%B2%84%EC%97%90-Git-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)에서 추천하고 있다.
```bash
# 레포지토리 생성 및 초기화
mkdir <repository이름>
cd <repository이름>
git init
echo "# Hello World!" > README.md
git add .
git commit -m "feat: initial commit"

# origin 등록
# 접속 ssh 계정이 bare repository에 대한 권한이 있어야한다.
git remote add origin ssh://<계정명>@<서버주소>[:<접속포트>]/<Bare Repository 디렉터리 경로>
# ex) git remote add origin ssh://git@192.168.0.2/srv/git/repo.git

# 브랜치 설정 및 push
git branch -M <브랜치이름>
git push -u origin <브랜치이름>
```

# 4. 클라이언트 clone
``` bash
git clone ssh://<계정명>@<서버주소>[:<접속포트>]/<Bare Repository 디렉터리 경로>
```
클라이언트 로컬에 repository가 정상적으로 받아졌으면 성공이다.
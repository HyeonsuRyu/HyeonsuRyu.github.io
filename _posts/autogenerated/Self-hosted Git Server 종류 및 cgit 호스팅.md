---
layout: post
title: Git Server 종류 및 cgit 호스팅
subtitle:
categories: Infrastructure
tags:
  - Git Server
  - cgit
date: 2026-09-24
---
# 1. Self-hosted Git Server의 필요성
`GitHub`, `GitLab`, `SourceForge` 등 Git hosting 사이트는 [여러 곳](https://archive.kernel.org/oldwiki/git.wiki.kernel.org/index.php/GitHosting.html)이 있다. 그러나 Private Repository를 사용하더라도, 데이터를 외부 서버에 저장된다는 점에서 **민감한 정보**를 취급하기 어렵다.

 나의 경우 홈랩의 구성을 PKM으로 관리하여 RAG를 연동해 사용하고자 하려는 중이었다. PKM 저장소를 Git으로 형상 관리하며 여러 기기에서 동기화 하며 사용하는 환경인데, 홈랩의 포트 정보, 내부 IP 구조, 방화벽 정책 등을 GitHub에 업로드 하는 것은 부담되어 홈랩에 Git Server를 직접 호스팅하기로 결정했다.

# 2. Git Server의 종류
 직접 호스팅 할 수 있는 Git Server에는 아래와 같이 여러 종류가 있다.
 1. Bare Git server
 2. cgit
 3. Gitea
 4. Forgejo
 
## 2-1 Bare Git server
 특별한 서비스 없이, 서버에 **Git 저장소를** 하나 생성한 후 이를 클라이언트 기기에서 **원격 저장소**로 사용하는 방식이다. 아주 기본적인 원격 저장소의 기능만 하기 때문에, web 인터페이스가 없으며, SSH 키 등을 통해 권한을 가지고 사용할 수 있다

#
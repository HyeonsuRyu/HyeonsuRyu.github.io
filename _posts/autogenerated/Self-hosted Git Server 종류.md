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
# 1 Self-hosted Git Server의 필요성

`GitHub`, `GitLab`, `SourceForge` 등 Git hosting 사이트는 [여러 곳](https://archive.kernel.org/oldwiki/git.wiki.kernel.org/index.php/GitHosting.html)이 있다. 그러나 Private Repository를 사용하더라도, 데이터를 외부 서버에 저장된다는 점에서 **민감한 정보**를 취급하기 어렵다.

최근 [GItHub의 가동률](https://www.githubstatus.com/)이 99.9%에 미치지 못하는 경우가 자주 있다. 특히 Github Actions에서 찾은 장애를 보이는데, 고가용성이 중요한 환경인 경우 셀프 호스팅을 고려할 수 있다.

 나의 경우 홈랩의 구성을 PKM으로 관리하여 RAG를 연동해 사용하고자 하려는 중이었다. PKM 저장소를 Git으로 형상 관리하며 여러 기기에서 동기화 하며 사용하는 환경인데, 홈랩의 포트 정보, 내부 IP 구조, 방화벽 정책 등을 GitHub에 업로드 하는 것은 부담되어 홈랩에 Git Server를 직접 호스팅하기로 결정했다.

---
# 2 Git Server의 종류
 직접 호스팅 할 수 있는 Git Server에는 아래와 같이 여러 종류가 있다.
 1. Bare Git server
 2. cgit
 3. Gitea
 4. Forgejo
 
## 2-1 [Bare Git server](https://git-scm.com/book/ko/v2/Git-%ec%84%9c%eb%b2%84-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c)

특별한 서비스 없이, 서버에 **Bare Git 저장소를** 하나 생성한 후 이를 클라이언트 기기에서 **원격 저장소**로 사용하는 방식이다. 아주 기본적인 원격 저장소의 기능만 하기 때문에, web 인터페이스가 없으며, SSH 키 등을 통해 권한을 가지고 사용할 수 있다.

### 장점
 * 추가 프로그램 설치가 없다.
 * 매우 가볍다.
 
### 단점
 * CI/CD, Pull Request 등 협업이나 Git 외의 기능이 불가능 하다.
 * Web 환경이 제공되지 않는다.

## 2-2 [cgit](https://git.zx2c4.com/cgit/about/)

cgit은 git 서버 프로그램이 아니다. Bare Git Server와 함께 사용되며, 레포지토리를 웹으로 보여주는 단순한 기능만 한다. 따라서 Nginx 인증, VPN 등의 별도 접근 제어가 필요하다.

### 장점
* 매우 가볍다.
* Web 환경이 제공된다.
* 오픈소스이다. ([GPLv2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html))

### 단점
* CI/CD, Pull Request 등 협업이나 Git 외의 기능이 불가능 하다.

## 2-3 [Gitea](https://about.gitea.com/) / [Forgejo](https://forgejo.org/)

GitHub와 유사한 환경을 제공하는 셀프 호스팅 플랫폼으로, 경량화가 잘 되어있는 것이 특징이다. Forgejo는 원래 Gitea에서 포크되어 나온 프로젝트로, 현재는 하드 포크되어 gitea와 더이상 호환이 보장되지 않는다.

### 장점
* 가볍다.
* 오픈소스이다. ([MIT](https://github.com/go-gitea/gitea/blob/main/LICENSE)(Giteea) / [GPLv3](https://codeberg.org/forgejo/forgejo/src/branch/forgejo/LICENSE))(Forgejo))
* Pull Request, Ci/CD 기능이 포함되어 있다.
* On-premise, cloude 환경이 모두 지원된다.
* 필요한 경우, Enterprise 버전을 구독하여 기술 지원을 받을 수 있다.

### 단점
* 가볍긴 하나, Bare Git Server, cgit과 비교하면 오버헤드가 있다.
* 단순히 Git으로 형상 관리만 하는 경우 불필요하 기능들이 포함된다.
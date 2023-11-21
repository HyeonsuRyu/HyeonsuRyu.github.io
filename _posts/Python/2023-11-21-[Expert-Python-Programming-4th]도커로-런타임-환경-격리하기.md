---
layout: post
title: 「Expert Python Programming 4th」도커로 런타임 환경 격리하기
categories: Python
tags: [환경 격리, 머신 가상화, 컨테이너화, 도커, 전문가를 위한 파이썬 프로그래밍]
---
## 도커
도커는 [런타임 환경 격리](https://hyeonsuryu.github.io/python/2023/11/04/Expert-Python-Programming-4th-%EB%9F%B0%ED%83%80%EC%9E%84-%ED%99%98%EA%B2%BD-%EA%B2%A9%EB%A6%AC.html)를 구현하기 위하여 사용되는 컨테이너의 구현체 중 하나이다. Dockerfile에 컨테이너에 대한 이미지를 기술하여 쉽게 런타임 환경을 구현하도록 해준다. 도커 설치는 [가이드](https://www.docker.com/get-started)를 참조하면 된다.
* OS별 가이드 링크
	* [Mac](https://docs.docker.com/desktop/install/mac-install/)
	* [Windows](https://docs.docker.com/desktop/install/windows-install/)
	* [Linux](https://docs.docker.com/desktop/install/linux-install/)

### 도커 파일
도커파일은 애플리케이션을 실행하기 위한 모든 `시스템 라이브러리`, `파일`, `소스코드` 및 `디펜던시`들을 캡슐화 한다.  
도커 파일을 작성하기 위해서는 아래 형식의 인스트럭션을 사용한다.  
```bash
INSTRUCTION arguments
```
도커를 사용할 때 기본적으로 사용되는 인스트럭션은 다음과 같다.  
* `FROM <image-name>`
	* 이미지가 사용할 기반 이미지(base image)를 기술한다.
	* 일반적으로 리눅스 시스템 배포판, 추가 라이브러리와 소프트웨어로 구성된다.
	* 기본 도커 이미지 저장소는 [도커 허브(Docker Hub)](https://hub.docker.com)로 무료로 이미지를 구할 수 있다.
* `COPY <src>...<dst>`
	* 로컬 빌드 환경(src)에서 컨테이너 파일시스템(dst)에 파일을 복사한다.
* `ADD <src>...<dst>`
	* 로컬 빌드 환경에서 파일을 복사하는 COPY와 다르게 src에 입력되는 URL에서 아카이브의 압축을 풀고 dst에 추가한다.
* `RUN <command>`
	* 이전 레이어의 최상층에서 command를 실행한다.
	* 실행 후 결과를 새로운 이미지 레이어로 파일시스템에 추가한다.
* `ENTRYPOINT ["<executable>", "<param>", ...]`
	* 컨테이너가 시작할 때 실행되는 기본 명령어를 설정한다.
	* Entrypoint가 지정되지 않으면 `/bin/sh -c`(Bash 등의 이미지 기본 셸)가 실행된다.
* `CMD ["<param>", ...]`
	* Entrypoint에 대한 기본 파라미터를 지정한다.
	* Entrypoint가 지정되지 않은 경우 `/bin/sh -c`가 실행되므로 `CMD ["<executable>", "<param>", ...]`로 사용할 수 있다.
		* 그러나 ENTRYPOINT 인스트럭션으로 실행 대상을 직접 정의하고 CMD는 매개변수 정의를 위해 사용하는 것이 권장된다.
* `WORKDIR <dir>`
	* `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD` 인스트럭션을 위한 현재 작업 디렉터리를 설정한다.  

### 도커 파일 예시
HTTP 에코 웹서버를 도커화 하는 과정은 다음과 같다.
1. [애플리케이션 작성](####애플리케이션-작성)
2. [requirements.txt 작성](####requirements.txt-작성)
3. [도커파일 작성](####도커파일-작성)
4. [이미지 빌드](####이미지-빌드)
5. [컨테이너 실행](####컨테이너-실행)

#### 애플리케이션 작성
HTTP 에코 웹서버를 간단히 구현하기 위해 flask를 사용한다. flask가 설치되어있지 않다면 `pip`를 통해 설치한다.
```bash
$ pip install flask
```
flask를 이용한 웹서버 코드는 아래와 같다.
```python
# echo.py

from flask import Flask, request
app = Flask(__name__)

@app.route('/')
def echo():
	return(
		f"METHOD: {request.method}\n"
		f"HEADERS:\n{request.headers}"
		f"BODY:\n{request.data.decode()}"
	)

if __name__ == '__main__':
	app.run(host="0.0.0.0", port=5000) #port가 지정되지 않은 경우 기본으로 5000번 포트를 사용하며, 필요시 변경할 수 있다.
```

코드를 작성했다면 아래 명령어로 실행해 볼 수 있다.
```bash
$ python3 echo.py
```
명령어를 실행하면 5000번 port에서 서버가 시작되어 `http://localhost:5000`으로 인터넷 브라우저에서 접속할 수 있다.

#### requirements.txt 작성
애플리케이션은 `flask` 하나만 사용하므로 requirements.txt의 내용은 다음과 같다.
``` requirements.txt
flask==3.0.0 #pip list를 이용하여 버전 확인
```
requirements.txt 작성의 자세한 내용은 이 [포스트](https://hyeonsuryu.github.io/python/2023/11/04/Expert-Python-Programming-4th-%EB%9F%B0%ED%83%80%EC%9E%84-%ED%99%98%EA%B2%BD-%EA%B2%A9%EB%A6%AC.html#h-%EB%94%94%ED%8E%9C%EB%8D%98%EC%8B%9C-%EA%B4%80%EB%A6%AC)를 참고하면 된다.

#### 도커파일 작성
HTTP 에코 웹서버를 컨테이너화 하기 위해서는 도커파일에 다음 내용이 기술되어야 한다.
1. Python이 포함된 기반 이미지
2. 작업 디렉터리 설정
3. requirements.txt 파일 작업 디렉터리 최상단에 복사
4. requirements.txt 파일 이미지에 추가
5. `echo.py` 파일 작업 디렉터리 최상단에 복사
6. 컨테이너 시작 시 python 실행  

이를 dockerfile로 작성하면 다음과 같다.
```dockerfile
FROM python:3.10-slim
WORKDIR /app/

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY echo.py .
ENTRYPOINT ["python", "echo.py"]
```

#### 이미지 빌드
컨테이너를 실행 하기 전 아래 명령어로 도커파일에 정의된 이미지를 빌드해야 한다.
```bash
$ docker build -t <name> <path>
```
* `-t <name>`
	* 이미지의 이름을 지정한다.
* `<path>`
	* 도커파일이 위치한 디렉터리 경로를 지정한다.  

예시는 다음과 같다.
```bash
$ docker build -t echo .
```
docker images를 통해 사용할 수 있는 이미지 목록을 검색하면 echo가 생성된 것을 확인할 수 있다.
```bash
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
echo         latest    xxxxxxxxxxxx   16 seconds ago   140MB
```
파일 크기가 굉장히 큰 것을 확인할 수 있는데, 리눅스 시스템 배포판 전체와 파이썬까지 포함되었기 때문이다. 그러나 도커 이미지들은 레이어 구조로 되어있어 다시 사용되는 레이어는 캐시를 통해 중복되어 로드되지 않는다.

#### 컨테이너 실행
`echo` 이미지를 실행하기 위해 다음 명령어를 사용한다.
```bash
docker run -it --rm --publish 5000:5000 echo
```
* `-it`
	* -i와 -t를 결합한 것이다.
	* -i는 컨테이너 프로세스가 분리되더라도 STDIN을 열린 상태로 유지한다.
	* -t는 표준 입출력과 연결된 터미널을 컨테이너에 할당한다.
	* -it는 실시간 로그 확인과 키보드 인터럽트를 통한 프로세스 종료를 하게 해준다.
* `--rm`
	* 도커 컨테이너가 존재하면 자동으로 제거하는 옵션이다.
	* 도커는 디버깅을 위해 컨테이너를 제거하지 않는데, 필요하지 않은 경우 --rm 옵션을 통해 제거하는 것이 좋다.
* `--publish 5000:5000`
	* 도커 컨테이너의 5000번 포트를 호스트 인터페이스의 5000번 포트로 바인딩한다.
	* echo 애플리케이션의 8080 포트를 노출시키고 싶다면 `--publish 8080:5000` 과 같이 사용할 수 있다.
		* 이 옵션을 사용하면 `echo.py`에는 5000포트를 사용하도록 되어있지만, `curl http://localhost:8080`을 통해 서버에 접속 가능하다.

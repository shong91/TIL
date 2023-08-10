---
layout: post
title: [Docker] ch02-3. 도커파일 
tags: [docker, kubernetes]
author: shong91
excerpt_separator: 
---
# chapter02-3 도커 파일

## 1. 도커파일로 이미지 생성하기
이미지 생성을 위해 컨테이너에 설치해야하는 패키지, 추가해야하는 소스코드, 명령어, 셸 스크립트 등을 하나의 파일(도커파일)에 기록해두고, 도커는 이 도커파일을 읽어 컨테이너에서 작업을 수행하여 이미지를 생성한다.

→ 도커파일을 이용하여 직접 컨테이너 생성 / 이미지로 커밋하는 번거로움을 줄이고, 애플리케이션 빌드 및 배포의 자동화 가능


## 2. DockerFile 작성 - 주요 명령어 정리

```
FROM ubuntu:14.04
MAINTAINER syeon
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND                            
```
-	FROM: 생성할 이미지의 베이스가 될 이미지. (ex) ubuntu:14.04)
-	MAINTAINER(LABEL maintainer): 이미지를 생성한 개발자의 정보
-	LABEL: 이미지에 메타 데이터 추가 (key:value)
-	RUN: 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행 (ex) apt-get install~)
-	ADD: 파일을 이미지에 추가. DockerFile이 위치한 컨텍스트 디렉터리에서 가져온다.
-	WORKDIR: 명령어를 실행할 디렉토리 
-	EXPOSE: DockerFile의 빌드로 생성된 이미지에서 노출할 포트 설정
-	CMD: 컨테이너가 시작될 때마다 실행할 커맨드를 설정. 커맨드를 내장하면 컨테이너 생성 시 별도의 커맨드를 입력하지 않아도 내장된 커맨드가 적용되어 컨테이너가 실행된다. 


## 3. DockerFile 빌드 

도커파일 이미지 빌드하기
```
docker build -t mybuild:0.0 ./ (현재 디렉터리)
```

이미지 빌드 후, 생성된 이미지로 컨테이너 실행하기 
```
docker run -d -P --name myserver mybuild:0.0
```

-P 옵션을 통해 도커파일에서 EXPOSE한 포트를 호스트에 연결하도록 설정한다. `docker port 컨테이너(myserver)` 명령어를 통해 컨테이너가 호스트의 어떤 포트와 연결됐는지 확인할 수 있다.


### 빌드 컨텍스트

빌드 컨텍스트: 이미지 생성에 필요한 각종 파일, 소스코드, 메타데이터 등을 담고있는 디렉토리. DockerFile이 위치한 디렉토리가 빌드 컨텍스트가 된다. 

- 이미지 빌드 순서: 빌드 컨텍스트 읽기 -> 빌드 컨텍스트의 파일을 이미지에 추가 -> ADD/COPY 명령어를 통해 파일을 이미지에 추가
- 빌드는 이미지 빌드에 필요한 파일만 있는 빌드 컨텍스트 디렉토리에서 진행하여야 하며, 루트 디렉토리 등 다른 곳에서 이미지 빌드할 경우 하위 디렉토리까지 전부 포함하게 되어 불필요한 파일로 인해 호스트메모리에 부하가 올 수 있다. 

    => 이를 방지하기 위해 **.dockerignore** 파일을 이용하여 제외할 파일 관리한다.

### 도커파일을 이용한 컨테이너 생성과 커밋
DockerFile의 RUN, ADD 등의 명령어가 실행될 때마다 새로운 컨테이너가 하나씩 생성되며, 이를 이미지로 커밋한다. 

(일괄 생성&커밋이 아니라, 도커 파일을 한 줄 한 줄 읽을 때마다 새로운 컨테이너를 생성하고, 다시 새로운 이미지 레이어로 저장한다)

### 캐시를 이용한 이미지 빌드
한 번 이미지 빌드를 마친 뒤, 다시 같은 빌드를 진행하면 이전의 이미지 빌드에서 사용했던 캐시를 사용한다.
```
docker build -f Dockerfile2 -t mycache:0.0 ./
```
 
```
...
--->Using cache
...
```


캐시 기능이 필요하지 않을 시에는 build 명령어에 --no-cache 옵션을 추가한다.
```
docker build –no-cache -t mybuild:0.0 .
```


--cache-from 옵션을 사용하여 특정 이미지를 캐시로 사용하도록 직접 지정할 수 있다. 
```
docker build –cache-from nginx -t my_extend_nginx:0.0 .
```


### 멀티 스테이지를 이용한 이미지 빌드
일반적으로 애플리케이션 빌드 시, 의존성 패키지와 라이브러리를 많이 사용하게 된다. 

이러한 라이브러리 등이 불필요하게 이미지의 크기를 차지하게 되는 것을 막기 위하여, 도커 엔진헤서는 멀티 스테이지 빌드 방법을 사용한다. 
 
**멀티 스테이지 빌드**는, 하나의 Dockerfile 안에 여러 개의 FROM 이미지를 정의함으로써 빌드 완료 시 최종적으로 생성될 이미지의 크기를 줄이는 역할을 한다.

```$xslt
FROM golang
ADD main.go /root
WORKDIR /root
RUN go build -o /root/mainApp /root/main.go

FROM golang
ADD main2.go /root
WORKDIR /root
RUN go build -o /root/mainApp2 /root/main2.go


FROM alpine:latest
WORKDIR /root
COPY --from=0 /root/mainApp
COPY --from=1 /root/mainApp2
CMD ["./mainApp"]
                    
```

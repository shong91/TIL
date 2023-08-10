---
layout: post
title: [Docker] ch04. 도커 컴포즈 
tags: [docker, kubernetes]
author: shong91
excerpt_separator: 
---
# chapter04 도커 컴포즈

## 4.1 도커 컴포즈
매번 run 명령어에 옵션을 더해 CLI로 컨테이너를 생성하기 보다는, 여러 개의 컨테이너를 하나의 서비스로 정의해 컨테이너 묶음으로 관리하는 것이 더욱 편리하다.
도커 컴포즈는 컨테이너를 이용한 서비스의 개발과 CI를 위해 **여러 개의 컨테이너를 하나의 프로젝트로서 다룰 수 있는 작업 환경을 제공**한다. 


- 여러 개의 컨테이너의 옵션과 환경을 정의한 파일을 읽어 컨테이너를 순차적으로 생성한다
- run 명령어의 옵션을 그대로 사용 가능하며, 컨테이너의 의존성, 네트워크, 볼륨 등을 함께 정의할 수 있다
- 스웜 모드와 유사하게, 컨테이너 수를 유동적으로 조절할 수 있으며, 디스커버리도 자동으로 설정 가능하다

## 4.2 도커 컴포즈의 활용

### 1. 설치 & 버전 확인 

```
# curl -L https://github.com/docker/compose/releases/download/1.11.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# chmod +x /usr/local/bin/docker-compose

# docker-compose -v
```

### 2. docker-compose.yml 활용
```
version: '3.0'
services:
  web:
        image: alicek106/composetest:web
        ports:
          - "80:80"
        links:
         - mysql:db
        command: apachectl -DFOREGROUND
  mysql:
        image: alicek106/composetest:mysql
        command: mysqld
~                          
```

- version: yaml 파일 포맷의 버전. 
- services: 도커 컴포즈로 생성할 컨테이너 옵션을 정의. 이 항목에 쓰인 서비스는 컨테이너로 구현되며, 하나의 프로젝트로서 도커 컴포즈에 의해 관리된다. 
- image: 서비스의 컨테이너를 생성할 때 쓰일 이미지
- links: 다른 서비스에 서비스명만으로 접근할 수 있도록 설정 (docker run --link)
- environment: 서비스의 컨테이너 내부에서 사용할 환경변수를 지정 (docker run --env / -e)
- command: 컨테이너가 실행될 때 수행할 명령어
- depends_on: 특정 컨테이너에 대한 의존관계. 이 항목에 명시된 컨테이너가 먼저 실행된다
- ports: 서비스의 컨테이너를 개방할 포트 설정 (docker run -p)

### 3. 도커 컴포즈 실행/삭제
```
# docker-compose scale mysql=2

# docker-compose up -d

# docker-compose down

# docker-compose ps
```


```
# 이미지를 빌드하기만 하며, 컨테이너를 시작하지는 않음
docker-compose build

# 이미지가 존재하지 않을 경우에만 빌드하며, 컨테이너를 시작
docker-compose up

# 필요치 않을 때도 강제로 이미지를 빌드하며, 컨테이너를 시작
docker-compose up --build

# 이미지 빌드 없이, 컨테이너를 시작 (이미지 없을 시 실패)
docker-compose up --no-build 
```
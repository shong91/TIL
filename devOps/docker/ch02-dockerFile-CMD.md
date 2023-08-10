---
layout: post
title: [Docker] ch02-3. 도커파일 - 명령어 정리
tags: [docker, kubernetes]
author: shong91
excerpt_separator: 
---
# chapter02-3 도커 파일

## 4. DockerFile 기타 명령어

### 1. ENV
DockerFile에서 사용될 환경변수 지정. 설정된 환경변수는 ${ENV_NAME} 또는 $ENV_NAME 의 형태로 사용한다. 

환경변수는 도커파일과 이미지에 저장되므로, 이미지로 컨테이너를 생성할 시에 환경변수를 사용할 수 있다. 
```
# vi DockerFile

FROM ubuntu:14.04
ENV test /home
WORKDIR $test
RUN touch $test/mytouchFile

...
[root@localhost ~]# docker build -t myenv:0.0 ./
...
[root@localhost ~]# docker run -i -t --name env_test myenv:0.0 /bin/bash
root@5b81f9b3a77b:/home# echo $test
/home
```

docker run 명령어에서 `-e` 옵션을 사용해 같은 이름의 환경변수를 사용할 시 기존의 값은 덮어쓰여진다. 

```
[root@localhost ~]# docker run -i -t --name env_test_override \
> -e test=myvalue \
> myenv:0.0 /bin/bash
root@efe4c6e1ed2d:/home# echo $test
myvalue

```

`${env_name:-value}`: 기존 값이 설정되어 있지 않을 경우, value 값을 환경변수로 설정한다. 
반대로, `${env_name:+value}`는 기존 값이 설정되어 있다면 value 값을 환경변수값으로 설정하며, 기존 값이 없다면 빈 문자열을 사용한다.
    
    
```
# vi DockerFile

FROM ubuntu:14.04
ENV my_env my_value
RUN echo ${my_env:-value} / ${my_env:+value_update} / ${my_env_2:-value} / ${my_env_2:+value}

...
[root@localhost ~]# docker build .
...
Step 3/3 : RUN echo ${my_env:-value} / ${my_env:+value_update} / ${my_env2:-value} / ${my_env2:+value}
 ---> Running in 5aa32adaa2f4
my_value / value_update / value /

```

### 2. VOLUME
빌드된 이미지로 컨테이너를 생성 시, 호스트와 공유할 컨테이너 내부의 디렉터리를 설정한다.

여러 개의 디렉터리를 설정 시, 연속으로 여러 개를 작성하거나 JSON 배열 형식을 사용할 수 있다. 

```
# vi Dockerfile

FROM ubuntu:14.04
RUN mkdir /home/volume
RUN echo test >> /home/volume/testfile
VOLUME /home/volume

...
[root@localhost ~]# docker build -t myvolume:0.0 .
[root@localhost ~]# docker run -i -t --name volume_test myvolume:0.0

```

### 3. ARG
docker build 명령어를 실행할 때, Dockerfile 내에서 사용될 변수의 값을 설정한다. 

입력 형식은 `key=value` 의 쌍을 이룬다. 

```
# vi Dockerfile

FROM ubuntu:14.04
ARG my_arg
ARG my_arg_2=value2
RUN touch ${my_arg}/mytouch

...
[root@localhost ~]# docker build --build-arg my_arg=/home -t myarg:0.0 ./
[root@localhost ~]# docker run -i -t --name arg_test myarg:0.0
root@daa1fe309051:/# ls /home/mytouch
/home/mytouch

```

### 4. USER
컨테이너 내에서 사용될 사용자 계정의 이름이나 UID 를 설정하면, 그 아래의 명령어는 해당 사용자 권한으로 실행된다. 

컨테이너가 호스트의 root 권한을 가지는 것은 보안 측면에서 바람직하지 않으므로, 컨테이너 내부에서는 root 사용자를 설정하는 것을 권장한다. 
```
...
RUN groupadd -r author && useradd -r -g author $user_id
USER $user_id
```

<hr>

### 5. ONBUILD
빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile로 생성될 때 실행할 명령어를 추가한다. 

```
# vi Dockerfile
FROM ubuntu:14.04
RUN echo "this is onbuild test"!
ONBUILD RUN echo "onbuild!" >> /onbuild_file

# vi Dockerfile2
FROM onbuild_test:0.0
RUN echo "this is child image!"

...
[root@localhost ~]# docker build -f ./Dockerfile2 ./ -t onbuild_test:0.1
# Executing 1 build trigger
Step 1/1 :  RUN echo "onbuild!" >> /onbuild_file

...
[root@localhost ~]# docker run -i -t --rm onbuild_test:0.1 ls /
bin   dev  home  lib64	mnt	      opt   root  sbin	sys  usr
boot  etc  lib	 media	onbuild_file  proc  run   srv	tmp  var

```

Dockerfile2 를 빌드할 때, Dockerfile에 설정해놓은 ONBUILD 명령어가 동작한다. onbuild_test:0.1 이미지로 컨테이너를 생성하면, echo >> /onbuild_file 이 존재하는 것을 확인할 수 있다. 
 
이렇듯, onbuild 명령어는 이미지가 빌드될 때 수행되어야 하는 각종 명령어를 미리 저장해놓을 수 있다. 다른 Dockerfile 명령어들이 이미지의 속성을 설정하는 것과 달리, ONBUILD 명령어는 부모 이미지의 자식 이미지에만 적용되며, 자식 이미지는 ONBUILD 속성을 상속받지 않는다. 

또한, `ONBUILD ADD` 를 이용하여 보다 깔끔하게 Dockerfile 을 사용할 수 있다. 


### 6. STOPSIGNAL
컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정한다. (DEFAULT = SIGTERM)

```
# vi Dockerfile
FROM ubuntu:14.04
STOPSIGNAL SIGKILL

...
[root@localhost ~]# docker build . -t stopsignal:0.0
[root@localhost ~]# docker run -itd --name stopsignal_container stopsignal:0.0
[root@localhost ~]# docker inspect stopsignal_container | grep Stop
            "StopSignal": "SIGKILL"

```

### 7. HEALCHECK
이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정한다. (HEALTHCHECK 에서 사용되는 명령어인 `curl` 을 먼저 설치하여야 함)

```
# vi Dockerfile
FROM nginx
RUN apt-get update -y && apt-get install curl -y
HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1

...
[root@localhost ~]# docker run -d -P nginx:healthcheck
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                        PORTS                   NAMES
c46432c52003        nginx:healthcheck   "/docker-entrypoint.…"   About a minute ago   Up About a minute (healthy)   0.0.0.0:32768->80/tcp   sharp_gauss
7808a25a0150        stopsignal:0.0      "/bin/bash"              7 minutes ago        Up 7 minutes                                          stopsignal_container
...
```

### 8. SHELL
Dockerfile 에서 기본적으로 사용하는 셸은, 리눅스에서는 `/bin/sh -c`, 윈도우에서는 `cmd /S /C` 이다. 

별도의 셸을 지정하고자 할 시 SHELL 명령어를 사용하여 설정할 수 있다. 


<hr>

### 9. ADD, COPY
컨텍스트로부터 이미지에 파일을 추가/복사 한다는 점에서 ADD/COPY의 기능은 같으나, 

COPY는 로컬 디렉터리의 파일만 이미지에 추가할 수 있는 반면, ADD는 외부 URL 및 tar 파일에서도 파일을 추가할 수 있다. (ADD ⊃ COPY)

(가용 범위는 ADD가 넓지만, 로컬 컨텍스트로부터 파일을 직접 추가하는 COPY 가 보다 명확하기 때문에 ADD 보다는 COPY를 사용하는 것이 권장된다.)


### 10. ENTRYPOINT, CMD
두 명령어는 컨테이너가 시작될 때 실행할 명령어를 설정한다. 단, ENTRYPOINT 는 커맨드를 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있다는 점에서 차이가 있다. 

ENTRYPOINT 를 설정하지 않고 CMD 만 설정했을 경우, CMD 에 설정된 명령어를 그대로 실행하지만, 

ENTRYPOINT 를 지정하였을 경우, CMD 는 ENTRYPOINT 의 인자값으로 기능한다. 


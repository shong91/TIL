---
layout: post
title: [Docker] ch02-3. 도커파일 - 명령어 정리 (1)
tags: [docker, kubernetes]
author: hhhongso
excerpt_separator: 
---
# chapter02-3 도커 파일

## 4. DockerFile 기타 명령어

#### 1. ENV
- DockerFile에서 사용될 환경변수 지정. 설정된 환경변수는 ${ENV_NAME} 또는 $ENV_NAME 의 형태로 사용한다. 
- 환경변수는 도커파일과 이미지에 저장되므로, 이미지로 컨테이너를 생성할 시에 환경변수를 사용할 수 있다. 
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

- docker run 명령어에서 `-e` 옵션을 사용해 같은 이름의 환경변수를 사용할 시 기존의 값은 덮어쓰여진다. 

```
[root@localhost ~]# docker run -i -t --name env_test_override \
> -e test=myvalue \
> myenv:0.0 /bin/bash
root@efe4c6e1ed2d:/home# echo $test
myvalue

```

- `${env_name:-value}`: 기존 값이 설정되어 있지 않을 경우, value 값을 환경변수로 설정한다. 
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

#### 2. VOLUME
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

#### 3. ARG
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

#### 4. USER
컨테이너 내에서 사용될 사용자 계정의 이름이나 UID 를 설정하면, 그 아래의 명령어는 해당 사용자 권한으로 실행된다. 

컨테이너가 호스트의 root 권한을 가지는 것은 보안 측면에서 바람직하지 않으므로, 컨테이너 내부에서는 root 사용자를 설정하는 것을 권장한다. 
```
...
RUN groupadd -r author && useradd -r -g author $user_id
USER $user_id
```

<hr>

#### 5. ONBUILD
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


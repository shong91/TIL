---
layout: post
title: [Docker] ch02-4. 도커 데몬
tags: [docker, kubernetes]
author: shong91
excerpt_separator: 
---
# chapter02-4 도커 데몬

## 1. 도커의 구조
클라이언트로서의 도커, 서버로서의 도커

- 도커 서버: 컨테이너를 생성, 실행하며 이미지를 관리하는 주체. dockerd 프로세스로 동작

    도커 엔진은 외부에서 API 입력을 받아 도커 엔진의 기능을 수행하는데, 도커 프로세스가 실행되어 서버로서 입력받을 준비가 된 상태를 도커 데몬이라고 한다. 

- 도커 클라이언트: 도커 데몬은 API 입력을 받아 도커 엔진의 기능을 수행하는데, 이 API를 사용할 수 있도록 CLI(Command Line Interface)를 제공하는 것.

- 프로세스: 사용자가 docker 명령어 입력 -> 도커 클라이언트가 유닉스 소켓(docker.sock) 을 사용하여 도커 데몬에게 명령어 전달 -> 도커 데몬은 명령어를 파싱하여 이에 해당하는 작업 수행 -> 수행 결과를 도커 클라이언트에게 반환

## 2. 도커 데몬 실행 
도커 서비스는 `dockerd` 로 도커 데몬을 실행한다. 
```
# dockerd

...
INFO[2020-06-14T04:12:09.083782797-04:00] API listen on /var/run/docker.sock  
```

## 3. 도커 데몬 설정 - 명령어 옵션 정리 
### 1. 도커 데몬 제어: -H
도커 데몬의 API를 사용할 수 있는 방법을 추가한다.

기본값은 유닉스 소켓인 `/var/run.docker.sock` 이며, -H 옵션에 IP주소와 포트번호를 입력하면 Docker Remote API로 도커를 제어할 수 있다. 

단, Remote API를 지정할 경우 유닉스 소켓을 지정하지 않고 Remote API 바인딩 주소만 지정할 경우, 유닉스 소켓은 비활성화 되므로 도커 클라이언트를 사용할 수 없게 된다. 
(유닉스 소켓과 Remote API 바인딩 주소를 동시에 설정해주어야 함!)

```
# dockerd -H unix:///var/run/docker.sock -H tcp:0.0.0.0:2375
```

### 2. 도커 데몬에 보안 적용: --tlsverify
도커에는 기본적으로 보안 연결이 설정되어 있지 않으므로, 도커 제어를 위한 보안 설정이 필요하다. 

### 3. 도커 스토리지 드라이버 변경: --storage-driver
기본값은 ubuntu: overlay2, CentOS: devicemapper 이며, --storage-driver 옵션을 이용하여 OverlayFS, AUFS, Btrfs, Devicemapper, VFS, ZFS 등으로 선택 가능하다. 

cf)**스토리지 드라이버의 원리** 

컨테이너 내부에서 읽기, 쓰기 작업이 일어날 때에는 드라이버에 따라 CoW(Copy-on-Write), RoW(Redirect-on-Write) 개념을 사용한다. 도커 스토리지 드라이버는 CoW, RoW 을 지원한다. 

스냅숏(Snapshot)은, 원본 파일은 읽기 전용으로 사용하되, 이 파일이 변경되면 새로운 공간을 할당한다는 개념이다.

도커 이미지/컨테이너에서, 이미지 레이어는 각 스냅숏에 해당하고, 컨테이너는 이 스냅숏을 사용하는 변경점이라 볼 수 있다. 
컨테이너 레이어에는 이전 이미지에서 변경된 사항이 저장되어 있으며, 컨테이너를 이미지로 만들면 변경된 사항이 스냅숏으로 생성되고 하나의 이미지 레이어로 존재하게 된다. 

## 4. 도커 데몬 모니터링
도서 서버의 효율적인 관리 & 문제 발생 시 원인 파악, PaaS로써 도커를 제공하기 위해 실시간으로 도커 데몬의 상태를 체크하기 위해 도커 데몬을 모니터링한다. 

### 1. 도커 데몬 디버그 모드: -D
도커 데몬을 디버그 옵션을 실행하여 Remote API의 입출력, 로컬 도커 클라이언트에서 오가는 모든 명령어 로그를 출력할 수 있다. 

단, 모든 로그가 출력되기 때문에 보다 효율적으로 모니터링하기 위한 명령어를 사용한다. 

```
# dockerd -D

```

### 2. 도커 데몬 모니터링 명령어 - events, stats, system df 
1) events <br>
    도커 데몬에 일어나고 있는 일을 실시간 스트림 로그로 보여주는 명령어. 
    
    컨테이너 관련 명령어(attach, commit, copy, create 등), 이미지 관련 명령어(delete, import, load, pull, push 등), 볼륨, 네트워크, 플러그인 등에 관한 명령어의 수행 결과를 출력한다.  
   
    ```
    # docker events
    # docker system events    
    ```
   
2) stats <br>
    실행 중인 모든 컨테이너의 자원 사용량(CPU, 메모리 제한 및 사용량, 네트워크 I/O, 블록 I/O 등)을 스트림으로 출력하는 명령어. 스트림이 아닌, 한 번만 출력하는 방식으로 사용 시에는 `--no-stream` 옵션을 추가한다. 
    
    ```
    # docker stats --no-stream
    ```

3) system df <br>
    도커에서 사용하고 있는 이미지, 컨테이너, 로컬 볼륨의 총 개수 및 사용 중인 개수, 크기, 삭제함으로써 확보 가능한 공간을 출력하는 명령어. 
    
    => 사용하지 않는 컨테이너와 볼륨은 `docker container prune`, `docker volume prune` 으로 삭제하여 공간을 확보한다. 
    (연습한답시고 이거저거 만들어놓고 안지웠더니 RECLAIMABLE의 상태가... 확인하고 싹 지웠다. )
    
   ```
   # docker system df
   ...
    TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
    Images              38                  8                   3.613GB             3.557GB (98%)
    Containers          10                  1                   1.682kB             1.682kB (100%)
    Local Volumes       12                  2                   857.6MB             857.6MB (99%)
    Build Cache         0                   0                   0B                  0B

    ```
   
### 3. 컨테이너 모니터링 도구 - CAdvisor
생성된 모든 컨테이너의 자원 사용량, 도커 데몬의 정보, 상태, 호스트의 자원 사용량을 한 번에 확인할 수 있다. 

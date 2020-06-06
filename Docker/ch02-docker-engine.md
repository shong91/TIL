---
layout: post
title: ［Docker］ ch02-1. 도커 이미지와 컨테이너 
tags: [docker, kubernetes]
author: hhhongso
excerpt_separator: 
---
# chapter02-1 도커 이미지와 컨테이너 

## 2.1 도커 이미지와 컨테이너
1. 도커 이미지
    - 컨테이너를 생성할 때 필요한 요소
    - 여러 개의 계층으로 된 바이너리 파일로 존재하며, 컨테이너의 생성/실행 시 읽기 전용으로 사용
    - OS(우분투, CentOS), 애플리케이션(Apache 웹 서버, MySQL 데이터베이스 등), 빅데이터 분석 도구(하둡, 스파크, 스톰) 등을 도커 이미지로 생성할 수 있음. 
    - \[저장소 이름\]/\[이미지 이름\]:\[이미지 버전\] ex) shong91/ubuntu:14.04
<br><br>
2. 도커 컨테이너
    - 도커 이미지로 컨테이너를 생성하여 파일 시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 **독립된 공간**
    - 컨테이너에서 무엇을 하든 원래 이미지는 영향을 받지 않으며, 특정 컨테이너에서 애플리케이션을 설치/삭제하더라도 다른 컨테이너/호스트에는 변화가 없다
    - 도커 이미지의 종류에 따라 알맞은 설정/파일을 가지고 있으며, 도커 이미지의 목적에 따라 사용되는 것이 일반적
 

## 2.2 도커 컨테이너

### 1. 컨테이너 생성
1) 컨테이너 생성 <br>
    run 명령어: 컨테이너 생성 && 컨테이너 내부로 들어간다
   ```
   docker run -i -t ubuntu:14.04
   ```
   <br>
   create 명령어: 컨테이너 생성만. 실행 및 내부로 들어가기 위해서는 `start` 및 `attach` 명령어 사용
   
   ```
   docker create -i -t --name myubuntu ubuntu:14.04
   docker start myubuntu
   docker attach myubuntu 
   ```
          
2) 이미지 내려받기 <br>
    ```
   docker pull centos:7
    ```

3) 이미지 목록 확인 <br>
    ```
   docker images
    ```

<br><br>
   
### 2. 컨테이너 목록 확인 / 삭제 
1) 컨테이너 목록 확인 
    `docker ps` 는 정지되지 않은 컨테이너만 출력한다. 정지된 컨테이너를 포함하여 모든 컨테이너를 출력 시 `-a` 옵션을 추가한다. 
   
    ```
   docker ps -a
    ```

2) 컨테이너 삭제 
    실행 중인 컨테이너는 삭제할 수 없으므로, 정지한 뒤 삭제하거나 `-f` 옵션을 추가한다.
   
   ```
   docker rm -f myubuntu
   ```  
   
   모든 컨테이너를 삭제할 때에는 `prune` 명령어를 사용한다. 단, 삭제된 컨테이너는 복구할 수 없으므로 유의하자. 
   
   ```
   docker container prune
   ```
   
   
   
### 3. 컨테이너 바인딩
새로운 컨테이너를 생성한 뒤, `ifconfig` 명령어로 네트워크 인터페이스를 확인한다.
  
```
root@562548c36a7d:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:04  
          inet addr:172.17.0.4  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:586 (586.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```  

<br><br>
도커의 NAT IP인 172.17.0.4 를 할당받은 eth0 인터페이스와, 로컬 호스트인 lo 인터페이스가 있다. 
기본적으로 컨테이너는 외부에서 접근할 수 없으며 도커가 설치된 호스트에서만 접근할 수 있기 때문에, 외부에 컨테이너의 애플리케이션을 노출하기 위해서는 eth0의 IP와 포트를 호스트IP와 포트에 바인딩하여야 한다. 
<br><br>

`-p` 옵션을 추가하여 컨테이너의 포트를 호스트의 포트와 바인딩한다. (`-p [호스트의 포트]:[컨테이너의 포트]`)
여러 개의 포트를 외부에 개방 시 `-p` 옵션을 여러번 사용하여 설정할 수 있다. 

```
docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
docker run -i -t -p 3306:3306 -p 192.168.0.100:7777:80 ubuntu:14.04
```
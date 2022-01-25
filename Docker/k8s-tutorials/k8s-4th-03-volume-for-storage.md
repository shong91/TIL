# Configure a Pod to Use a Volume for Storage

https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/

## what is Volume ?

컨테이너를 실행시키면, 컨테이너 내부에 디스크가 존재하는데, 이 디스크에 저장된 내용은 컨테이너가 죽으면 데이터가 날라가는 휘발성 구조이다.

이러한 컨테이너의 특징을 보완하기 위해 PVC 볼륨을 사용한다.

PVC 볼륨을 사용하여 컨테이너와 독립적이며 보다 일관된 스토리지를 사용하여, 데이터의 영속성을 보존할 수 있다.

이는 레디스(Redis)와 같은 키-값 저장소나 데이터베이스와 같은 스테이트풀 애플리케이션에 매우 중요하다.

## Hands-on

### 목표:

- 스토리지의 볼륨을 사용하는 파드 구성
- 생성한 파드를 kill 한 후 재기동 하였을 때, 데이터가 유지되는지 확인

### 실습

### 1. 파드에 볼륨 구성

컨테이너가 종료되고, 재시작 하더라도 파드의 수명동안 지속되는 `emptyDir` 유형으로 파드를 생성한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

```
kubectl apply -f https://k8s.io/examples/pods/storage/redis.yaml
```

파드의 상태를 계속 지켜보기 위해 아래 명령어를 띄워두고, 새 터미널에서 실습을 계속한다.

```
kubectl get pod redis --watch

```

### 2. 파일 생성 & 파드 강제 kill

실행중인 파드 컨테이너의 셸로 접근하여, `/data/redis` 경로에 파일을 생성한다.

```
kubectl exec -it redis -- /bin/bash
root@redis:/data# cd /data/redis/
root@redis:/data/redis# echo Hello > test-file
```

셸에서 실행 중인 프로세스 목록을 확인한다.

```
root@redis:/data/redis# apt-get update
root@redis:/data/redis# apt-get install procps
root@redis:/data/redis# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
redis        1  0.1  0.1  33308  3828 ?        Ssl  00:46   0:00 redis-server *:6379
root        12  0.0  0.0  20228  3020 ?        Ss   00:47   0:00 /bin/bash
root        15  0.0  0.0  17500  2072 ?        R+   00:48   0:00 ps aux
```

위와 같이 프로세스 목록을 확인 후, redis 프로세스를 죽여보자.

```
root@redis:/data/redis# kill <pid>
```

`kubectl get pod redis --watch` 를 띄워놓은 터미널에 들어가보면,

Redis 파드가 `restartPolicy:Always` 로 생성되어 있기 때문에, redis 파드가 죽은 뒤 자동으로 다시 생성된다.

```
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          82s
redis   0/1     Completed   0          98s
redis   1/1     Running     1          101s
```

### 3. 파일 손실 여부 확인

재시작된 컨테이너의 셸에 접근하여, `/data/redis` 경로에 파일이 여전히 존재하는지 확인한다.

```
kubectl exec -it redis -- /bin/bash
root@redis:/data/redis# cd /data/redis/
root@redis:/data/redis# ls
test-file
```

## References.

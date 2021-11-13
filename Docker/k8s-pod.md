# Pod

## Pod's Lifecycle

파드는 아래의 생명주기를 가진다.

1. Pending

- 최초의 단계
- Status는 Phase, Conditions, Reason 으로 구성됨
  - Phase: Pending, Running, Succeeded, Failed, Unknown
  - Conditions: Initialized, ContainerReady, PodScheduled, Ready
  - Reason: ContainersNotReady, PodCompleted
- 컨테이너를 초기화하고, 노드 스케쥴링을 설정

2. Running

- Pending 이후 파드가 실행되는 단계

3. Succeeded

- Job/CronJob 으로 생성된 파드의 경우, 일을 마치면 더이상 돌지(Running) 않고 Succeeded / Failed 상태로 전환0
- 파드의 모든 컨테이너가 성공적으로 배포 완료된 상태

4. Failed

- 장애가 발생하여 배포 실패한 상태

## Pod's Probe

프로브는 컨테이너에서 kubelet에 의해 주기적으로 수행되는 진단(diagnostic). 파드의 상태를 체크하며 쿠버네티스 운영의 안정성을 더해주는 기능을 한다.

진단 결과는 Success, Failure, Unknown 으로 나뉘며, Probe의 종류는 ReadinessProbe, LivenessProbe, StartupProbe가 있다.

1. ReadinessProbe

하나의 노드에 장애가 발생하여 죽을 경우, 파드는 Auto healing 기능을 통해 새로운 노드에 생성&배포된다.

이 때, Application 이 아직 완전히 구동되지 않은 상태라면, 서비스와 연결이 되자마자 트래픽이 새로운 파드로 유입되어 사용자는 50퍼센트의 확률로 에러페이지를 보게된다.

이를 방지하기 위해 `ReadinessProbe` 기능을 이용하여, Application 이 완전히 구동되기 전까지는 해당 파드와의 연결을 끊어놓고(Disconnection)

애플리케이션이 Ready 되면 해당 파드에 트래픽이 분산될 수 있도록 한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-readiness-exec1
  labels:
    app: readiness
spec:
  containers:
  - name: readiness
    image: kubetm/app
    ports:
    - containerPort: 8080
    readinessProbe:
      exec:
        command: ["cat", "/readiness/ready.txt"]
      initialDelaySeconds: 5    # 5초 지연 후 초기화
      periodSeconds: 10         # 10초 후에 재시도
      successThreshold: 3       # 3회 성공시 Ready: true
    volumeMounts:
    - name: host-path
      mountPath: /readiness
  volumes:
  - name : host-path
    hostPath:
      path: /tmp/readiness
      type: DirectoryOrCreate
  terminationGracePeriodSeconds: 0
```

2. LivenessProbe

Application 에 장애가 발생하는 경우는 크게 두 가지(서버 자체의 오류 / 서비스 오류)이다.

서비스 내에서 장애가 발생한 경우, Pod는 계속 Running 상태이기 트래픽 문제가 발생하게 된다.

이를 방지하기 위해 `LivenessProbe` 옵션을 사용하여, 해당앱에 문제가 생긴 후 파드를 재실행하게 만들어서

잠깐의 트래픽에러는 발생하겠지만, 지속적으로 에러가 발생하여 장애상황이 생기는것을 방지해준다.

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget1
  labels:
    app: liveness
spec:
  containers:
  - name: liveness
    image: kubetm/app
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 5   # 5초 지연 후 초기화
      periodSeconds: 10        # 10초 후에 재시도
      failureThreshold: 3      # 3회 실패시 Restart
  terminationGracePeriodSeconds: 0
```

## Quality of Service (QoS)

1. Guaranteed

2. Burstable

3. BestEffort

# Probe - Health check

- 컨테이너에서 kubelet에 의해 주기적으로 수행되는 진단(diagnostic). 파드의 상태를 체크하며 쿠버네티스 운영의 안정성을 더해주는 기능을 한다.

- 진단 결과는 Success, Failure, Unknown 으로 나뉘며, Probe의 종류는 ReadinessProbe, LivenessProbe, StartupProbe가 있다.

- 쿠버네티스는 각 컨테이너의 상태를 주기적으로 체크해서, 문제가 있는 컨테이너를 자동으로 재시작하거나 또는 문제가 있는 컨테이너(Pod를) 서비스에서 제외할 수 있다.

## 1. Liveness probe:

### 컨테이너가 살아 있는지 아닌지를 체크

컨테이너의 상태를 주기적으로 체크해서, 응답이 없으면 컨테이너를 자동으로 재시작해준다.

Liveness probe는 Pod의 상태를 체크하다가, Pod의 상태가 비정상인 경우 kubelet을 통해서 재시작한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: liveness
    image: gcr.io/terrycho-sandbox/liveness:v1
    imagePullPolicy: Always
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /readiness
        port: 8080
      initialDelaySeconds: 5   # 5초 지연 후 초기화
      periodSeconds: 10        # 10초 후에 재시도
      failureThreshold: 3      # 3회 실패시 Restart

```

initialDelaySecond를 주는 이유는, 컨테이너가 기동 되면서 애플리케이션이 기동될 때 설정 정보나 각종 초기화 작업이 필요하기 때문에, 컨테이너가 기동되자 마자 헬스 체크를 하게 되면, 서비스할 준비가 되지 않았기 때문에 헬스 체크에 실패할 수 있기 때문에 준비 기간을 주는 것이다.

준비 시간이 끝나면, periodSecond에 정의된 주기에 따라 헬스 체크를 진행하게 된다.

## 2. Readiness probe:

### 컨테이너가 서비스가 가능한 상태인지를 체크

컨테이너의 상태 체크중에 liveness의 경우에는 컨테이너가 비정상적으로 작동이 불가능한 경우도 있지만, Configuration을 로딩하거나, 많은 데이터를 로딩하거나, 외부 서비스를 호출하는 경우에는 일시적으로 서비스가 불가능한 상태가 될 수 있다.

이런 경우에는 컨테이너를 재시작한다 하더라도 정상적으로 서비스가 불가능할 수 있기 때문에, ReadinessProbe 를 사용하여 컨테이너를 일시적으로 서비스가 불가능한 상태로 마킹해주고(해당 파드와의 연결을 끊어놓고(Disconnection)), 애플리케이션이 Ready 되면 해당 파드에 트래픽이 분산될 수 있도록 한다.

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: readiness-rc
spec:
  replicas: 2
  selector:
    app: readiness
  template:
    metadata:
      name: readiness-pod
      labels:
        app: readiness
    spec:
      containers:
      - name: readiness
        image: gcr.io/terrycho-sandbox/readiness:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8080
          initialDelaySeconds: 5   # 5초 지연 후 초기화
          periodSeconds: 10        # 10초 후에 재시도
          failureThreshold: 3      # 3회 실패시 Restart
```

예를 들어 쿠버네티스 서비스에서 3개의 Pod를 로드밸런싱으로 서비스를 하고 있을때, Readiness probe 를 이용해서 서비스 가능 여부를 주기적으로 체크한다고 하자.

이 경우 하나의 Pod가 서비스가 불가능한 상태가 되었을때, 즉 Readiness Probe에 대해서 응답이 없거나 실패 응답을 보냈을때는 해당 Pod를 사용 불가능한 상태로 체크하고 서비스 목록에서 제외한다.

Liveness probe는 컨테이너의 상태가 비정상이라고 판단하면 해당 Pod를 재시작하는데 반해,

Readiness probe는 컨테이너가 비정상일 경우에는 해당 Pod를 사용할 수 없음으로 표시하고, 서비스 등에서 제외한다.

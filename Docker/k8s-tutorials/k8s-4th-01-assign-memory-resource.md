# Assign Memory Resources to Containers and Pods

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/57yt8

## what is ${개념} ?

컨테이너는 요청량 만큼의 메모리 확보가 보장되나, 상한보다 더 많은 메모리는 사용할 수 없다.
컨테이너에 `request`, `limit` 을 할당하여 컨테이너의 메모리를 관리할 수 있다.

## Hands-on

### 목표:

- 컨테이너 및 파드 메모리 리소스 할당하기 : 메모리 request, limit 을 지정
- 정상 지정/초과 지정하여 파드의 상태 확인하기

### 실습

### 1. 메모리 request, limit 을 정상 지정

메모리 요청량, 상한이 모두 range 안에 들어오는 파드를 생성해본다.

아래 파드는 100MiB 메모리 요청량과 200MiB 메모리 상한을 가지며,

`args` section 에서는 컨테이너가 시작될 때 arguments 를 제공한다.

`"--vm-bytes"`, `"150M"` 아규먼트는 컨테이너가 150 MiB 할당을 시도 하도록 한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

```
kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit.yaml --namespace=mem-example
```

request, limit 이 모두 정상 range 안에 들어오므로 파드가 정상 생성 & 구동됨을 확인할 수 있다.

```
kubectl get pod memory-demo --namespace=mem-example
kubectl get pod memory-demo --output=yaml --namespace=mem-example

```

`kubectl top` 명령어로 파드 메트릭을 가져온다.
파드가 약 150 MiB 해당하는 약 162,900,000 바이트 메모리를 사용하는 것을 보여준다. 이는 파드의 100 MiB 요청 보다 많으나 파드의 200 MiB 상한보다는 적다.

```
kubectl top pod memory-demo --namespace=mem-example

```

> Troubleshooting

error: Metrics API not available 가 왜 발생하는지 원인을 분석해보야아 ....

```
k top pod memory-demo --namespace=syhong
W0124 14:27:33.835662 968 top_pod.go:140] Using json format to get metrics. Next release will switch to protocol-buffers, switch early by passing --use-protocol-buffers flag
error: Metrics API not available
```

### 2. memory limit 을 초과하는 컨테이너를 생성해보기

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
```

```
kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit-2.yaml --namespace=mem-example
```

```
kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY     STATUS      RESTARTS   AGE
memory-demo-2   0/1       OOMKilled   1          24s
```

상한이 100MiB 인 컨테이너에 이를 초과하는 250MiB 의 메모리를 할당하려 하였기에, 컨테이너가 Running 되지 못하고 죽는 것을 확인할 수 있다. (STATUS OOMKilled = OutOfMemory)

> Troubleshooting

CrashedBackOff -> OOMKilled

### 3. 노드에 비해 너무 큰 메모리 request 지정

메모리 요청량과 상한은 컨테이너와 관련있지만, 파드가 가지는 메모리 요청량과 상한으로 이해하면 유용하다. 파드의 메모리 요청량은 파드 내 모든 컨테이너의 메모리 요청량의 합이다. 마찬가지로 파드의 메모리 상한은 파드 내 모든 컨테이너의 메모리 상한의 합이다.

파드는 요청량을 기반하여 스케줄링된다. 노드에 파드의 메모리 요청량을 충족하기에 충분한 메모리가 있는 경우에만 파드가 노드에서 스케줄링된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-3
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-3-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "1000Gi"
      requests:
        memory: "1000Gi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

```
kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit-3.yaml --namespace=mem-example
```

노드 내 메모리가 부족하여 파드가 스케줄링 되지 못하고, PENDING가 계속 지속된다.

```
kubectl get pod memory-demo-3 --namespace=mem-example
NAME            READY     STATUS    RESTARTS   AGE
memory-demo-3   0/1       Pending   0          25s
```

### 메모리 limit 을 지정하지 않으면 ?

컨테이너에 메모리 상한을 지정하지 않으면 다음 중 하나가 적용된다.

- 컨테이너가 사용할 수 있는 메모리 상한은 없다. 컨테이너가 실행 중인 노드에서 사용 가능한 모든 메모리를 사용하여 OOM Killer가 실행될 수 있다. 또한 메모리 부족으로 인한 종료 시 메모리 상한이 없는 컨테이너가 종료될 가능성이 크다.

- 기본 메모리 상한을 갖는 네임스페이스 내에서 실행중인 컨테이너는 자동으로 기본 메모리 상한이 할당된다. 클러스터 관리자들은 LimitRange를 사용해 메모리 상한의 기본 값을 지정 가능하다.

## References.

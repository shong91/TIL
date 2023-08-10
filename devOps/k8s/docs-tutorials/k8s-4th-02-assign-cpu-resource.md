# Assign CPU Resources to Containers and Pods

https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/

## CPU 리소스 관리 ?

메모리 리소스에 요청량과 상한을 할당하는 것과 마찬가지로, 컨테이너의 CPU 에 요청량과 상한을 할당할 수 있다.

컨테이너는 CPU 에 할당된 상한을 넘겨 사용할 수 없다.

## Hands-on

### 목표:

- 컨테이너 및 파드 CPU 리소스 할당하기 : CPU request, limit 을 지정

- 정상 지정/초과 지정하여 파드의 상태 확인하기

### 실습

### 1. CPU request, limit 을 정상 지정

```
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo
  namespace: cpu-example
spec:
  containers:
  - name: cpu-demo-ctr
    image: vish/stress
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: "0.5"
    args:
    - -cpus
    - "2"
```

```
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit.yaml --namespace=cpu-example
```

request, limit 이 모두 정상 range 안에 들어오므로 파드가 정상 생성 & 구동됨을 확인할 수 있다.

```
kubectl get pod cpu-demo --namespace=cpu-example
kubectl get pod cpu-demo --output=yaml --namespace=cpu-example
```

`kubectl top` 명령어로 파드 메트릭을 가져온다.

```
kubectl top pod cpu-demo --namespace=cpu-example
NAME                        CPU(cores)   MEMORY(bytes)
cpu-demo                    974m         <something>
```

### 2. 노드에 비해 너무 큰 CPU request 지정

```
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo-2
  namespace: cpu-example
spec:
  containers:
  - name: cpu-demo-ctr-2
    image: vish/stress
    resources:
      limits:
        cpu: "100"
      requests:
        cpu: "100"
    args:
    - -cpus
    - "2"
```

```
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit-2.yaml --namespace=cpu-example
```

상한이 100인 CPU에 요청량 100의 CPU를 2대 할당하려고 한다.

이 경우 노드 내 메모리가 부족하여 파드가 스케줄링 되지 못하고, PENDING가 계속 지속된다.

```
kubectl get pod cpu-demo-2 --namespace=cpu-example
NAME         READY     STATUS    RESTARTS   AGE
cpu-demo-2   0/1       Pending   0          7m

```

### CPU limit 을 지정하지 않으면 ?

컨테이너에 CPU 상한을 지정하지 않으면 다음 중 하나가 적용된다.

- The Container has no upper bound on the CPU resources it can use. The Container could use all of the CPU resources available on the Node where it is running.

- The Container is running in a namespace that has a default CPU limit, and the Container is automatically assigned the default limit. Cluster administrators can use a LimitRange to specify a default value for the CPU limit.

## References.

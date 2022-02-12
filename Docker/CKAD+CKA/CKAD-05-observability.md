# CKAD exercises

# 5. Observability (18%)

## 5.1 Liveness and readiness probes

- livenessProbe/readinessProbe 를 추가한 파드 생성

1. `initialDelaySeconds`: 5초 지연 후 컨테이너를 초기화
2. `periodSeconds`: 실행 실패 시, 10초 후에 재시도
3. `failureThreshold`: 3회 실패시 Restart

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
>>>>>>>>>> 추가하는 부분
    ports:
      - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
    livenessProbe:
      initialDelaySeconds: 5   # 5초 지연 후 초기화
      periodSeconds: 10        # 10초 후에 재시도
      failureThreshold: 3      # 3회 실패시 Restart
      exec:
        command:
        - ls
    readinessProbe:
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 5   # 5초 지연 후 초기화
      periodSeconds: 10        # 10초 후에 재시도
      failureThreshold: 3      # 3회 실패시 Restart
<<<<<<<<<<
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

- 리소스 확인

```
kubectl describe po nginx | grep -i liveness
kubectl describe po nginx | grep -i readiness
```

### 연습문제

1. namespaces=qa,alan,test,production 에 파드가 러닝 중이다.
2. 모든 파드의 livenessProbe 를 확인하여, `failed` 이 떨어진 pods 의 리스트를 `<namespace>/<podname>` 형태로 출력하라.

조건에 해당하는 pods 를 네임스페이스별로 확인한다

```
kubectl get ns # check namespaces
kubectl -n qa get events | grep -i "Liveness probe failed"
kubectl -n alan get events | grep -i "Liveness probe failed"
kubectl -n test get events | grep -i "Liveness probe failed"
kubectl -n production get events | grep -i "Liveness probe failed"
```

## 5.2 Logging

- 1초마다 현재 시간을 출력하는 명령어를 추가하여, pod 를 생성하고 log 모니터링하기

```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # follow the logs
```

## 5.3 Debugging

의도적으로 오류가 나는 명령어를 입력하고, `kubectl describe` 및 `kubectl get events` 명령어를 통해 오류 내용 디버깅한다.

```
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error (no such a directory)
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

```
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # will bring nothing! container never started
kubectl describe po busybox # in the events section, you'll see the error
# also...
kubectl get events | grep -i error # you'll see the error here as well
kubectl delete po busybox --force --grace-period=0
```

- 노드의 CPU/memory 사용량 확인

`kubectl top` 명령어로 파드 메트릭을 가져오기 위해서는, metric-server 실행 중에 있어야 한다.

(Metric server: 클러스터에서 리소스 사용량 데이터의 집계자 역할. 각 노드에 설치된 kubelet 을 통해서 노드 및 파드의 CPU, Memory 의 사용량 Metric 을 수집한다.)

```
kubectl top nodes
```

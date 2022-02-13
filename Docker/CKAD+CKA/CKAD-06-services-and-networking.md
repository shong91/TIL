# CKAD exercises

# 6. Services and Networking (13%)

- 80 포트로 노출하는 nginx 파드 생성

`--expose` 플래그를 추가하면 pod 와 svc 가 함께 생성된다.

```
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
```

- 리소스 확인

```
kubectl get svc nginx # services
kubectl get ep        # endpoints
kubectl run busybox --rm --image=busybox -it --restart=Never -- sh
wget -O- ${IP}:80
```

- 서비스 type 변경하기

서비스는 default: ClusterIP 로 생성되며, `kubectl edit svc` 명령어로 직접 변경할 수 있다.

- 8080 포트를 노출하는 deployment 생성하고 라벨 추가하기

```
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
kubectl label deployment foo --overwrite app=foo
```

- 8080 포트를 타겟 포트로 하는 서비스를, 6262 포트로 노출시켜 생성

```
kubectl expose deploy foo --port=6262 --target-port=8080
kubectl get service foo
kubectl get endpoints foo
```

### 연습문제

1. 80 포트를 노출하고 replica=2 인 nginx deployment 생성
2. networkPolicy를 생성하여 'access:granted' 라벨이 붙은 파드만 deployment 에 접근 가능하도록 정책 적용

```
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80
```

아래는 networkPolicy 의 내용이다. (policy.yaml)

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # pick a name
spec:
  podSelector:
    matchLabels:
      app: nginx # selector for the pods
  ingress: # allow ingress traffic
  - from:
    - podSelector: # from pods
        matchLabels: # with this label
          access: granted
```

NetworkPolicy 를 생성한 후, 올바르게 생성 되었는지 확인한다.

`matchLabels.access=granted` 의 내용에 따라, 아래와 같이 `--labels=access=granted` 플래그가 있어야만 올바르게 동작하여야 한다.

```
# make sure that your cluster's network provider supports Network Policy (https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#before-you-begin)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2                          # This should not work. --timeout is optional here. But it helps to get answer more quickly (in seconds vs minutes)
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  # This should be fine
```

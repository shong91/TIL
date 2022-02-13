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

## 연습문제

1. 80 포트를 노출하고 replica=2 인 nginx deployment 및 type=LoadBalancer인 서비스 생성
2. 8080 포트를 노출하고 replica=2 인 apache deployment 및 type=LoadBalancer인 서비스 생성
3. nginx ingress controller 배포
4. path: /nginx -> nginx svc 로, /apache -> apache svc 로 리다이렉트하도록 인그레스 서비스 생성

```
kubectl create deployment nginx-lb --image=nginx:latest --replicas=2
kubectl expose deployment nginx-lb --port=80 --target-port=80 --type=LoadBalancer
kubectl describe svc nginx-lb
# curl ${EXTERNAL_IP} and check nginx index page

kubectl create deployment apache-lb --image=bitnami/apache:latest --replicas=2
kubectl expose deployment apache-lb --port=8080 --target-port=8080 --type=LoadBalancer # Replace by NodePort if you don't have a LoadBalancer provider
kubectl describe svc apache-lb
# curl ${EXTERNAL_IP} and check apache index page
```

web-ingress.yaml:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: nginx-or-apache.com
    http:
      paths:
      - pathType: Prefix
        path: /nginx
        backend:
          service:
            name: nginx-lb
            port:
              number: 80
      - pathType: Prefix
        path: /apache
        backend:
          service:
            name: apache-lb
            port:
              number: 8080
```

nginx ingress controller 를 배포한 뒤,

```
# If using metallb or cloud deployment
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/cloud/deploy.yaml
# If using NodePort
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/baremetal/deploy.yaml
```

web-ingress.yaml 파일을 배포한다.

```
kubectl apply -f web-ingress.yaml
kubectl describe ingress web-ingress

...
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  nginx-or-apache.com
                       /nginx    nginx-lb:80 (10.244.1.30:80,10.244.2.32:80)
                       /apache   apache-lb:8080 (10.244.1.32:8080,10.244.2.35:8080)
...
```

의도한 대로 리다이렉트 되는지 확인하기 위해, busybox pod 를 생성한 뒤 두 서비스의 상태를 확인해본다.

```
kubectl run busybox --image=busybox --rm -it --restart=Never -- sh
# nslookup apache-lb
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	apache-lb.default.svc.cluster.local
Address: 10.101.245.9

# nslookup nginx-lb
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	nginx-lb.default.svc.cluster.local
Address: 10.108.72.239
```

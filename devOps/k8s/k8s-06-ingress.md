# Ingress
![img_01](https://kubetm.github.io/img/practice/intermediate/Ingress%20with%20Loadbalancing%20and%20Canary%20Concept%20for%20Kubernetes.jpg)

## Ingress

- HTTP(S) 기반의 L7 로드밸런싱 기능을 제공하는 컴포넌트. 
- 외부로부터 서비스 호출(요청) 시, ingress 를 걸쳐 Service 로 접근하게 된다. 
- url 라우팅 path 에 따라 service 에 맞추어 연결한다 

## Ingress Controller

- Ingress 에 정의된 규칙을 실제 동작하도록 하는 컨트롤러.
- 인그레스 리소스가 작동하려면, 클러스터는 실행 중인 인그레스 컨트롤러가 반드시 필요하다.
- 대표적으로 AWS, GCE, nginx Ingress Controller 가 있다. 


## Ingress 적용하기 

1. Ingress-controller 적용 (https://github.com/kubernetes/ingress-nginx)

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/baremetal/deploy.yaml
```

`--namespace=ngress-nginx` 에 ingress controller 와 관련된 리소스를 생성하였다. 

서비스를 확인해보면, `type=NodePort` 로 서비스가 노출되었음을 확인할 수 있다.

ingress를 통해 노출되는 서비스는 {DOMAIN(ADDRESS IP)}:{INGRESS_CONTROLLER_PORT} 로 노출된다.

```
$ kubectl get svc --namespace=ingress-nginx
```

2. Ingress 생성 

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com       #  DNS 주소를 설정 시 이와 같이 작성하며, 없을 시에는 host 부분을 삭제. 
    http:
      paths:
      - path: /foo
        backend:
          serviceName: svc1
          servicePort: 8000
      - path: /bar
        backend:
          serviceName: svc2
          servicePort: 9000
```

```
$ kubectl apply -f ingress.yaml --namespace=ingress-nginx
$ kubectl get ingress 
```

정상적으로 배포 되었는지 확인한다. 
host 를 적어주지 않았다면 `*` 으로 표시되며, ADDRESS 값은 배포가 완료되어야 노출되니 약간 시간이 필요할 수 있다.


3. Pod/Service 생성 

이제 인그레스 규칙에 맞추어 서비스를 생성해본다. 
간단하게 `image:nginx` 인 파드를 생성한 뒤, [{`name=svc1`, `port=8000`}, {`name=svc2`, `port=9000`}] 로 노출하는 두 개의 서비스를 각각 생성하였다. 

외부와 통신하여야 하므로 타입은 ClusterIP 가 아닌 `NodePort` 로 생성 !

```
$ kubectl run nginx1 --image=nginx:latest --namespace=ingress-nginx
$ kubectl expose pod nginx1 --name=svc1 --port=8000 --type=NodePort --namespace=ingress-nginx

$ kubectl run nginx2 --image=nginx:latest --namespace=ingress-nginx
$ kubectl expose pod nginx2 --name=svc2 --port=9000 --type=NodePort --namespace=ingress-nginx
```

결과 확인 시 차이를 명확하게 보기 위해서, 각 파드의 컨테이너로 접근하여 index.html 을 살짝 수정해주었다. 

```
$ kubectl exec nginx1 -it --namespace=ingress-nginx -- /bin/bash
# echo This is from svc1! > /usr/share/nginx/html/index.html

$ kubectl exec nginx2 -it --namespace=ingress-nginx -- /bin/bash
# echo This is from svc2! > /usr/share/nginx/html/index.html
```

4. 결과 확인 

이제 인그레스 규칙이 제대로 적용되었는지 확인할 차례다. 

인그레스를 생성할 때, host DNS 를 설정하였다면, `/etc/hosts` 에서 IP:DNS ADDRESS 를 매핑해주어야 한다. (host 를 설정하지 않았다면 와일드카드(*) 로 생성되므로 별도 조치하지 않아도 된다. )


ingress-controller 서비스의 node-port로 도매인이 노출 되며, 설정해준 ingress 규칙 대로 서비스가 반응합니다.

svc1을 호출하기 위해서는 /foo 경로로, svc2를 호출하기 위해서는 /bar 경로로 요청을 보내야 한다. 

테스트를 위해 종료 시 자동으로 삭제되는 테스트용 파드를 생성하고, 
curl 명령어를 사용해 path 에 따라 어떤 서비스가 호출되는지 확인하였다. 

```
$ kubectl run nginx-curl --image=nginx --rm -it --restart=Never -- /bin/bash
# curl {INGRESS_ADDRESS(DNS)}:{INGRESS_CONTROLLER_SERVICE_PORT}/foo
This is from svc1! 
# curl {INGRESS_ADDRESS(DNS)}:{INGRESS_CONTROLLER_SERVICE_PORT}/bar
This is from svc2! 
```

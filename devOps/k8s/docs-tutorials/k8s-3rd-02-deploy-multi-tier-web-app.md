# Example: Deploying PHP Guestbook application with Redis

(https://kubernetes.io/docs/tutorials/stateless-application/guestbook/)

## Hands-on

### 목표: 간단한 multi-tier web application 빌드 & 배포

- backend: Redis Leader 실행
- backend: Redis Follower(2개) 실행
- frontend: guesbook service 실행
- scaling

### 실습

이 실습에서는 Redis 를 데이터베이스(backend) 로, PHP 를 frontend 로 사용할 것이다.

### 1. Redis Leader 생성

#### 1-1. Redis Deployment 생성 & 실행

Redis Deployment로 단일 복제본 Redis pod(redis-leader) 를 생성한다.

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: leader
        tier: backend
    spec:
      containers:
      - name: leader
        image: "docker.io/redis:6.0.5"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```

```
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-deployment.yaml
$ kubectl get pods
```

아래 명령어로 리더 파드의 로그를 확인할 수 있다. (계속 띄워두고 확인 가능)

```
$ kubectl logs -f deployment/redis-leader
```

#### 1-2. Redis leader Service 생성

guestbook application(frontend) 에서 데이터를 쓰는(write) 작업을 하기 위해서는 backend 인 Redis 와 통신하여야 한다.

Redis 파드로 트래픽을 프록시 하기 위해서는, 파드가 외부와 연결이 가능하도록 서비스를 생성하여야 한다.

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: v1
kind: Service
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: leader
    tier: backend
```

```
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-service.yaml
$ kubectl get service

```

이 매니페스트 파일은 이전에 정의된 레이블(redis-leader in Deployment) 과 일치하는 레이블 집합을 가진 redis-leader라는 서비스를 생성하므로, 서비스는 네트워크 트래픽을 Redis 파드로 라우팅한다.

### 2. Redis Followers 생성

트래픽을 효율적으로 관리하고 애플리케이션의 가용성을 높이기 위해, Redis followers/replica 를 추가할 수 있다.

여기서는 2개의 followers 를 추가한다.

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: follower
        tier: backend
    spec:
      containers:
      - name: follower
        image: gcr.io/google_samples/gb-redis-follower:v2
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```

```
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-deployment.yaml
$ kubectl get pods
```

Redis Leader 를 생성할 때와 마찬가지로, Redis followers 가 외부와 통신하기 위해 새로운 Service 를 구성해준다.

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    app: redis
    role: follower
    tier: backend
```

```
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-service.yaml
$ kubectl get service
  NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
  redis-follower   ClusterIP   10.100.68.22    <none>        6379/TCP   7m5s
  redis-leader     ClusterIP   10.100.223.40   <none>        6379/TCP   9m48s
```

앞서 만들어둔 redis-leader 와 redis-follower 가 서비스에 생성된 것을 확인할 수 있다.

### 3. guesbook frontend 실행 & 서비스 노출

여기까지 guestbook backend 구성을 마쳤고, 이제 guestbook 웹 서버를 구성할 것이다.

frontend는 PHP 를 사용하며, DB에 대한 요청 (read/write)에 따라 redis leader/follwer 와 통신하도록 할 것이다.

#### 3-1. Guestbook Frontend Deployment 생성 & 실행

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
        app: guestbook
        tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v5
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 80
```

```
$ kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
$ kubectl get pods -l app=guestbook -l tier=frontend
```

`app=guesbook`, `tier=frontend` 라벨이 붙은 파드의 리스트를 확인하면, 3개의 레플리카 파드가 생성된 것을 확인할 수 있다.

#### 3-2. frontend Service 생성

서비스의 기본 유형은 ClusterIP 이기 때문에 생성한 Redis 서비스는 컨테이너 클러스터 내에서만 접근할 수 있다. ClusterIP는 서비스가 가리키는 파드 집합에 대한 단일 IP 주소를 제공한다. 이 IP 주소는 클러스터 내에서만 접근할 수 있다.

게스트가 guestbook 접근할 수 있도록 하려면, 외부에서 볼 수 있도록 프론트엔드 서비스를 구성해야 한다. 그렇게 하면 클라이언트가 쿠버네티스 클러스터 외부에서 서비스를 요청할 수 있다.

그러나 쿠버네티스 사용자는 ClusterIP를 사용하더라도 `kubectl port-forward`를 사용해서 서비스에 접근할 수 있다.

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 참고: Google Compute Engine 또는 Google Kubernetes Engine 과 같은 일부 클라우드 공급자는 외부 로드 밸런서를 지원한다.
  # 클라우드 공급자가 로드 밸런서를 지원하고 이를 사용하려면 type : LoadBalancer의 주석을 제거해야 한다.
  # type: LoadBalancer

  ports:
    # the port that this service should serve on
  - port: 80
  selector:
    app: guestbook
    tier: frontend
```

```
$ kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
$ kubectl get services
  NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
  frontend         ClusterIP   10.100.47.176   <none>        80/TCP     14m
```

#### 3-3-1. 포트 포워딩하여 frontend service 확인

`kubectl port-forward` 명령어로 {로컬포트:서비스포트} 를 연결한다.

```
$ kubectl port-forward svc/frontend 8080:80
  Forwarding from 127.0.0.1:8080 -> 80
  Forwarding from [::1]:8080 -> 80
```

로컬의 8080 포트가 서비스의 80 포트로 포워딩되어, 로컬의 8080 포트를 사용하여 해당 서비스를 로드할 수 있게 되었다.

#### 3-3-2. LoadBalancer 를 사용하여 frontend service 확인

3-2에서 서비스를 배포할 시 `type:LoadBalancer` 를 설정하였다면,

guesbook application 의 external-IP 주소를 확인하여 해당 주소로 서비스에 접근할 수도 있다.

```
$ kubectl get service frontend
  NAME       TYPE           CLUSTER-IP      EXTERNAL-IP        PORT(S)        AGE
  frontend   LoadBalancer   10.51.242.136   109.197.92.229     80:32372/TCP   1m
```

위와 같이 EXTERNAL-IP 를 확인하여, 해당 IP 주소로 서비스를 로드할 수 있다.

### 4. 서비스 확장/축소

서버가 디플로이먼트 컨트롤러를 사용하는 서비스로 정의되어 있으므로 필요에 따라 확장 또는 축소할 수 있다.

```
$ kubectl scale deployment frontend --replicas=5
$ kubectl get pods
```

3개 -> 5개로 파드의 수가 확장된 것을 확인할 수 있다.

물론 파드를 축소시키는 것도 가능하다.

```
$ kubectl scale deployment frontend --replicas=2
```

끝!

실습을 마치고 리소스를 지워주는 것을 잊지 말자!

특정 라벨이 붙은 리소스를 지우고 싶다면 `-l` 옵션을,

```
$ kubectl delete deployment -l app=redis
```

모든 리소스를 한 번에 지우려면 `--all` 옵션을 사용한다.

```
$ kubectl delete all --all  # 현재 네임스페이스에서 한 번에 모든 리소스를
$ kubectl delete all --all -n {namespace} # 특정 네임스페이스의 모든 리소스를 지우려면
```

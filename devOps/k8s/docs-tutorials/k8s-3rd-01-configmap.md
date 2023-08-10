# Configuring Redis using a ConfigMap

(https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/)

## what is ConfigMap ?

- Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.
- A ConfigMap allows you to **decouple environment-specific configuration from your container images**, so that your applications are easily portable.
- 컨테이너에서 필요한 환경 설정 내용을 컨테이너와 분리하여 제공하기 위한 기능
- ConfigMap을 컨테이너와 분리함으로써, 하나의 동일한 컨테이너를 가지고 dev, staging, prod용으로 모두 사용할 수 있다

## Hands-on

### 목표: ConfigMap 사용방법 숙지

- Redis pod 배포에 사용할 configmap 생성
- 생성한 configmap 을 마운트하여 Redis pod 생성
- 정상 적용 여부 확인

### 실습

### 1. ConfigMap 파일 생성

```
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

### 2. Configmap 및 Redis Pod 실행

Redis pod 의 내용을 살펴보면 아래와 같이, example-redis-config을 configMap 으로 사용하도록 설정되어 있다.

```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf

```

```
$ kubectl apply -f example-redis-config.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
$ kubectl get pod/redis configmap/example-redis-config
```

로컬에 생성한 ConfigMap 파일(`-f`)과 허브에서 받아오는 redis pod 를 실행한다. (ref) [kubectl create vs kubectl apply 차이점](https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create))

생성된 configMap 의 내용을 살펴보면, 아직 redis-config 키에 설정한 것이 없기 때문에 비어 있는 모습을 확인할 수 있다.

```
$ kubectl describe configmap/example-redis-config

Name:         example-redis-config
Namespace:    syhong
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
```

`kubectl exec` 명령어를 사용하여 파드 안으로 들어가서, `redis-cli` 를 실행시킨 후
최대 메모리 및 메모리 정책을 확인한다.

```
$ kubectl exec -it redis -- redis-cli

127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
127.0.0.1:6379> exit
```

모두 default 상태임을 확인할 수 있다.

### 3. Configmap 업데이트 후 Redis Pod 에 적용 여부 확인

이제 내가 설정하고 싶은 내용으로 configMap 을 수정 후, Redis 파드에 적용시킬 것이다. (example-redis-config.yaml)

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
          maxmemory 2mb
          maxmemory-policy allkeys-lru
~
```

이렇게 내용을 추가하고, 다시 configmap 을 실행한다.

`kubectl describe` 명령어로 다시 확인해보면, redis-config 의 내용이 추가된 것을 확인할 수 있다.

```
$ kubectl apply -f example-redis-config.yaml
$ kubectl describe configmap/example-redis-config
```

다시 redis-cli 를 실행해 내용이 반영되었는지 확인해본다.

```
$ kubectl exec -it redis -- redis-cli

127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
127.0.0.1:6379> exit
```

아직 default 상태로 남아있는데, 변경 내용을 적용하기 위해서는 Pod를 재시작하여야 하기 때문이다.

기존에 생성한 파드를 삭제하고, 다시 생성 & 실행해본다.

```
$ kubectl delete pod redis
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml

```

이제 다시 configuration value 들을 확인해보면,
`example-redis-config.yaml` 에 적용한 내용이 정상 반영됨을 확인할 수 있다.

```
$ kubectl exec -it redis -- redis-cli

127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
127.0.0.1:6379> exit
```

실습을 마치고 생성한 리소스를 삭제해준다.

```
kubectl delete pod/redis configmap/example-redis-config
```

이렇게 configMap 을 사용하여 환경 설정 내용을 컨테이너와 분리하여 관리하고,

하나의 컨테이너를 서로 다른 환경에서 사용 할 수 있다.

### Reference.

1. https://kubernetes.io/docs/concepts/configuration/configmap/

# CKAD exercises

# 4. Configuration (18%)

## 4.1 ConfigMap

- configmap 생성하기

1. `--from-literal`: 리터럴로 생성하기
2. `--from-file`: txt 파일로 생성하기
3. `--from-env-file`: env 파일로 생성하기

key 를 부여하고 싶다면, [1~3의 플래그 + 사용하고자 하는 키 명칭 + 파일] 형태로 가능하다.

```
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
kubectl create cm configmap2 --from-file=config.txt
kubectl create cm configmap3 --from-env-file=config.env
kubectl create cm configmap4 --from-file=special=config4.txt

```

- configmap 을 적용하여 파드 생성

configmap 의 특정 value 만 사용하고자 하는 경우 (configmap/options 는 이미 생성하였다고 가정)

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
>>>>>>>>> 추가하는 부분
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
<<<<<<<<<
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

configmap 전체를 적용하는 경우 (configmap/anotherone 는 이미 생성하였다고 가정)

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
>>>>>>>>> 추가하는 부분
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: anotherone # the name of the config map
<<<<<<<<<
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

configmap 을 볼륨 마운트하여 사용할 경우 (configmap/cmvolume 는 이미 생성하였다고 가정)

볼륨을 생성하고 컨테이너에 마운트 할 때에는,
`pod.spec.volumes.name` 와 `pod.spec.containers.volumeMounts.name` 이 일치하여야 한다.

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

## 4.2 SecurityContext

SecurityContext 는 파드 또는 컨테이너의 권한 부여, 환경설정, 접근제어 등의 제어하는 기능을 제공한다.

Container 프로세스들이 사용하는 사용자(runAsUser)와 그룹(fsGroup), 가용량(capabilities), 권한 설정, 보안 정책(SELinux/AppArmor/Seccomp)을 설정하기 위해 사용된다.

[Security Context를 사용해서 제어가능한 목록](https://ikcoo.tistory.com/67)

1. userID=101 으로 실행되는 파드 생성
2. capabilities="NET_ADMIN", "SYS_TIME" 으로 하는 파드 생성

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
>>>>>>>> 추가하는 부분
  securityContext: # insert this line
    runAsUser: 101 # UID for the user
    capabilities: # and this
      add: ["NET_ADMIN", "SYS_TIME"] # this as well
<<<<<<<<
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

### 4.3 Requests and limits

- requests, limits 를 가지는 파드 생성하기

`kubectl run ... --request=${value} --limits=${value}` 의 형식으로도 가능하지만, 이는 비권장된다.

`kubectl run --dry-run=client -o yaml` 와 `kubectl set resources` 명령어를 결합하여 수정된 yaml 파일을 생성, 배포하는 것을 권장한다.

````
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi --local -o yaml > nginx-pod.yml```

````

### 4.4 Secrets

- Secrets 생성하기

1. `--from-literal`: 리터럴로 생성하기
2. `--from-file`: txt 파일로 생성하기
3. `--from-env-file`: env 파일로 생성하기

```
kubectl create secret generic mysecret --from-literal=password=mypass
kubectl create secret generic mysecret2 --from-file=username
```

- 리소스 확인

secret 의 내용은 자동으로 인코딩 되므로, `-d` (맥의 경우 `-D`) 플래그를 추가하여 디코딩한 값을 확인한다.

```
kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d
```

- yaml 파일에 적용하여 파드 생성

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
    env: # our env variables
    - name: USERNAME # asked name
      valueFrom:
        secretKeyRef:
          name: mysecret2
          key: username
<<<<<<<<<<
>>>>>>>>>> 볼륨에 올려 사용할 경우
    volumeMounts:
      - name: secret-volume
        mountPath: /etc/secret-volume
        readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret2
<<<<<<<<<<
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

위의 yaml 파일로 파드를 배포한 후, env USERNAME 의 내용을 확인해본다.

```
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # will show 'admin'
```

### 4.5 ServiceAccounts

각 네임스페이스에 대해 default service account 가 존재하며, 필요할 경우 sa 를 추가할 수 있다.

```
kubectl get sa --all-namespaces
kubectl create sa myuser
kubectl get sa myuser -o yaml > sa.yaml
```

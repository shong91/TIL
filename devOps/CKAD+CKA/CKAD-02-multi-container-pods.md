# CKAD exercises

# 2. Multi-container Pod (10%)

## 연습문제 1

1. image=busybox, command="echo hello; sleep 3600" 를 사용하는 두 개의 컨테이너를 하나의 파드로 생성
2. 두 번째 컨테이너 쉘에 접근하여 명령어 "ls" 출력

가장 간단한 방법은, `dry-run:client` 플래그로 파드는 생성하지 않되 output yaml 파일을 생성한 뒤, 해당 yaml 파일을 조건에 맞게 수정한 뒤 파드를 생성하는 것이다.

```
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox
    resources: {}
>>>>>>>> 추가하는 부분
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
<<<<<<<<
status: {}
```

위와 같이 conatainers[].args[0], conatainers[].args[1]의 내용을 추가 후, 파드를 생성한다.

```
kubectl create -f pod.yaml
```

파드가 가진 두 개의 컨테이너 중, 두번째 컨테이너의 쉘에 접근하여 명령어를 날리고 종료한다.

```
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
quit

# or do the above with just an one-liner
kubectl exec -it busybox -c busybox2 -- ls
```

## 연습문제 2

1. nginx 컨테이너로 파드를 생성 후 port=80 으로 노출시킨다.
2. busybox 를 init container 로 추가한다.

- busybox 는 "wget -O /work-dir/index.html http://neverssl.com/online" 페이지를 다운로드하는 명령어를 포함한다.

3. type=emptyDir 의 볼륨을 생성하여 nginx, busybox 컨테이너에 마운트한다.

- mountPath in nginx: "/usr/share/nginx/html"
- mountPath in busybox: "/work-dir"

4. 완료 후, 생성된 파드의 IP 로 busybox 파드를 생성하고, 해당 파드의 아이피를 확인하여 "wget -O- ${IP}" 를 실행한다

역시 `dry-run:client` 플래그를 활용하여 output yaml 파일을 생성/수정하자.

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: web
spec:
>>>>>>>> 추가하는 부분 (조건 2, 3-2)
  initContainers:
  - args:
          - /bin/sh
          - -c
          - wget -O /work-dir/index.html http://neverssl.com/online
    image: busybox
    name: busybox
    volumeMounts:
    - name: vol
      mountPath: /work-dir
<<<<<<<<
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
>>>>>>>> 추가하는 부분 (조건 3-1)
    volumeMounts:
    - name: vol
      mountPath: /usr/share/nginx/html
<<<<<<<<
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
>>>>>>>> 추가하는 부분 (조건 3)
  volumes:
  - name: vol
    emptyDir: {}
<<<<<<<<
status: {}
```

파드를 생성하고, IP 를 가져와

```
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP (192.168.83.35)
kubectl get po -o wide

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- 192.168.83.35 "
```

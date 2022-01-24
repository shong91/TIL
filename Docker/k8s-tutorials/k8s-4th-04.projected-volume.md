# Configure a Pod to Use a Projected Volume for Storage

https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/

## what is projected volume ?

Projected Volume 은 여러 기존 볼륨 소스를 동일한 디렉토리에 매핑한다.

모든 볼륨 소스는 파드와 동일한 네임스페이스에 존재하여야 하며, 아래 유형의 볼륨을 projected 할 수 있다.

- secret
- configMap
- downwardAPI
- serviceAccountToken volumes

## Hands-on

### 목표:

- Projected Volume 에 볼륨 마운트 하여 사용

### 실습

### 1. Secret 생성

username.txt, password.txt 파일을 각각 생성하고, 해당 파일로 Secret 을 생성한다.

```
# Create files containing the username and password:
echo -n "admin" > ./username.txt
echo -n "1f2d1e2e67df" > ./password.txt

# Package these files into secrets:
kubectl create secret generic user --from-file=./username.txt
kubectl create secret generic pass --from-file=./password.txt

```

```
k get secrets
```

user, pass 라는 이름으로 Secret 이 생성된 것을 확인할 수 있다.

### 2. Projected Volume 생성

1 에서 생성한 Secret 을 사용하는 projected volume 을 생성한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: test-projected-volume
spec:
  containers:
  - name: test-projected-volume
    image: busybox
    args:
    - sleep
    - "86400"
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: user
      - secret:
          name: pass
```

```
kubectl apply -f https://k8s.io/examples/pods/storage/projected.yaml
pod/test-projected-volume created
```

파드가 정상적으로 생성되고, running 되고 있는지 아래 커맨드를 동해 지켜본다.

```
kubectl get --watch pod test-projected-volume

```

### 3. 생성된 Projected Volume 의 내용 확인

다른 터미널을 열어, 생성한 파드 내부로 들어가 secret 볼륨이 정상적으로 마운트 되어 사용되고 있는지 확인한다.

```
kubectl exec -it test-projected-volume -- /bin/sh
```

```
ls /projected-volume/
password.txt  username.txt
```

## References.

https://kubernetes.io/ko/docs/concepts/storage/volumes/

# CKAD exercises

# 6. State Persistence (8%)

- PV, PVC 생성하여 파드에 붙이기

* pv.yaml

```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

- pvc.yaml

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

- 파드 yaml 파일을 아래와 같이 수정한 뒤 배포한다.

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 # don't forget to change the name during copy paste, must be different from the first container's name!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
>>>>>>>>>>>>> 일반 볼륨을 사용할 경우(emptyDir)
  volumes: #
  - name: myvolume #
    emptyDir: {} #
<<<<<<<<<<<<<
>>>>>>>>>>>>> PVC 를 생성하여 볼륨에 붙인 경우
   volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
<<<<<<<<<<<<<
```

PVC 볼륨으로 연결되어 있기 때문에, 1번째 컨테이너에서 내용을 수정하더라도 2번째 컨테이너에 데이터가 그대로 보존된다.

```
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
kubectl exec busybox2 -- ls /etc/foo # will show 'passwd'
```

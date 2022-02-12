# CKAD exercises

# 3. Pod design (20%)

## 3.1 Labels and annotations

### Labels

- `app=v1` 을 라벨로 갖는 파드 ngix1,nginx2,nginx3 생성

```
for i in `seq 1 3`; do kubectl run nginx$i --image=nginx -l app=v1 ; done
```

- 라벨 확인

```
kubectl get po --show-labels  # 파드의 모든 라벨 확인
kubectl get po -L app         # 파드의 특정 라벨 확인 (app)
# or
kubectl get po --label-columns=app
```

- 특정 라벨값의 파드 조회

```
kubectl get po -l app=v2
```

- 라벨 수정/삭제

```
kubectl label po nginx2 app=v2 --overwrite
kubectl label po nginx2 app-
```

- 노드에 라벨 적용

```
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
```

### Annotation

- 어노테이션 생성

```
kubectl annotate po nginx{1..3} description='my description'
```

- 어노테이션 확인

```
kubectl annotate pod nginx1 --list # 파드의 모든 어노테이션 확인
# or
kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description  # 파드의 특정 어노테이션 확인 (description)
```

- 특정 어노테이션값 확인

- 수정/삭제

```
kubectl annotate po nginx1 description='update' --overwrite
kubectl annotate po nginx{1..3} description-
```

## 3.2 Deployments

- deployment 생성

```
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

- 생성한 리소스 확인

```
kubectl get deploy nginx -o yaml
kubectl get rs -l app=nginx
kubectl get po -l app=nginx
```

- rollout 상태 확인

```
kubectl rollout status deploy nginx
```

- rollout 기록 확인

  특정 리비전의 기록을 확인하고자 한다면 `--revision=4` 플래그를 추가한다.

```
kubectl rollout history deploy nginx
```

- rollout undo/pause/resume

  undo 시, 특정 리비전으로 되돌리고 싶다면 `--to-revision=2` 플래그를 추가한다.

```
kubectl rollout undo deploy nginx
kubectl rollout pause deploy nginx
kubectl rollout resume deploy nginx
```

- scaling

```
kubectl scale deploy nginx --replicas=5
```

- auto scaling

```
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
kubectl get hpa nginx
```

## 3.3 Job

Job은, 하나 이상의 파드를 지정하고, 지정된 수의 파드를 성공적으로 실행하도록 하는 설정이다.
노드의 장애로 파드가 정상 실행되지 않았을 경우, job 은 새로운 파드를 자동으로 생성한다.

- commang args 를 "perl -Mbignum=bpi -wle 'print bpi(2000)'" 으로 하는 job 생성

```
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

- 생성한 리소스 확인

```
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl logs job/pi
kubectl delete job pi
```

- job 스케쥴링

  `--dry-run=client` 플래그를 추가하여 output yaml 파일을 먼저 생성한 뒤, 스케쥴링 조건에 맞추어 yaml 파일을 수정 후 배포한다.

```
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
```

1. `activeDeadlineSeconds`: 실행 30초 후 자동으로 종료
2. `completions`: 5개 이상 정상 종료될 경우 job을 성공처리 (완료) 한다.
3. `parallelism`: 한번에 3개의 파드를 병렬적으로 실행한다.

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
>>>>>>>> 추가하는 부분
  activeDeadlineSeconds: 30
  completions: 5
  parallelism: 3
<<<<<<<<
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

## 3.4 CronJob

- schedule="\*/1 \* \* \*" 의 스케쥴을 가지는 cronjob 생성

```
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

- 리소스 확인

```
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
```

- job 스케쥴링

  `--dry-run=client` 플래그를 추가하여 output yaml 파일을 먼저 생성한 뒤, 스케쥴링 조건에 맞추어 yaml 파일을 수정 후 배포한다.

```
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
```

1. `startingDeadlineSeconds`: 스케쥴 되어있는 시간이 지난 후, 17초가 지나는 cronjob을 종료시킨다. (ex 잡이 스케쥴 된 시간을 잊었다고 (오류) 판단)
2. `activeDeadlineSeconds`: 잡이 성공적으로 수행되었으나, 소요된 시간이 12초가 넘었다면 cronjob을 종료시킨다.

```
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busybox
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busybox
    spec:
>>>>>>>>> 추가하는 부분
      startingDeadlineSeconds: 17
      activeDeadlineSeconds: 12
<<<<<<<<<
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: busybox
            resources: {}
          restartPolicy: OnFailure
  schedule: '* * * * *'
istatus: {}
```

# Statefulset

Statefulset 은 Pod를 Scaling Up/Down 또는 Deploy 할 때, 각 파드의 기존 스펙을 유지시켜준다. 즉, Statefulset 은 sticky 값을 유지하고 생성할 때 기존 값을 사용하여 그대로 만들어준다. 

- 안정적이고 고유한 네트워크 식별자 사용
- 안정적이고 지속적인 스토리지 사용 
- 안정적인 파드 배치 및 확장, 자동 롤링 업데이트 설정 

- Statefulset app. web server: 모든 application 이 같은 역할을 함 (단순 복제), 하나의 볼륨에 모두 연결 가능 
- Statefulset app. database: 각 application 마다 다른 역할을 하며, 애플리케이션 특징에 맞게 트래픽이 분산됨. 애플리케이션마다 각각 다른 볼륨 사용

![img_01](https://kubetm.github.io/img/practice/intermediate/StatefulSet%20with%20Stateless,%20Stateful%20Application%20for%20Kubernetes.jpg)

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-db
spec:
  replicas: 1
  selector:
    matchLabels:
      type: db
  template:
    metadata:
      labels:
        type: db
    spec:
      containers:
      - name: container
        image: kubetm/app
      terminationGracePeriodSeconds: 10 # 강제 종료까지 대기하는 시간
    volumeClaimTemplates: # PVC 설정을 저장하는 부분; 동적 생성 & 자동으로 연결 
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi 
```

## References.
https://freedeveloper.tistory.com/344


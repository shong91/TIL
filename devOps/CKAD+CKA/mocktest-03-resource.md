# Resources
kubectl explain pods --recursive

## static pod를 구성하기
- **static pod를 생성** Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.
- **kubelet에서 static pod 위치 지정** Use /etc/kubernetes/manifests as the Static Pod path for example
```
## create pod 
kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml
## Copy the contents of this file or use scp command to transfer this file from controlplane to node01
scp static.yaml node01:/root/
## Perform SSH to get into node01
ssh node01
## create static pod directory 
mkdir -p /etc/kubernetes/manifests
## complete path to the staticPodPath field in the kubelet config.yaml file. *kubelet에서 static pod 위치 지정* 

  vi /var/lib/kubelet/config.yaml
  ...
  staticPodPath: /etc/kubernetes/manifests
  ...

## move/copy the static.yaml to static path
cp /root/static.yaml /etc/kubernetes/manifests/
## exit from node01 and check if Pod nginx-critical-node01 is up and running
exit
kubectl get pods 
 ```


## Pod를 특정 Node에 Schedule 하기
- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

- affinity는 노드 셋을 (기본 설정 또는 어려운 요구 사항으로) 끌어들이는 파드의 속성이다. 
- taint 는 그 반대로, 노드가 파드 셋을 제외할 수 있다.
- toleration 은 파드에 적용되며, 파드를 일치하는 테인트가 있는 노드에 스케줄되게 하지만 필수는 아니다.
1. taint
```
kubectl taint nodes node1 key1=value1:NoSchedule
```

2. tolation
```
tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

3. nodeSelector
```
nodeSelector:
  disktype: ssd     # 문제에서 요구하는 레이블
```
4. affinity
```
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
```
## Pod Expose 시키기
```
k expose pod ${pod-name} --port=8080
```

# NetworkPolicy를 사용해서 한 namespace에서 다른 namespace로 통신 가능하게하고, 그 외의 통신 막기
- https://kubernetes.io/docs/concepts/services-networking/network-policies/

정책을 적용할 pod와 namespace에 label링을 하기
to(pod label)와 from(namespace label) 구분, 무슨 포트만 허용할건지도 명시

## Resource Limit(CPU, Memory) 걸어서 Pod 만들기

## clusterrole, clusterrolebinding 하기
- 롤(Role)은 특정 api나 리소스에 대한 권한들을 명시해둔 규칙들의 집합

1. 롤(Role)
- 그 롤이 속한 네임스페이스 한 곳에만 적용 

2. 클러스터롤(ClusterRole)
- 특정 네임스페이스에 대한 권한이 아닌 클러스터 전체에 대한 권한을 관리
- 네임스페이스에 한정되지 않은 자원 및 api들에 대한 권한을 지정
- 노드/엔드포인트에 대한 권한 관리

## two containers pod 만들기
## pv 만들기 (hostPath): pvc 만들고 pod 연결하기. 그리고 pvc resize 하기
pvc 용량 증가시키고 기록하기

   (생성시킨 후 pvc 대상으로 kubectl edit , patch를 쓰라는데.. 이거에 대해 기록하는 명령어가 있는건가..?)
## Ingress 만들기
-  https://kubernetes.io/docs/concepts/services-networking/ingress/
- 확인용 파드 생성 (--rm -it --restart=Never)
- curl <Internal-IP>/hi 통신되는지 확인
- nslookup 

# Node, Monitoring, Troubleshooting 

## Troubleshooting - Not Ready인 node 고치기
kube-apiserver나 kubelet 확인

```
# node2가 NotReady 로 조회됨 
kubectl get node 
# node2로 접속 
ssh node2 

# Make sure kubelet is running
systemctl status kubelet 
# restart kubelet
systemctl restart kubelet 

exit 
# 일정 시간 이후 정상 상태 확인 
kubectl get node
```

and more...
- https://stackoverflow.com/questions/47107117/how-to-debug-when-kubernetes-nodes-are-in-not-ready-state

## 기존 스케쥴링된 node가 not ready인 경우의 pod을 다른 node에 스케쥴링하기

not ready인 node를 uncordon한 후, 해당 pod을 재생성

(그 전에 node와 pod의 labeling을 확인하고 해당 노드의 label을 가지고 있는 pod를 지웠다 재생성)


## deploy scale-out 하고, scale-out이 안되는 원인 찾아 고치기
```
kubectl scale deploy nginx-deploy --replicas=3
```
스케일링 되지 않고 여전히 1개의 레플리카만 구동 중이다. 

kube-system 네임스페이스의 controller-manager 이라는 파드에서 replicaset 의 스케일링을 관리하는데, 해당 파드에 문제가 생겼을 확률이 높다. 
```
kubectl get pods -n kube-system
...
kube-contro1ler-manager-controlplane   0/1     ImagePullBackOff   0          15m
...
```

파드에 문제가 생겼다. 내용을 확인해보니 contro1ler 라는 오타가 존재하여서 파드가 구동되지 않은 것을 확인하였다. 
오타가 난 부분을 일괄 수정한 뒤, 파드가 정상 러닝되는 것을 확인한다. 
```
sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
```

다시 스케일링 해보면 정상적으로 작동한다.


#  테인트(Taints)가 적용된 노드 확인하기
- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
```
kubectl describe nodes | grep -i taint
```

## 노드 갯수 카운트 하기
```
kubectl get node | grep 조건
```

## 특정 레이블을 가진 파드 중 CPU 사용률 높은 pod 이름 알아내기
```
kubectl top pod -l {LABEL}
```

## sidecar (Logging)
- https://kubernetes.io/ko/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/ 
- https://kim-dragon.tistory.com/148

기존에 파드가 이미 존재하고, sidecar 형태로 pod를 띄워 로깅 아키텍처를 구성하라는 문제였다.
kubectl logs {파드명} 


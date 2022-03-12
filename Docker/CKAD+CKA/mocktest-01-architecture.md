- 17문제 2시간 
- 66/100 (PASS)
- 7점짜리가 5문제 정도로 생각보다 많이 나왔던 것 같고, 5점짜리는 4문제 정도, 나머지는 4점짜리 문제가 나왔던 것 같습니다.


# Cluster Architecture

## kubelet의 관련 명령어 
```
journalctl -u kubelet -f    # kubelet 로그 확인하기
service kubelet status   ## kubelet의 상태 확인하기
```

## kubeadm을 통해 클러스터 구성하고, 업그레이드 하기
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/
- https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- https://daintree.tistory.com/16?category=886935
- https://kim-dragon.tistory.com/65

**업그레이드 규칙!** 
1. 먼저 마스터 노드를 업그레이드 한 후 워커 노드를 업그레이드한다.
2. 모든 노드는 업그레이드 전 drain, 업그레이드 후 uncordon을 실행한다.
  - kubectl drian {노드명}: 노드에서 실행 중인 파드를 다른 노드로 옮기고 새로운 파드의 스케줄링을 막는다. 
    업그레이드 중인 노드 내부에는 데몬셋을 제외한 파드가 없어야 하고 새로운 파드 생성 시 업그레이드 중인 노드에 스케줄링이 되면 안되기 때문에 업그레이드 전 해당 명령을 수행해야 한다.
  - kubectl uncordon {노드명}: 해당 노드의 스케줄링을 다시 허가한다. 해당 명령은 단순히 스케줄링만 허가하기 때문에, drain시 다른 노드로 배치되었던 파드가 다시 돌아오지는 않는다. 
3. 노드의 kubeadm을 먼저 업그레이드 한 후
4. kubelet과 kubectl을 업그레이드 한다. 
  - kubectl은 단순 클라이언트 도구이기 때문에 워커 노드에 없는 경우도 있다. 이 경우 kubelet만 업그레이드 한다.


1. 마스터 노드 drain
```
kubectl drain controlplane --ignore-daemonsets
```
2. 마스터 노드의 kubeadm 업그레이드
```
# 현재 버전 확인
kubeadm version

# 업그레이드할 버전 결정
apt update
apt-cache madison kubeadm

# 목록에서 최신 버전(1.23)을 찾는다
# 1.23.x-00과 같아야 한다. 여기서 x는 최신 패치이다.
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.x-00 && \
apt-mark hold kubeadm

# 다운로드하려는 버전이 잘 받아졌는지 확인한다.
kubeadm version

# 클러스터를 업그레이드할 수 있는지를 확인하고, 업그레이드할 수 있는 버전을 가져온다. 
kubeadm upgrade plan

# 이 업그레이드를 위해 선택한 패치 버전으로 x를 바꾼다.
sudo kubeadm upgrade apply v1.23.x
```

3. 마스터 노드의 kubectl, kubelet 업그레이드
```
# kubelet, kubectl 업그레이드 
apt-mark unhold kubelet kubectl && \ apt-get update && apt-get install -y kubelet=1.19.0-00 kubectl=1.19.0-00 && \ apt-mark hold kubelet kubectl 

#kubelet 재시작 
sudo systemctl daemon-reload 
sudo systemctl restart kubelet 

#kubelet, kubectl 버전 확인 
kubelet --version
```

4. 마스터 노드 uncordon
```
kubectl uncordon controlplane
```

이하 워커 노드 업그레이드를 동일한 방식으로 진행한다. 

5. 워커 노드 drain
6. 워커 노드의 kubeadm 업그레이드
7. 워커 노드의 kubectl, kubelet 업그레이드(kubectl이 없을 경우 kubelet만 업그레이드)
8. 워커 노드 uncordon

## kubeadm을 통해 새로운 노드를 워커노드로 붙이기
- https://hiaurea.tistory.com/146?category=942239

## etcdctl을 통해 ETCD를 백업하고 복구하기
- https://zgundam.tistory.com/197

- 백업 
```
ETCDCTL_API=3 etcdctl snapshot save --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \     #  --cert=<cert-file>
    --key=/etc/kubernetes/pki/etcd/server.key \    # --key=<key-file> 
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \     # --cacert=<trusted-ca-file>
    <backup-file-location> # 백업을 저장할 경로/파일

```
- 복구

```
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 snapshot restore  <backup-file-location> # 백업을 저장할 경로/파일
```


# kubectl config current-context 

volume 맞음

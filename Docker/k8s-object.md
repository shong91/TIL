# k8s Architecture

쿠버네티스는 중앙(Master)에 API 서버와 상태 저장소를 두고, 각 서버(Node)의 에이전트(kubelet) 과 통신하는 구조.

- master: 전체 클러스터를 관리하는 서버
- node: 컨테이너가 배포되는 서버

**Master 구성 요소**

- API 서버 (kube-apiserver): 모든 요청을 처리하는 API 서버
- 분산 데이터 저장소 (etcd): key-value 저장소
- 스케쥴러, 컨트롤러 (kube-scheduler, kube-controller-manager, cloud-controller-manager)

**Node 구성 요소**

- kubelet: 노드에 할당된 파드의 생명주기 관리
- kube-proxy: 파드로 연결된 네트워크 관리
- 추상화: 컨테이너(containerd, rktm ,CRI-O) 런타임 지원

# 기본 오브젝트

## Pod

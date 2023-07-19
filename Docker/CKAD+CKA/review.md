## 시험 문제 복기

1. kubeadm을 통해 클러스터 업그레이드 하기

- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- master node 만 업그레이드 할 것

2. etcdctl을 통해 ETCD를 백업하고 복구하기

- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
- 백업, 복구 모두 해야함
- 백업은 /data/backup/backup.db 에, 복구는 /data/backup/backup-previous.db 에 해달라고 함
- 이런 케이스로 연습해본적이 없어서... 제대로 못했음

3. NetworkPolicy를 사용해서 한 namespace에서 다른 namespace로 통신 가능하게하고, 그 외의 통신 막기

- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- `namespace=foo` 인 모든 파드에 대하여, `namespace=bar` 이고, `port=9000` 인 인그레스 네트워킹만을 허용

4. 특정 네임스페이스에 serviceaccount 생성하고, clusterrole, clusterrolebinding 하기

- verb, resource 를 유의
- clusterrolebinding 할 때 --serviceaccount:namespace:serviceaccountname 으로 적기는 했는데,
  이 클러스터 롤 & 롤 바인딩 리소스 자체를 동일 네임스페이스에 생성해야 하는건지?

클러스터롤은 전체 네임스페이스에 적용되는 것이기 때문에, default namespace에 생성해야 한다고 생각해서
그렇게 하긴 했는데.. 이게 맞는지는 모르겠음
(k get clusterrole 이랑, k get clusterrol -n 조건ns 으로 했을 때 결과가 같기도 하고..)

5. Ingress 만들기

- https://kubernetes.io/docs/concepts/services-networking/ingress/
- curl <Internal-IP>/hi 통신되는지 확인

`port=1234`인 `service=hello` 에 대하여 `path=/hello`로 호출되도록 인그레스 생성

6. Not Ready인 node 고치기

- systemctl status kubelet 확인

7. 기존 스케쥴링된 node가 not ready인 경우의 pod을 다른 node에 스케쥴링하기

- k drain node 하여 ScheduleDisabled 상태로 만든 후, 해당 노드에 할당되어 있던 파드를 재생성한다.

8. 조건에 맞는 노드 개수 count

9. 특정 Label을 가진 파드 중 CPU 사용률 높은 pod 이름 알아내기

- k top pod -l 라벨=조건

10. Logging sidecar 생성하기

11. Two containers pod 생성하기

12. nodeSelector 옵션을 추가하여 파드 생성하기

13. PV 생성하기

14. PVC 생성하고 pod 연결하기. 그리고 pvc resize 하고 내용 기록하기

15. deployment 스케일하기

16. deployment service 노출시키기
    이거 뭐하는 문제인지 제대로 모르겠음...
    deployment 에 container port name=http, port=TCP/80 를 설정하고
    새로운 서비스를 name=http, port=TCP/80 으로 생성하였는데
    그게 기존 파드 - 생성한 서비스 - 업데이트한 디플로이먼트를 제대로 바인딩 하고 있는건지 잘 .. 모르겠음..

17.

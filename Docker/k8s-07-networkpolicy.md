# NetworkPolicy 

NetworkPolicy를 사용하여 파드가 네트워크 상의 다양한 네트워크 엔티티와 통신할 수 있도록 허용하는 방법을 지정할 수  있다.

파드가 통신할 수 있는 엔티티는 다음 3개의 식별자 조합을 통해 식별된다.

1. 허용되는 다른 파드(예외: 파드는 자신에 대한 접근을 차단할 수 없음)
2. 허용되는 네임스페이스
3. IP 블록(예외: 파드 또는 노드의 IP 주소와 관계없이 파드가 실행 중인 노드와의 트래픽은 항상 허용됨)


파드는 파드를 선택한 네트워크폴리시에 의해서 격리된다. 네임스페이스에 특정 파드를 선택하는 네트워크폴리시가 있으면 해당 파드는 네트워크폴리시에서 허용하지 않는 모든 연결을 거부한다.


```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  # ingress/egress 트래픽에 대해,
  # default 네임스페이스에서 role=db 레이블을 사용하는 모든 파드를 격리한다.
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  # spec.podSelecter 의 조건으로 격리된 파드에 대해, TCP port 6397 로의 연결을 허용한다.
  # 허용대상:
  # role=frontend 레이블을 가진 default 네임스페이스의 모든 파드
  # 네임스페이스와 project:myproject 레이블을 가진 모든 파드
  # 172.17.0.0–172.17.0.255 와 172.17.2.0–172.17.255.255 의 범위를 가지는 IP 주소(예: 172.17.0.0/16 전체에서 172.17.1.0/24 를 제외)
  # TCP port 6397 으로의 연결을 허용한다.
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
      # 인그레스/이그레스 대상으로 허용되어야 하는 특정 namespace label
        matchLabels:
          project: myproject
    - podSelector:
      # 인그레스/이그레스 목적지로 허용되어야 하는 동일한 네임스페이스에 있는 특정 파드 pod label
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  # spec.podSelector 의 조건으로 격리된 파드에 대해, TCP port 5978 의 CIDR 10.0.0.0/24 로의 >연결을 허용한다.
  # 허용대상: all (from: 의 조건이 따로 없음)
  egress:
  - to:
  - ipBlock:
      cidr: 10.0.0.0/24
  ports:
  - protocol: TCP
    port: 5978

```
